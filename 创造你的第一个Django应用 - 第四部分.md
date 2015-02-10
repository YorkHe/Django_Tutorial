#创造你的第一个Django应用 - 第四部分

这章教程承接第三章结束的地方， 我们将继续那个网络民调应用， 并且将关注与简单的表单处理与精简我们的代码。

##创造一个简单的表单

让我们来升级一下上个教程中创建的问题详细内容模板(`polls/detail.html`)， 让它包含一个HTML的`<form>`元素:

```html
<!--polls/templates/polls/detail.html-->
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
{% endfor %}
<input type="submit" value="Vote" />
</form>
```

快速浏览一下上面的代码：

- 上面的这个模板给每一个选项显示了一个单选按钮。 每一个单选按钮的值与每一个选项的`id`相联系。 每一个单选按钮的名字都是`choice`. 这意味着， 当某人选择了这些单选按钮之一然后提交这个页面的时候， 系统会使用POST传递一个数据`choice=#`, 此处的`#`，代表选中选项的id。 这是HTML表单的基本概念。

- 我们把这个表单的动作属性设置为`{%url 'polls:vote' question.id%}`， 然后我们设置动作方法为`post`。 此处使用`method='post'`（与之相对的是`method='get'`）十分重要， 因为此处提交表单的这个动作将会出发服务器端的数据操作。 只要当你创建一个需要服务器端数据操作的表单的时候， 使用`post`总是最佳选择。 这个建议并不仅仅真对于Django， 这是Web开发的实际经验。

- `forloop.counter`记录了`for`标签内循环的次数。

- 既然我们创建了一个`POST`表单（拥有着操作数据的功能），我们就需要担心一下跨站请求伪造(Cross Site Request Forgeries)的问题。 但是谢天谢地，你并不需要太过于担心这个， 因为Django创造了一个非常易用的系统来避免这个问题。 简而言之， 所有使用`POST`提交，并且目标是一个内链URL的表单， 都需要有一个`{% CSRF token %}`标签。

现在， 让我们来创建一个能够处理提交的数据的Django视图吧。 别忘了， 在第三章教程中， 我们为这个民调应用创建了一个URL设置， 包含着如下代码：

	url(r'^(?P<question_id>\d+)/vote/$', views.vote, name='vote')

我们之前也为vote视图创造了一个简单版本， 现在让我们来写一个完整版的：

```python
#polls/views.py

from django.shortcuts import get_object_or_404, render
from django.http import HttpResponseRedirect, HttpResponse
from django.core.urlresolvers import reverse

form polls.models import Choice, Question
#...
def vote(request, question_id):
	p = get_object_or_404(Question, pk=question_id)
	try:
		selected_choice = p.choice_set.get(pk = request.POST['choice'])
	except (KeyError, Choice.DoesNotExist):
		#重新显示问题投票表单
		return render(request, 'polls/detail.html', {
		'question': p,
		'error_message': "You didn't select a choice.",
	})
	else:
		selected_choice.votes += 1
		selected_choice.save()
		#当成功提交一个POST数据之后， 要记得返回一个
		#HttpResponseRedirect， 不然当用户点击后退
		#按钮的时候， 同样的数据会被提交多次。
		return HttpResponseRedirect(reverse('polls:results', args=(p.id, )))
		

```

以上的代码包含一些我们之前没有提过的东西：

- `request.POST`是一个字典型对象。 它允许你通过键值对来访问POST传递的数据。 在上文的代码中， `request.POST['choice']`以字符串的形式，返回了选中的选项的ID。 `request.POST`的值总是以字符串表示的。

	Django同样提供`request.GET`来以同样的方式访问`GET`数据。 在我们的代码中，我们只使用`request.POST`， 来确保数据只由`POST`请求来操作。
- 若`choice`没有提供一个POST数据的话，`request.POST['choice']`会引发一个`KeyError`错误。 上面的代码检查了`KeyError`错误发生的可能性， 并在choice没有提供POST数据的时候重新显示表单页面，并且返回一个错误信息。
- 在添加了投票统计之后， 代码返回了一个`HttpResponseRedirect`而非一个普通的`HttpResponse`。 `HttpResponseRedirect`只需要一个参数： 被重定向的目标URL。 

	正如代码中的注释提到的那样， 你应当总是返回一个`HttpResponseRedirect`， 这不仅仅是对于Django而言。 这是一个优秀的编程实践经验。

- 我们在本例的`HttpResponseRedirect`构造器中使用了一个`reverse()`函数。 这个函数帮助我们避免了在视图函数中硬编码写入一个URL。 给定我们想要传递到的那个视图的名字与这个URL模式所需的参数，在这个例子中， 使用我们在第三章教程中创建的URL设置， 这个`reverse()`函数会返回如下字符串：

		'/polls/3/results/'

	这里的3是`p.id`的值。 这个重定向的URL会调用`result`视图来显示最终的页面。

正如我们在第三章教程中提到的那样， `request`是一个`HttpRequest`对象。 

在某人为某个问题投票之后， `vote()`视图重定向到这个问题的最终视图中， 它的视图代码如下：

```polls/views.ly
from django.shortcuts import get_object_or_404, render


def results(request, question_id):
	question = get_object_or_404(Question, pk=question_id)
	return render(request, 'polls/results.html',{'question':question})
```

它与我们在第三章写的`detail`视图几乎完全一样， 仅有的不同之处在于模板的名字。 我们将会在稍后修改这处冗余。

现在， 创建一个`polls/results.html`模板

```html
<!--polls/templates/polls/results.html-->
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

现在在你的浏览器中访问`/polls/1`来为这个问题投票。 你将见到一个随着你每次投票都有变化的结果页面。 如果你没有选择一个选项就提交了的话， 你还将见到一个错误页面。

##使用基础视图： 代码越少越好

之前的`detail()`与`result()`视图都极其简单， 以及， 像之前提过的那样—— 冗赘。 而那个显示着问题列表的`index()`视图， 也是一样。

这些视图体现着网页开发的一个基本模式：根据URL中传递的一个参数来从数据库中获取数据， 读取一个模板然后返回渲染之后的页面。 因为这实在是太平常了， Django提供了一个快捷表示， 叫做基础视图(generic views)（或者叫泛型视图）.

基础视图将程序抽象表示， 以至于你甚至不需要书写Python代码就能够创建一个应用。

然我们把民调应用转化成使用基础视图系统， 所以我们就可以删去一大堆无用的代码。 我们只需做一下这几步就可以完成转化：

- 改变URL设置
- 删除一些旧的，不需要的视图
- 引入基于基础视图系统的新的视图


###修改URL设置

首先， 打开`polls/urls.py`, 并作如下修改：

```python
#polls/urls.py

from django.conf.urls import patterns, url

from polls import views

urlpatterns = patterns('',
	url(r'^$',views.IndexView.as_view(), name ='index'),
	url(r'^(?P<pk>\d+)/$', views.DetailView.as_view(), name='detail'),
	url(r'^(?P<pk>\d+)/results/$', views.ResultsView.as_view(), name='results'),
    url(r'^(?P<question_id>\d+)/vote/$', views.vote, name='vote'),
)
```

注意第二个和第三个正则表达式匹配参数名已经从`<question_id>`变成了`<pk>`

###修改视图

接下来， 我们要对旧的`index`, `detail`和`result`视图做一些修改， 并且使用之前提到的Django的基础视图。 打开`polls/views.py`， 并作如下修改。

```python
#polls/views.py

from django.shortcuts import get_object_or_404, render
from django.http import HttpResponseRedirect
from django.core.urlresolvers import reverse
from django.views import generic

from polls.models import Choice, Question

class IndexView(generic.ListView):
	template_name = 'polls/index.html'
	context_object_name = 'latest_question_list'

	def get_queryset(self):
		return Question.objects.order_by('-pub_date')[:5]

class DetailView(generic.DetailView):
	model = Question
	template_name = 'polls/detail.html'

class ResultsView(generic.DetailView):
	model = Question
	template_name = 'polls/result.html'

def vote(request, question_id):
	...#与之前相同
```

我们使用了两个基础视图： `ListView`与`DetailView`。 这两个视图分别抽象了“显示一列对象”和“显示某个特定对象的详细内容”两个概念。
- 每一个抽象视图都需要直到它所作用的数据模型。 这个信息由`model`属性传递
- `DetailView`基础视图需要由URL中匹配得到被称为`pk`的主键键值， 所以我们在URL设置中把`question_id`改名作`pk`

默认情况下， `DetailView`基础视图使用叫做`<app name>/<model name>_detail.html`的模板。 在本例中， 它使用的是`polls/question_detail.html`。 这里的`template_name`属性被用来告诉Django去使用一个特定的模板名称， 而非Django自动默认的模板。 我们还为`result`视图指定了一个模板名称 - 这确保了`result`视图和`detail`视图在渲染时会有不同的显示， 尽管他们都是基于`DetailView`。

同样的， `ListView`使用默认的`<app name>/<model name>_list.html` 我们使用`template_name`来告诉`ListView`使用我们存在的`polls/index.html`模板

在之前的教程中， 我们将一个包含着`question`和`latest_question_list`的内容对象传递到模板中。 对于`DetailView`来说， 这些信息都是自动传递的。 因为我们使用了一个Django的视图。 Django能够自己为这个内容决定一个合适的名字。 但是， 对于`ListView`,自动生成的内容变量是`question_list`。 为了覆盖这个， 我们提供了`context_object_name`属性， 特殊指定了我们想要替代性地使用`latest_question_list`。 作为一个可选的方法， 你应该更改你的模板来匹配新的默认内容变量 - 但是让Django使用你想要的变量会更简单。

运行服务器，你就会看见基于基础视图的全新的民意调查应用。