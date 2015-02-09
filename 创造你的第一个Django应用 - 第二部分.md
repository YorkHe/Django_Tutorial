#创造你的第一个Django应用 - 第二部分

这章教程从教程一结束的地方开始。 我们继续编写这个网络民调应用， 并且将关注于Django自动生成的后台站点。

> **思想**
> 为你的同事或者客户生成一个自动站点，以供他们添加，修改和删除内容是一件多么无趣而又重复，缺乏创造的事情。 出于这个原因， Django提供了完全自动化的基于数据模型的后台站点的生成功能。
> Django最初是作为一个新闻站点框架使用的， 有着非常清晰的内容提供者和公众的角色区分。 网站管理员操作这个系统并且添加新的新闻， 专访， 体育比赛的得分等等。 这些内容将在公共站点显示出来。 Django通过创建一个统一的编辑后台来解决了这个问题。
> 编辑后台并非是为网站访问者所准备的， 他们是为了网站管理员设计的。

##创建一个管理员用户

首先我们需要创建一个能够登录进入管理员后台的用户。  运行如下代码：

	$ python manage.py createsuperuser

输入你想要取的用户名后按下回车

	Username: admin

然后输入邮箱

	Email address: admin@example.com

最后一步是输入密码， 你将会被要求输入两次密码， 第二次是对第一次输入的密码的确认。

	Password: ********
	Password(again): ********
	Superuser created successfully.

##启动开发服务器

Django管理员后台将会默认自动运行。 让我们启动管理员后台然后探索一下它吧。

回忆一下在第一章教程中学到的如何启动开发服务器的命令：

	$ python manage.py runserver

现在， 启动一个网页浏览器， 然后访问你的根域名的"/admin/“， 比如http://127.0.0.1:8000 你将会看见如下的管理员登录界面：
![管理员登陆界面](https://docs.djangoproject.com/en/1.7/_images/admin01.png)

因为翻译功能会被自动启动， 所以很有可能你会看见这个登录界面的语言将会与你的浏览器的默认语言保持一致。 

> **和你所看见的不一样？**
> 如果这时候，你看见的是如下的错误页面而非如上的登录界面的话：
> 
	ImportError at /admin
	   cannot import name patterns
	   ...
> 这说明你很有可能使用着一个与此教程不相符的Django版本。 你将需要一个更新的Django或者一个更老的教程

##进入管理员后台

现在试着使用你在之前的步骤中创建的超级用户账号登录进去。 你应该能够看见如下的Django管理员主页：
![Django管理员主页](https://docs.djangoproject.com/en/1.7/_images/admin02.png)

你将看见几种可以修改的内容： 用户组(Groups)和用户(Users)。 这些功能由django.contrib.auth， Django的授权功能框架提供。

##让我们的民调应用能够在后台系统中进行修改

但是我们写的应用在哪里？ 它并没有显示在后台系统里。

其实只要做一件事情就够了： 我们需要告诉管理员后台Question对象具有一个管理员借口。 要完成这件事， 打开polls/admin.py 文件， 然后作如下修改：

```python
#polls/admin.py
from django.contrib import admin
from polls.models import Question

admin.site.register(Question)
```

##探索一下免费（free）的管理员功能

现在我们已经把Question对象在管理员后台进行了注册， Django现在明白它应该被显示在后台系统的主页中：
![后台系统](https://docs.djangoproject.com/en/1.7/_images/admin03t.png)

点击"Questions"， 你就进入了问题列表的 "Change list"(修改列表)之中。 这个页面显示在数据库之中的所有问题， 并允许你选择一个来对其进行修改。 这里有我们在先前的教程之中创建的”What's up"问题

![What's up](https://docs.djangoproject.com/en/1.7/_images/admin04t.png)

点击"What's up"来对其进行修改

![enter image description here](https://docs.djangoproject.com/en/1.7/_images/admin05t.png)

此处需要注意：
- 这个表格由Question数据模型自动生成
- 不同的域类型(`DateTimeField`, `CharField`)对应于不同的HTML输入空间。 每一种域类型都明白其应当如何在Django后台中展现/
- 每一个`DateTimeField` 都有其自己的JavaScript快捷方式。 日期表示有一个“今天”快捷方式， 并且有一个日历图标。 时间表示有一个“现在”快捷方式并且有一个钟表图标，弹出常用时间的列表。

页面的末尾给你一些选项：
- 保存 - 保存更改并且返回这个对象的修改列表页面
- 保存并继续 - 保存更改并重新读取到这个对象的管理页面
- 保存并添加下一个 - 保存更改并且从这个对象读取一个新的创建表单
- 删除 - 显示一个删除确认表单

如果”发布日期”的值与你在之前创建这个问题的时间并不相同的话。这可能意味着你忘记了修正`TIME_ZONE`设定的值。 改正它， 并重新读取这个界面， 检查一下是否显示正确的值。

点击“今天”或者“现在“ 快捷方式来修改“发布日期”。 然后点击”保存并继续“， 然后点击右上方的“历史” ， 你会看见一个列举着通过Django管理后台对这个对象进行修改的记录的页面， 时间戳显示在左侧， 修改者显示在中间。 
![TimeStamp](https://docs.djangoproject.com/en/1.7/_images/admin06t.png)

##自定义管理员表单

给你一点时间来为你省下不用写的那么一大堆代码感到惊讶。 通过把Question对象利用admin.site.register(Qustion)来注册到管理后台， Django能够通过其自己的理解创建一个默认的表单。 一般来说， 你可能会希望自定义这个表单的样子以及工作方式。 你可以在通过注册这个对象时告诉Django这些选项来做到这一点。

然我们看看它是如何做到把编辑表单的两个域的顺序调换过来把。 把`admin.site.register(Question)`一行做如下替换：
```python
#polls/admin.py
from django.contrib import admin
from polls.models import Question

class QuestionAdmin(admin.ModelAdmin):
	fields = ['pub_date', 'question_text']

admin.site.register(Question, QusestionAdmin)
```

你只需照着这个模式来就可以： 创建一个数据模型的管理员对象， 然后把它作为一个第二变量传递给admin.site.register()　——　在任何你想要为一个对象改变其管理员后台选项的时候

上面的这个改动把”发布日期“　提前到了　“问题内容”之前
![Before](https://docs.djangoproject.com/en/1.7/_images/admin07.png)


对于两个域来说， 这并不引人惊讶。 但是对于一个有着很多个域的管理员表单来说， 为这些域给定一个确定的顺序就显得至关重要了。 

说到有很多个域的表单， 你可能会希望把这些域分成不同的类别：

```python
#polls/admin.py

from django.contrib import admin
from polls.models import Question

class QuestionAdmin(admin.ModelAdmin):
	fieldsets = [
		(None, {'fields':['question_text']}),
		('Date information', {'fields':['pub_date']}),
	]

admin.site.register(Question, QuestionAdmin)
```

`fieldsets`内的每个元组的第一个元素，就是每一个不同的类别的标题。  我们的表格现在长这个样子：

![form](https://docs.djangoproject.com/en/1.7/_images/admin08t.png)

你可以为每个分类任意制定HTML类。 Django提供了一个`collapse`类， 在默认情况下， 会把这个分类给折叠起来。 这在你有一个很长的表单的时候十分重要。

```python
#polls/admin.py

from django.contrib import admin
from polls.models import Question

class QuestionAdmin(admin.ModelAdmin):
	fieldsets = [
		(None, {'fields':['question_text']}),
		('Date information',{'fields':['pub_date'], 'classes':['collapse']}),
	]
admin.site.register(Question, QuestionAdmin)
```

![Collapse](https://docs.djangoproject.com/en/1.7/_images/admin09.png)


##添加相关的数据对象

好了， 我们有了问题对象的后台界面了。 但是一个问题有多个选项，而目前的后台系统并不能显示选项。

仅仅是**目前**来说。

有两个方式能够解决这个问题。 第一种方式是就像我们之前注册问题对象那样注册一个选项对象。

```python
#polls/admin.py

from django.contrib import admin
from polls.models import Choice, Question
# ...
admin.site.register(Choice)
```

现在”Choice”也可以在后台系统被访问到了。 “添加选项” 的表单像这个样子：
![Choice](https://docs.djangoproject.com/en/1.7/_images/admin10.png)

在这个表单中， Question域作为一个选项框存在。 它包含着数据库中的所有问题。 Django懂得一个“外键”意味着这个域应当在后台表单中被表达为一个选项框。 在现在， 选项框中只有一个问题。

注意在Question旁边的那个“添加新问题”链接。 每一个有着外键关系的域都有这么一个选项。 当你点击它的时候， 有一个带着“添加问题”表单的窗口会弹出来。 如果你在这个窗口之中添加了一个新的问题并点击保存。 DJango会将其存进数据库中并且会动态地在你所浏览的这个页面的选项框中添加你刚刚添加的这个问题。 

但是， 实际上， 这是一个十分低效的添加问题的方法。 更好的方法是在你创建一个新问题的时候就创建相对应的一族选项。 让我们开始做吧。

删除Choice模型调用的register函数， 然后依照如下代码进行修改：

```python
#polls/admin.py

from django.contrib import admin
from polls.models import Choice, Question

class ChoiceInline(admin.StackedInline):
	model = Choice
	extra = 3

class QuestionAdmin(admin.ModelAdmin):
	fieldsets = [
		(None, {'fields':['question_text']}),
		('Date information',{'fields':['pub_date'], 'classes':['collapse']}),
	]
	inlines = [ChoiceInline]

admin.site.register(Question, QuestionAdmin)
```

以上代码告诉Django， Choice选项由Question管理主页进行编辑。 默认情况下， 提供3个空白的选项以供选择。

读取“添加问题”页面来看看它长什么样子。

![ADD Question](https://docs.djangoproject.com/en/1.7/_images/admin11t.png)

工作原理如下： 这里有三个空白的选项 —— 由extra定义的—— 每一次你访问这个已经存在的类的时候， 你都会得到另外三个空白的选项。

在这三个空白选项的最后， 你会看见一个“添加另一个新的选项”链接， 如果你点击了它， 就会有一个更新的选项被添加进去。 你这时可以点击新的选项的右上角的那个“叉”来删除他。 但是你不能删除默认的那三个选项：


![Add](https://docs.djangoproject.com/en/1.7/_images/admin15t.png)


有一个小小的问题就是，这么多选项占据了满满一个屏幕。 出于这个原因， Django提供了一个更为灵活的方式将每一个选项极其相关内容在同一行进行显示。  你只需要依照如下修改`ChoiceInline`就可以：

```python
#polls/admin.py
class ChoiceInline(admin.TabularInline):
	#...
```

继承了`admin.TabularInline`类， （而非`StackedInline`), 相关的对象就会以一个更为紧凑的方式进行排列：

![compact](https://docs.djangoproject.com/en/1.7/_images/admin12t.png)

注意到那个多出来的"Delete?"列了么？ 那使得你可以删掉后面再新增的选项。

##自定义修改列表

现在问题的管理界面看上去挺不错，让我们对“修改列表”页面——那个显示着 数据库中所有问题的页面做一些小小的修改吧。 

现在这个页面长这个样子：

![enter image description here](https://docs.djangoproject.com/en/1.7/_images/admin04t.png)

默认情况下，Django会显示每个对象的str()方法的返回值。 但是在某些时候如果只显示个别的域会更有帮助。 出于这个目的， 我们可以使用`list_display`管理选项。 这个选项划定了一个显示对象的元组， 元组内的元素作为最终显示表单的列进行表示。

```python
#polls/admin.py
class QuestionAdmin(admin.ModelAdmin):
    #...
    list_display = ('question_text', 'pub_date')
```

为了更好的显示， 我们把在第一章教程中创建的自定义方法`was_published_recently`也包括进来

```python
#polls/admin.py
class QuestionAdmin(admin.ModelAdmin):
    #...
    list_display = ('question_text', 'pub_date', 'was_published_recently')
```

现在问题的修改列表长这个样子：

![enter image description here](https://docs.djangoproject.com/en/1.7/_images/admin13t.png)

你可以通过点击列表的表头来对其中的值进行排序--除了`was_published_recently`之外。 因为目前还不支持对一个自建方法进行排序。 同时还要注意的是， `was_published_recently`方法的表头名字是将方法名的下划线用空格代替， 之后把首字母大写得到。 并且这一列的每一行都包括着这个方法的出输出值的字符串表示。

你可以通过给模型方法一些新的属性改正这个列表的显示：

```python
#polls/models.py

class Question(models.Model):
    #...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
    was_published_recently.admin_order_field = 'pub_date'
    was_published_recently.boolean = True
    was_published_recently.short_description = 'Published recently?'
```
阅读["list_display"](https://docs.djangoproject.com/en/1.7/ref/contrib/admin/#django.contrib.admin.ModelAdmin.list_display)文档来获得更多关于这些方法属性的信息

再一次修改你的 polls/admin.py 文件来添加一个对于Question修改列表页面的改进 ： 一个使用list_filter方法的过滤器。 在QuestionAdmin方法中添加如下一行：

    list_filter = ['pub_date']
    
这一行代码添加了一个允许人们按照`pub_date`域来过滤表内信息的侧边栏：
!["filter"](https://docs.djangoproject.com/en/1.7/_images/admin14t.png)

这一种类型的过滤器的显示取决于它所基于的域的数据类型。 因为`pub_date`是`DateTimeField`类型的。 Django懂得应该给予其如下的过滤选项:任何时候、今天、七天内、这个月、这年

到目前为止， 一切看上去都还好。  让我们再添加一点搜索功能：

    search_fields = ['question_text']
    
这在修改列表界面的顶端添加了一个搜索框。 当某个人输入一个搜索词的时候， Django会搜索`question_text` 域。 你想包括多少个域都可以 - 因为这个搜索背后使用的是SQL的`LIKE` 命令， 所以限制一下你的搜索域可能会对搜索速度等有更大的帮助。
    
这个界面还提供了换页的连接。 默认可以在每一个页显示100个元素。  如果你喜欢的话，换页连接， 搜索框， 过滤器， 日期层次和列表头排序这几项功能是可以工作得很顺畅的。
    
##自定义管理后台的外观
    
很明显， 在管理后台每一页的顶端有一个"Django 管理后台"的字是很蠢的。 这只是一个文字占位符而已。
    
通过Django的模板系统，他们是很容易改变的。 Django的管理后台是依靠Django自身搭建的， 它的接口也使用的Django自身的模板系统。

###自定义你的项目的模板
在你的项目目录下创建一个`templates`文件夹。 模板可以存放在任何可以被Django访问到的地方。 (Django和你的服务器运行在同一个地方)。 不管怎么说， 把你的模板与项目放在同一个地方总是一个很好的习惯。

打开你的设置文件， 添加一个`TEMPLATE_DIRS`设置选项：

    TEMPLATE_DIRS = [os.path.join(BASE_DIR, 'templates')]
    
TEMPLATE_DIRS 是一个当Django读取模板文件时查询的地址列表。

现在在`templates`里创建一个`admin`目录， 把Django框架源代码(django/contrib/admin/templates)中的admin/base_site.html文件拷进该文件夹中。

> Django源代码放在哪里？
> 如果你在寻找Django框架源代码的时候出现了问题， 请运行如下代码：
> 
    $ python -c "
        import sys
        sys.path = sys.path[1:]
        import django
        print(django.__path__)"

然后就只需打开文件，把`{{site_header|default:_('Django administration')}}`（包括大括号）给替换成你想要写的站点名字就可以了。 修改完的代码如下所示：

```
{% block crading %}
<h1 id="site-name"><a href="{% url 'admin:index' %}">Polls Administration</a></h1>
{% endblock %}
```

我们通过这种方法来教你如何重写模板。 在实际的项目中， 你更有可能使用`django.contrib.admin.AdminSite.site_header`属性来更简单地做这个自定义的操作。

这个模板文件包含了许许多多类似于`{%block brading %}`和`{{title}}`的文字。 这里的`{%`和`{{`标签都属于Django的模板语言的一部分。 当Django对 admin/base_site.html进行渲染的时候， 这个模板语言将会被作为基准来生成最终的HTML页面。 如果你现在对于Django的模板语言一无所知， 也不需要担心。 我们将会在第三章的教程中仔细介绍它的。 

请记住， 任一个Django的默认后台系统都是可以被修改的。 要想修改一个模板， 只需做如同你对base_site.html做的事情一样 -  把它从默认的目录拷贝出来， 然后对其做修改。

###自定义你的*应用*模板


聪明的读者会问： 既然默认情况下, `TEMPLATE_DIRS`是空的， 那么Django是怎样招到默认的后台系统模板的？ 这个问题的答案是： 在默认情况下， Django自动地查找每一个应用包的`templates/`子目录， 作为一个备用方法（别忘了django.contrib.admin 也是一个应用）

我们的民调应用并非十分复杂， 所以我们不需要自定义一个后台模板。 但是如果它逐渐发展至更加复杂并需要修改以增加部分功能的时候， 修改这个应用的模板，而非修改整个项目的通用模板， 可能会是一个更好的办法。 在这种情况下， 你可以把这个民调应用使用与任一个新的应用中， 并且确保它能够自己找到其需要的自定义模板。

###自定义后台主页

同样的， 你可能希望要自定义Django后台主页的样子。

默认情况下， 它以字母序显示`INSTALLED_APPS`中所有在管理员应用中注册的应用。 你可能希望能够对整体页面布局做一个大的调整。 毕竟后台主页是整个后台系统最重要的页面， 它应当做到尽可能的易用。

被提供来自定义的模板是 admin/index.html （做你之前对base_site.html相同的事——把它从默认文件夹拷出来， 然后再做修改）。 修改这个文件， 你会看见它使用了一个模板变量叫做`app_list`。 这个变量包含了所有安装的Django应用。  你也可以不使用这个默认的变量， 反而直接引用一个确定的应用的管理界面。  再说一遍， 不用担心你现在不能理解模板语言， 我们将会在第三章中仔细介绍的。
