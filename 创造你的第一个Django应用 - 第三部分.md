#创造你的第一个Django应用 - 第三部分

这章教程将继续第二章结束的地方， 我们将继续编写我们的网络民调应用，并切将专注于创建一个界面 - **视图(view)**

##思想

视图， 是在你的Django应用中具有一个特定模板的特定的函数， 它生成某“种” 特定的Web页面。

举例来说， 在一个博客应用之中， 你可能会有如下的视图：

- 博客主页， 显示最新的几篇博文
- 博文的“详细” 页面， 完整展示某一篇博文的详细信息
- 以年为基础的分类页面 ，以某一年的月份为基础分类博文
- 以月为基础的分类页面， 以某一月的日子为基础分类博文
- 以日为基础的分类界面， 显示某一天发布的所有博文
- 评论动作， 处理对某一篇博文的评论功能

在我们的民意调查应用中， 我们有如下四个视图：

- 问题“索引”视图： 显示最新的几个问题
- 问题“详细”视图： 显示一个问题的文本， 以及一个投票的表单
- 问题“结果”视图： 显示一个特定问题的投票结果
- 投票动作： 处理对某个特定问题的特定选项的投票操作

在Django中， 网页和其它的内容是由视图所呈现的。 每个视图由一个单独的Python函数所表示（或者在以类为基础的视图中， 以方法来表示）。 Django会通过审视被要求访问的URL（准确地说， 域名之后的那部分URL）来选择一个特定的视图。

你可能经常会遇见像 "ME2/Sites/dirmod.asp?sid=&type=gen&mod=Core+Pages&gid=A6CD4967199A42D9B65B1B" 这样的URL， 你会十分欣慰Django有着一个比这要优雅的多的URL系统。

一个URL模式字符串其实就是URL的一般形式， 例如： `/newsarchive/<year>/<month>`

要从一个URL中获得一个视图， Django使用被称为`URLconfs`的方法。 URLconfs适配URL模式字符串（用正则表达式的形式描述），并选择相对应的视图。

##编写你的第一个视图

让我们来着手编写第一个视图， 打开`polls/views.py`文件， 然后输入如下的Python代码：

```python
#polls/views.py

from django.http import HttpResponse


def index(request):
	return HttpResponse("Hello, world. You're at the polls index.")
```

这是Django能够提供的最基础的视图。 要访问这个视图， 我们需要将其映射到URL上 -  所以我们需要一个URL配置。

若想在应用目录下设置一个URL配置， 创建一个叫做`urls.py`的文件。 你的应用目录现在应该长这个样子：

```
polls/
	__init__.py
	admin.py
	models.py
	tests.py
	urls.py
	views.py
```

在`polls/urls.py`文件中应当包含如下内容：

```python
#polls/urls.py

from django.conf.urls import patterns, url

from polls import views


urlpatterns = patterns('',
	url(r'^$',views.index, name='index'),
)
```

下一步是把URL设置的根节点指向`polls.urls`模块。 在`mysite/urls`中添加一个`include()`方法：

```python
#mysite/urls.py

from django.conf.urls import patterns, include, url
from django.contrib import admin

urlpatterns = patterns('',
	url(r'^polls/', include('polls.urls')),
	url(r'^admin/', include(admin.site.urls)),
)
```

你现在就已经把一个`index`视图添加进了URL设置之中。 在你的浏览器中访问 http://localhost:8000/polls/ ， 你就将见到你在index视图中定义的那行文字： *Hello, world. You're at the polls index*

url函数传递四个参数， 两个是必须参数： `regex`正则表达式 和 `view`视图。  以及两个可选参数： `kwargs`关键字参数,`name`名称。 

现在让我们一个一个介绍一下他们各自分别代表着什么：

###url()参数： regex 正则表达式

术语'regex' 是 正则表达式'regular expression' 的缩写， 代表着一种对字符串中特定模式进行匹配的语法。 Django从第一个正则表达式开始， 并按照列表依次对要求访问的URL进行匹配， 直到找到所需的URL。 

请注意， 这些URL的正则匹配并不会对POST和GET的传递参量进行匹配。 如在一个`http://www.example.com/myapp`的URL请求中， URL设置会匹配 `myapp/`。 而在一个`http://www.example.com/myapp/?page=3`的请求中， URL设置仍然只会匹配`myapp/`

如果在正则表达式方面需要任何帮助， 请访问Wikipedia的正则表达式相关页面，与Python的re模块的相关官方文档。 以及由O’Relly公司出版的由Jeffrey Friedl所著的《精通正则表达式》也是相当不错的选择。 但实际上， 你并不需要成为一名正则表达式的专家， 你仅仅需要知道怎么去对一些简单的模式进行匹配就好了。 事实上， 十分复杂的正则表达式有着很惨的使用表现，所以你最好还是不要太过依赖于正则表达式。

最终， 一个性能方面的提醒： 这些正则表达式在URL设置模块被加载的时候就被编译好了， 他们运行的速度超级快（如果这个匹配并非像前文所讲的那么的复杂的话）。 

-------------------------

###url()参数： view 视图

当Django找到了一个正则表达式的匹配， 它会调用相对应的视图函数， 并传递一个`HttpRequest`作为第一参数，被正则表达式匹配的其它内容作为剩余参数。 如果正则表达式仅作简单的匹配， 匹配值将作为普通参数(positional arguments)进行传递， 如果使用了名称匹配， 那么匹配值将作为键值对来传递。 我们将会在稍后提供一个小小的例子。

----------------------------

###url()参数： kwargs 关键字

任意的关键字都可以被以字典的形式作为参数传递给目标视图。 在此教程中我们将不会使用这个功能。

--------------------------
###url()参数： name 名称

为你的URL起一个名字， 使得你可以在Django项目的其他地方， 尤其是模板中明确地调用它。 这个强大的功能使得你可以在只用在一个文件中就可以对url模式匹配做一个全局的修改。

----------------------------------
##编写更多的视图

现在让我们对polls/views.py 文件添加更多的视图。 这些视图有着些许的差别， 因为他们都接受了一个参数：

```python
#polls/views.py

def detail(request, question_id):
	return HttpResponse("You're looking at question %s." %question_id)

def results(request, question_id):
	response = "You're looking at the results of question %s."
	return HttpResponse(response %question_id)

def vote(request, question_id):
	return HttpResponse("You're voting on question %s." %question_id)
```

对这些新增的视图添加url链接

```python
#polls/urls.py

from django.conf.urls import patterns, url
from polls import views

urlpatterns = patterns('',
	#ex: /polls/
	url(r'^$', views.index, name='index'),
	#ex: /polls/5/
	url(r'^(?P<question_id>\d+)/$', views.detail, name='detail'),
	#ex: /polls/5/results/
	url(r'^(?P<question_id>\d+)/results/$', views.results, name = 'results'),
	#ex: /polls/5/vote/
	url(r'^(?P<question_id>\d+)/vote/$', views.vote, name='vote'),
)
```

现在在你的浏览器中实验一下： 如果输入`/polls/34`的话。 它将会运行`detail()`函数， 并且会显示你所输入的问题id所对应的那个问题。  试试`/polls/34/results/`和`/polls/34/vote/`也将出现相对应的结果

当某人对你的服务器请求一个网页： 比如说`/polls/34`的时候， Django会读取`mysite.urls`Python模块， 因为`ROOT_URLCONF`设置指向的这里。 它寻找到名为`urlpatterns`的变量并且逐一匹配其中的正则表达式。 这里我们使用的`include()`函数是对其它的URL设置的简单引用。 注意这里的`include()`函数中的正则表达式并没有一个`$`（标记字符串末尾）字符，而是一个下划线。 当Django遇到`include()`的时候， 它会将当前部分已经被匹配的URL部分删去， 然后把剩余部分交予include的URL设置去做进一步处理。

`include()`函数背后的思想是把URL变得“即插即用”。 在这里我们的民调应用是被放在`polls/urls.py`， 他们还可以被放在polls/下， 或者在/fun_polls/下， 或者在/content/polls/下， 他们还可以被放在其它目录下， 而这个应用仍然可以工作。

当用户访问`/polls/34/`的时候，Django会做如下操作：
- Django会找到与这个URL匹配的正则表达式`^polls/`
- 然后Django会截去匹配的字符串`polls/`， 然后把剩下的文本`34/`传递到`polls.urls`URL设置中， 来做进一步操作。 
- 此时这个字符串与`r'^(?P<question_id>\d+)/$'`相匹配， 最终映射到一个对`detail()`函数的调用上：
 
		detail(request = <HttpRequest object>, question_id='34')

上面的`question_id='34'`这一部分来自于`(?P<question_id>\d+)`。 这里使用一个括号来把一个模式“包裹”起来， 它“捕捉”其匹配的那个字符串， 然后把它作为一个参数传递给视图。 这里`?P<question_id>`定义了这个匹配内容的名字。 `\d+`是正则表达式语法中对一个数字序列的匹配定义。

因为URL模式是一系列正则表达式， 实际上他们能做的事情是没有限制的。 并没有需要添加一个类似于.html的后缀 —— 除非你真的想这么做， 在这种情况下你可以写如下的代码：

	(r'^polls/latest\.html$','polls.views.index'),

但是千万别， 这样真的太傻了。

##写真正能够做一些事的视图

每个视图都承担着一个或两个责任： 返回一个`HttpResponse`对象， 包含着这个被请求的页面的信息， 或者引发一个诸如`Http404`的异常——这都由你决定。

你的视图可以从一个数据库中读取记录。 它可以使用一个模板系统——Django默认的， 或者是第三方的Python模板——或者不使用。 它可以生成一个PDF文件， 输出XML文件， 实时生成zip文件…… 做一切你想的事情， 只要使用相对应的Python支持库就可以。

而Django所需要的仅仅是一个HttpResponse， 或者是一个异常。

出于方便起见， 让我们使用Django自带的数据库API。  以下是我们的新的index()视图， 它显示了数据库中最新的五个问题， 用逗号连接， 并根据发布时间排序。

```python
#polls/views.py

from django.http import HttpResponse
from polls.models import Question

def index(request):
	latest_question_list = Question.objects.order_by('-pub_date')[:5]
	output = ','.join([p.question_text for p in latest_question_list])
	return HttpResponse(output)
```

这里有一个问题： 这个页面的内容是被直接编码在视图源代码中的， 如果你想要修改的话。 需要修改这个视图的Python代码。 所以在这里我们使用Django的模板系统创建一个模板， 来把设计与视图分割开来。

首先， 在你的polls目录下创建一个templates文件夹。 Django将在这里寻找网页模板。

Django的`TEMPLATE_LOADERS`设置包含了一个指示从不同的源头调用模板的列表。 其中的一个默认选项是`django.template.loaders.app_directories.Loader`， 它自动在每一个`INSTALLED_APPS`中寻找一个'templates'文件夹。 这就是在第二章教程的时候，在我们没有修改	`TEMPLATE_DIRS`的情况下Django依然能够找到模板的原因。

> **安排模板**
> 我们*可以*把我们所有的模板都放在同一个地方， 比如一个巨大的模板目录下， 此时它依然可以工作得十分完美。 但是，这个模板属于那个民调应用， 所以与我们创建的管理后台的页面模板不同， 我们把这个模板放在应用的模板目录下， 而不是放在项目的模板目录下。 

在你刚刚创建的这个`templates`文件夹中， 再创建另一个叫做`polls`的文件夹。 并且在这个文件夹中， 创建一个叫做`index.html`的文件。 你的模板此时的目录应该是`polls/templates/polls/index.html`。 因为app_directories模板加载器如之前我们所描述的那样工作， 所以此时你可以简单地在Django中使用`polls/index.html`就可以调用这个模板。

> **模板命名空间**
> 因为我们*可能*会直接把文件放在 polls/templates之中（而不是另外新建一个polls文件夹）， 但是这确实是一个坏主意。 Django会选择与这个名字相匹配的第一个模板。 而如果此时你有一个与其同名而在不同应用下的模板的话。 Django将不能区分它们两个。 我们需要能够让Django准确无误地找到正确的那一个。 此时最简单的方式就是为其确定命名空间， 也就是把每个模板都放进用每个应用的名字命名的文件夹中


在这个模板中输入如下内容：

```html
<!--polls/templates/polls/index.html -->

{% if latest_question_list %}
	<ul>
	{% for question in latest_question_list %}
		<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
	{% endfor %}
	</ul>
{% else %}
	<p> No polls are available. </p>
{% endif %}
```
 
 然后升级我们的index视图， 来使用这个模板：

```python
#polls/views.py

from django.http import HttpResponse
from django.template import RequestContext, loader

from polls.models import Qeustion


def index(request):
	latest_question_list = Question.objects.order_by('-pub_date')[:5]
	template = loader.get_template('polls/index.html')
	context = Requestcontext(request, {
		'latest_question_list': latest_question_list,
	})
	return HttpResponse(template.render(context))
```

这个代码从叫做`polls/index.html`的文档中读取了模板， 并且把它传递给一个内容对象。 内容对象是一个将模板变量名与Python对象一一映射的字典对象。

通过你的浏览器访问`polls/`来读取这个页面， 此时你将能够看见一个列表显示着我们在第一章中创建的"What's up"问题。 这个链接指向这个问题的详细页面。

###一个快捷表示: `render()`

读取一个模板， 填入内容， 再用HttpResponse返回处理后的模板， 这一系列操作经常被执行。 所以Django提供了一个快捷表示， 我们用它重写一下`index()`视图：

```python
#polls/views.py
from django.shortcuts import render

from polls.models import Question


def index(request):
	latest_question_list = Question.objects.order_by('-pub_date')[:5]
	context = {'latest_question_list': latest_question_list}
	return render(request, 'polls/index.html', context)
	
```

这里我们可以注意到， 如果我们使用了render的话， 我们就不再需要导入`loader`, `RequestContext`和`HttpResponse`了。

`render()`方法把request对象作为第一参数， 然后把模板名字作为第二参数， 然后一个可选的字典变量作为第三参数。 它返回一个给定的模板通过给定的内容渲染之后得到的HttpResponse对象。

###引发一个404错误

现在， 让我们改动一下`detail`视图 - 那个显示给定问题的详细内容的视图： 

``` python
#polls/views.py

from django.http import Http404
from django.shortcuts import render

from polls.models import Question
#...

def detail(request, question_id):
	try:
		question = Question.objects.get(pk = question_id)
	except Question.DoesNotExist:
		raise Http404("Question does not exist")
	return render(request, 'polls/detail.html', {'question': question})
```

这里有一个全新的概念： 如果所要求的ID不存在的话， 这个视图引发了一个Http404异常。

我们将在稍后讨论你应该在`polls/detail.html` 模板中放一些什么。 但如果你想尽快地完成这个教程的话， 一个包含如下内容的模板就够了：

```
<!--polls/templates/polls/detail.html-->
{{ question }}
```

###一个快捷表示： `get_object_or_404()`

调用`get()`函数， 并在对象不存在的时候引发Http404异常，也是一个十分常见的操作。 Django提供了一个快捷表示。 根据其重写`detail()`视图，如下所示：

```python
#polls/views.py

from django.shortcuts import get_object_or_404, render

from polls.models import Question
#...
def detail(request, question_id):
	question = get_object_or_404(Question, pk=question_id)
	return render(request, 'polls/detail.html',{'question': question})
```

这个`get_object_or_404()`函数把一个Django数据模型作为它的第一参数， 以及任选的键值对作为其它参数。 这个函数把参数传递给`get()`函数， 如果对象不存在的话， 引发404异常

> **思想**
> 为什么我们使用一个函数`get_object_or_404()`， 而不是自动使用更高层面的`ObjectDoesNotExist`异常， 或者用一个API来引发Http404异常呢？
> 
> 因为这样的话就会把数据模型与视图联系在一起。 而Django的一个哲学就是低耦合度。 一些系统自带的合作函数存放于`django.shortcuts`模型之中

同样也有一个`get_list_or_404()`函数， 使用方法和`get_object_or_404()`一样，只不过使用的是`filter()`函数， 如果列表是空的就返回404异常


##使用模板系统

回到我们的民调应用的`detail()`视图中。 给定一个内容变量`question`, 我们的`polls/detail.html`应该长这个样子：
```html
<!--polls/templates/polls/detail.html-->
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
	<li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

模板系统使用 点 来访问一个变量的属性。 在上面的`{{ question.question_text }}`的例子中， 首先Django在question对象中做一次字典查找。  如果没有找到， 他做一次属性查找——在这个例子中成功找到了—— 如果还是失败的话， 他就会做一次列表查找。

对方法的调用发生在`{% for %}`循环之中： `question.choice_set.all` 被解析为Python的代码`question.choice_set.all()`， 将会返回一个可迭代的Choice对象并且适于在`{% for %}`标签内使用。

##删去在模板中硬编码的URL

别忘了， 在之前我们写那个polls/index.html模板中的链接的时候， 我们用了一些硬编码的风格：

	<li><a href="/polls/{{question.id}}/">{{question.question_text}}</a></li>

在这里使用硬编码，紧耦合方法的弊端在于， 如果要修改链接的话， 会造成相当大的修改模板的工作量。 既然你在`polls.urls`模块中的url()方法里定义了正则表达式的名字参数， 你就可以使用`{%url%}`标签来代替这里特定的url地址。

	<li><a href = "{% url 'detail' question.id %}">{{ question.question_text }}</a></li>

这个函数的工作原理是查询`polls.urls`模块中的URL定义。 你可以清除地看见名称'detail'被做了如下定义：

	url(r'^(?P<question_id>\d+)/$', views.detail, name='detail'),


如果你想要把问题的详细页面转换到别的URL上， 也只用修改polls/urls.py就好了。


##为URL名称确定命名空间

这个教程项目只包含一个应用：polls。 在真实的Django应用环境中， 可能在同一个项目下有着五个， 十个， 乃至20个或更多不同的应用。 Django是怎样区别其中不同的URL的？ 举例来说， polls应用有一个detail视图， 同样一个博客应用也可能有一个相同的视图。 Django使怎样确保在使用`{% url %}`模板标签的时候不至于混乱呢？

这个问题的答案就是为你的根URL配置添加命名空间。 在`mysite/urls.py`文件中，添加命名空间的定义。

```python
#mysite/urls.py

from django.conf.urls import patterns, include, url
from django.contrib import admin

urlpatterns = patterns('',
	url(r'^polls/', include('polls.urls', namespace="polls")),
	url(r'^admin/', include(admin.site.urls)),
)
```

然后把具体模板中的代码从：

	<li><a href = "{% url 'detail' question.id %}">{{ question.question_text }}</a></li>

改成：

	<li><a href = "{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
