#创造你的第一个Django应用 - 第一部分
让我们通过示例来进行学习吧。
通过这个教程， 我们将带领你一步一步的创建一个基础的进行社会调查的网络应用。

它将包含两个部分：
- 一个公共的网站允许人们进行访问并通过其进行投票
- 一个后台的管理员应用， 来允许你添加， 修改， 以及删除调查的项目

我们假设你已经安装完成了所有的Django程序。 你可以通过以下命令来检测你的Django程序的安装是否完好以及其版本号：

	$python -c "import django; print(django.get_version())"

如果Django被完整安装， 你将得到你所安装的Django的版本号。 如果没有的话， 你将得到一个表示为**"No module named django"**的错误。

这个教程为Django 1.7， Python 3.2或之后的版本书写。 

点击[如何安装 Django](https://docs.djangoproject.com/en/1.7/topics/install/) 来移除老版本的Django并安装一个更新的版本。

> **在哪里能够获得帮助？**
> 如果你在进行这个教程的过程中遇到了某些问题， 请向[django-users](https://docs.djangoproject.com/en/1.7/internals/mailing-lists/#django-users-mailing-list)发送信息， 或者访问[#django on irc.freenode.net](irc://irc.freenode.net/django)来和其它可能能够提供帮助的的Django用户进行交流。

##创建一个新的项目

如果这是你第一次使用Django， 你将需要进行一次初始化安装的过程。 顾名思义， 你需要自动生成一些代码，来组建一个Django 项目 - 也就是为了创建一个Django实例而进行的一系列设置的集合，包括**数据库设置**(database configuration)， **Django提供的一些可选功能的设置**(Django-specific options)和**应用指向的设置**(application-specific settings)等。

在命令行窗口下， **cd** 进入一个你想要你的项目代码存放的文件夹， 然后运行一下命令：

	$ django-admin.py startproject mysite

这个命令将会在当前目录下创建一个mysite文件夹。 如果这个命令并没有正常运行， 请参见 [运行 django-admin.py 时出现的问题](https://docs.djangoproject.com/en/1.7/faq/troubleshooting/#troubleshooting-django-admin-py)

>**提示**
>你需要避免项目名称与Python或者Django的一些内置组件重名， 如你不得不避免使用诸如"**django**"（与Django本身重名） 或 "**test**"（与Python的一个内置包重名）


>**这些代码应该存放在哪里？**
>如果你之前仅有纯PHP的背景（没有用过任何现代框架）的话， 你很可能会将项目的代码放在网页服务器的根目录（如**/var/www**）下。 对于Django， 你最好别这么做。 把这些Python代码存放在你的网页服务器的根目录下可能并非是一个很好的注意。 因为这样有着被他人通过网页访问到你的代码的风险。 这对于安全来说有着很大的隐患。
>把你的代码放在根目录之外的任何地方把， 比如**/home/mycode**之类。

让我们看看`startproject`命令创建了什么东西

```
mysite/
	manage.py
	mysite/
		__init__.py
		settings.py
		urls.py
		wsgi.py
```
>**和你所见的文件结构不一样？**
>默认的项目文件结构可能已经发生了变化。 如果你看见了一个“扁平化“的架构（没有里面的那一层**mysite/**目录)， 你很可能使用着一个与这个教程并不相同的Django版本。 你需要切换到更老版本的Django教程， 或者是一个更新版本的Django程序

这些文件是：
- 最外一层**mysite/**根目录就是一个存放你的项目文件的地方。 它的名字对于Django来说并不重要， 你可以将其重命名为你喜欢的任何东西。

- **manage.py**： 一个命令行程序， 使得你能够通过许多方式来与这个Django项目进行交互。 你可以在[django-admin.py 与 manage.py](https://docs.djangoproject.com/en/1.7/ref/django-admin/)读到关于这个文件的更多细节。

- 里面的一层**mysite/**目录是你的项目的实际上的Python包文件。 它的名字就是你在导入其中任何东西时所需要使用的包名。(例如： **mysite.urls**)

- **mysite/\_\_init\_\_.py**：一个空的文件， 用来告诉Python这个目录应当被识别为一个Python包 (如果你是一个Python新手的话， 请阅读在Python官方文档中关于包的[更多介绍](http://docs.python.org/tutorial/modules.html#packages))

- **mysite/settings.py** ： 这个Django项目的配置文档。 [Django设置](https://docs.djangoproject.com/en/1.7/topics/settings/)文档会让你懂得配置文档是如何工作的

- **mysite/urls.py**：存放这个Django项目的url路由声明。 一个你的Django网站的内容表(table of contents)。 你可以在[URL分发文档](https://docs.djangoproject.com/en/1.7/topics/http/urls/)处获得关于URL的更多信息。

- **mysite/wsgi.py**：一个支持WSGI的服务器运行你的项目时的访问入口。 参阅[如何通过WSGI部署](https://docs.djangoproject.com/en/1.7/howto/deployment/wsgi/)以获得更多信息

##配置数据库

现在对 **mysite/settings.py**进行修改。 这是一个通过代表着不同Django设置的模块变量(module-level variables)来对Django项目进行设置的Python模块。

默认情况下， 配置文件使用的数据库是SQLite。 如果你对于数据库一无所知， 或者你只是想要尝试一下Django， 这是你的最佳选择。 SQLite已经被包括在Python之中， 所以你不需要再安装任何东西以获取对于数据库的支持。

如果你想要设置对于别的数据库的支持， 请安装相对应的[数据库适配程序](https://docs.djangoproject.com/en/1.7/topics/install/#database-installation)(database bingings), 并且把`DATABASES`键的下列属性从`defalut`修改为与你链接的数据库相匹配的属性：
- **ENGINE**: **django.db.backends.sqlite3**或者**django.db.backends.postgresql_psycopg2**, **django.db.backends.mysql**， 或**django.db.backends.oracle**
- **NAME**： 你的数据库文件的名字。 如果你使用的是SQLite的话， 数据库将以一个文件的形式存放于电脑中。 在这种情况下， `NAME`属性将是完整的绝对路径， 包括这个文件的文件名。 

如果你并非使用SQLite作为你的数据库引擎， 类似于`USER`, `PASSWORD`, `HOST` 这样的属性必须被添加。 更多资料， 请参考关于数据库设置的[官方文档](https://docs.djangoproject.com/en/1.7/ref/settings/#std:setting-DATABASES)

>**提示**
>如果你正在使用 PostgreSQL 或者 MySQL, 请确保在这个时候你已经创建了一个数据库。 通过在你的数据库交互界面中使用`CREATE DATABASE database_name`命令可以创建一个数据库
>
>如果你正在使用SQLite， 那么你不需要在之前创建任何的数据库 -  数据库将会在需要的时候自动创建。

当你在配置**mysite/setting.py**文件的时候， 把`TIME_ZONE`配置成你所在的时区

并且， 请确保`INSTALLED_APPS`配置选项位于文件的开始。 这个选项列举了所有在此Django项目中运行的Django应用的名字。 应用可以在多个项目中使用， 并且你可以将其封包并分发给他人以供其在自己的项目中使用。

默认情况下， `INSTALLED_APPS`包含一下应用， 他们都是Django的内置应用：
- **django.contrib.admin**： 管理员界面， 你可以在这个教程的第二部分见到他
- **django.contrib.auth**： 一个授权系统（用户系统）
- **django.contrib.contenttypes**：一个内容类型(content types)框架
- **django.contrib.sessions**：一个会话(session)框架
- **django.contrib.messages**： 一个消息(message)框架
- **django.contrib.staticfiles** ：一个用于管理静态文件的框架

这些应用在默认情况下， 出于方便考虑， 都默认被包括在项目之中。

这些应用的某一些需要使用至少一个数据表， 所以我们在使用他们之前， 需要先创建一些数据表。 出于这个目的， 运行以下命令：

	$python manage.py migrate


`migrate`命令查看`INSTALLED_APPS`配置， 并且根据**mysite/settings.py**中关于数据库的设置，创建运行必须的数据表， 并且将应用运行所需的数据转移进去。 每一个转移发生时你将会看见一个反馈的消息。 如果你感兴趣的话， 运行数据库程序的命令行工具并且键入**\dt**(PostgreSQL) **SHOW TABLES;**(MySQL) **.schema**(SQLite)来显示Django创建的数据表。

>**对于极简主义者(minimalists)来说**:
>就像我们之前说过的那样， 这些应用都是为了应付常见情况而被包括进去的， 并非所有人都需要他们。 所以当你不需要他们其中的任何一个的时候， 在运行`migrate`命令之前，删除它吧。 `migrate`命令只会遍历`INSTALLED_APPS`选项并运行其转移操作。

##开发服务器(The development server)

让我们确认一下你的Django项目能否正常运行： 进入**mysite**目录， 运行如下命令：

	$python manage.py runserver

你将会看见如下输出行：

```
Performing system checks...

0 errors found
February 02, 2015 - 15:50:33
Django version 1.7, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

此时你就已经启动了Django开发使用的服务器， 一个完全使用Python写成的轻型的Web服务器。 我们已经将其包括在Django之中， 所以你可以很快速地开发程序， 而不需要再去配置一个产品级的服务器 - 例如Apache - 指导你准备好进行工作。 

但是同时也要记住： **不要**将这个服务器用于任何实际的产品当中， 这只是一个供开发测试使用的轻型服务器软件。

现在服务器就已经开始运行了， 使用你的网页浏览器访问 http://127.0.0.1:8000/。 你将会看见一个有着淡蓝色优雅色调的"Welcome to Django" 界面。 它成功运行了！


> **修改端口**
> 默认情况下， `runserver`命令把开发服务器假设在内网的8000端口上。
> 如果你想要更改这个端口的话， 把它作为一个命令行参数进行传递。 举例来说， 如下的命令在8080端口启动了一个开发服务器。
> 
	$python manage.py runserver 8080
> 如果你想要修改这个服务器的IP地址， 也把它与端口号一起作为参数传递。 如下的命令将会使服务器监听所有的公有IP地址：
>
	$python manage.py runserver 0.0.0.0:8000
> 完整的文档请参阅[runserver](https://docs.djangoproject.com/en/1.7/ref/django-admin/#django-admin-runserver)文档

> **Runserver的自动重载**
> 在需要的时候， 开发服务器会自动重载。 你不需要在对代码进行改动之后重新加载这个服务器。 但是诸如像添加文件这样的操作并不会触发服务器的自动重载， 所以在这种情况下你需要手动重新启动服务器。


##创建数据模型(models)

现在你的运行环境——项目——已经创建起来了， 你可以开始正式的工作了。

每一个你使用Django书写的应用都包含着一个遵守一定约定的Python包。 Django有着自动生成一个app的基础目录结构的功能， 所以你可以更专心于书写代码而非创建文件目录。

> **项目(Projects) 与 应用(Apps)**
> 一个“项目”和一个“应用”之间究竟有什么差别？ 一个应用是一个能够做一些实际的事情的网络应用。 诸如：博客系统， 公共记录的数据库，或者一个简单的民意调查的应用。 一个项目是一个网站的配置文件与应用的集合。 一个项目可以包含许多应用， 一个应用可以在多个项目中使用。

你的应用可以存放在你的Python路径（Python进行包搜索的路径）下的任何地方。 在这个教程之中， 我们将把应用创建在manage.py文件同目录下， 这样拿就可以被作为其自身的一级模块(top-level module)所引用， 而不是mysite的子模块(submodule)。

要创建一个应用， 确保你和manage.py在同一个文件夹下， 并输入如下命令：

	$python manage.py startapp polls

这个命令会创建一个polls目录， 其结构如下：
```
polls/
	__init__.py
	admin.py
	migrations/
		__init__.py
	models.py
	tests.py
	views.py
```

这个文件结构是我们要编写的民调应用的基础。

使用Django创建一个基于数据库的网页应用的第一步是定义你的数据模型——必须的数据库结构以及一些附加的元数据(metadata)。

>**思想**
> 模型是你的数据的唯一， 明确的数据源。它包含了你想要存储的数据的必要的类型(fields)以及表现(behaviors)。 Django遵循DRY(Don't Repeat Yourself)原则。 目的就是让你的数据模型能够定义一次并且能多自动从其中生发出许多东西。
> 这包括了数据转移(migrations)， 不像Ruby on Rails那样， 数据转移完全是由你的数据模型中生发出来的， 并且其仅仅是一个历史记录， 以供Django能够升级你的数据库结构， 以满足你现在的数据模型。

在我们的简单的民调应用之中， 我们将创建两种数据模型： 问题与选项。 一个问题的模型包含一个问题，与一个发布日期。 一个选项的模型有两个域， 选项的文字以及一个点票结果。 每个选项都和一个问题相链接：

这些概念都可以由简单的Python类所表示。 依照如下代码修改**polls/models.py**文件：

```python
#polls/models.py
from django.db import models

class Question(models.Model):
	question_text =  models.CharField(max_length=200)
	pub_date = models.DateTimeField('date published')

class Choice(models.Model):
	question = models.ForeignKey(Question)
	choice_text = models.CharField(max_length=200)
	votes = models.IntegerField(defalut=0)
```


这段代码是十分简单直接的。 每个数据模型由一个继承django.db.models.Model的类表示。 每个数据模型有着一系列的类变量， 其每一个都表示着数据模型在数据库中的一个域。

每个域都由一个Field类的示例所表示——举例来说，`CharField`表示字符域而`DateTimeField`表示时间域。 这告诉了Django每一个域其内的数据类型。

每一个Field类的示例的名字（如`question_text`）这个域的名字——以一种电脑能够读懂的形式。 你将会在你的Python代码中使用这个值， 并且你的数据库将会将其作为其对应的列的名称。

你可以使用一个可选的置于首位的参数来指定一个人能够读得懂的名称。 这个用法常见于Django的一些内联(introspective)的部分之中， 并且它在参考文档之中出现的频率很可能翻倍。 如果这个参数并没有被指定， Django会使用机器能够读得懂的名字。 在这个例子中， 我们只为`Question.pub_date`指定了一个人能够读懂的名称， 对于其它的数据域， Django将默认使用其变量名称。

某些Field的类有着必填的参数。 例如`CharField`， 要求你必须提供一个`max_length`参数。 这不仅仅在数据库结构之中使用到。 也在数据验证之中发挥着作用， 我们将会很快见到。

一个Field也可以有着许多可选的参数， 在这个例子中， 我们将`votes`域的default参数设置为0

最后， 请注意通过使用`ForeignKey`， 我们定义了一个数据表之间的关系。这个操作告诉Django每一个Choice与一个单独的Question相对应。 Django支持所有常见的数据库关系， 如：多对一(many-to-one)，多对多(many-to-many)， 一对一(one-to-one)

##激活数据模型

前面那简短的代码给了Django许多有用的信息。 有了他， Django可以：
- 为这个应用创建一个数据库结构
- 为操作Question和Choice对象创建一个Python接入数据库的API

但是首先我们需要告诉我们项目，polls应用已经被安装了。

> **思想**
> Django应用都是“即插即用”的： 你可以把一个应用在多个项目中应用， 你也可以将应用分发给他人使用。 因为他们不必与特定的Django环境相绑定。

再次修改**mysite/settings.py**文件， 修改`INSTALLED_APPS`设置， 把'polls'加进去。 所以它看上去应该是这个样子的：

```
#mysite/settings.py
INSTALLED_APPS = (
	'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'polls',
)
```

现在Django懂得应当把polls应用给包括进来了。 运行另外一个命令：

	$python manage.py makemigrations polls

你将看见与下面类似的一些信息：

```
Migrations for 'polls':
	0001_initial.py:
		- Create model Question
		- Create model Choice
		- Add field question to choice
```

通过运行`makemigrations`， 你告诉Django你对于你的数据模型做了一些变化， 并且你需要将这些变化被作为一个数据转移(migration)存储起来

Django通过储存数据转移的方式将你的数据模型存储起来——他们就是你的磁盘上的一些文件。 如果你想的话， 你可以为你的新的数据模型去阅读这个数据转移文件。 他就是**polls/migrations/0001_initial.py**。 别担心， 并不是每次django创建一个新的文件的时候你都需要去读它， 但是它被设计为可以被人所直接编辑的， 以适应你想要适当人为调整Django的工作机理的情况。

有一个命令， 能够自动的运行数据转移并且处理好你的数据库结构 - 它叫做 `migrate`, 我们稍后将会提到。 但是首先让我们看看数据转移执行了哪些SQL操作。 `sqlmigrate` 命令接收数据转移文件的名称并且返回其SQL语言：

	$ python manage.py sqlmigrate polls 0001

你将看见与如下相似的一些东西：

```sql
BEGIN;
CREATE TABLE polls_question (
    "id" serial NOT NULL PRIMARY KEY,
    "question_text" varchar(200) NOT NULL,
    "pub_date" timestamp with time zone NOT NULL
);

CREATE TABLE polls_choice (
    "id" serial NOT NULL PRIMARY KEY,
    "question_id" integer NOT NULL,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL
);

CREATE INDEX polls_choice_7aa0f6ee ON "polls_choice" ("question_id");

ALTER TABLE "polls_choice"
  ADD CONSTRAINT polls_choice_question_id_246c99a640fbbd72_fk_polls_question_id
    FOREIGN KEY ("question_id")
    REFERENCES "polls_question" ("id")
    DEFERRABLE INITIALLY DEFERRED;
COMMIT;
```

请注意：

- 确切的输出将会取决于你正在使用的数据库引擎。 如上的例子是为PostgreSQL所生成的

-  数据表的名字将有你的app的名字与模块名字的小写自动组合而成。 （你可以通过设置来覆盖这个选项）

- 主键是自动被添加的（你也可以覆盖它）

- 依照惯例， Django在外键域的名字后添加一个 '_id' （你同样也可以覆盖它）

- 外键关系由`FOREIGN_KEY`确定。不用太担心 `DEFERRABLE`部分， 那只是告诉PostgreSQL直到操作结束不要增加外键的值。

- Django自动适配你正在使用的数据库。 所以各数据库专有的特性如`auto_increment`(MySQL), `serial`(PostgreSQL),`integer primary key autoincrement`(SQLite) 都被自动设置好了。 

- `sqlmigrate`命令并不会实际上去运行一个数据转移工作。 它仅仅是把SQL命令显示出来， 让你可以直到实际上发生了什么。 如果你有一个数据库管理员要求知道你对于数据库的操作命令的化， 这将派上不小的用处。

如果你感兴趣的话， 你可以运行`python manage.py check` 命令， 这个命令将会检查你的项目中的问题而不实际去运行数据迁移或者操作数据库。

现在再运行一次 `migrate` 命令， 来为你的数据模型创建数据表

```
$ python manage.py migrate
Operations to perform:
	Apply all migrations: admin, contenttypes, polls, auth, sessions
Running migrations:
	Applying <migration name>... OK
```

`migrate`命令把所有未进行的数据转移操作进行完成。 (Django 通过数据库中一个叫做`django_migrations`的数据表来进行跟踪哪一个数据转移操作未被进行)。 

数据转移是十分强大的。 它允许你能够编写项目的时候适时地更新你的数据模型，而不需要删除你之前的数据库。 这尤其能够帮助你实时更新你的数据库， 而不用失去之前所有的数据。 我们将在之后更深一层的教学之中解释这个问题， 但是现在， 记住一下进行数据模型改动的三部曲：

- 改动你的数据模型(在models.py之中)
- 运行 `python manage.py makemigrations`来为这些改动创建数据转移文件
- 运行 `python manage.py migrage` 来应用这些数据转移

把创建文件与应用更改分开的主要原因是你有可能将数据转移文件上传至你的版本控制系统并且与你的应用一起移动。  这样不仅仅使得你的开发更为简单， 还对于其它的开发者更加方便。

你可以参考 [django-admin.py 文档](https://docs.djangoproject.com/en/1.7/ref/django-admin/)来获得关于manage.py的完整功能的介绍。

##试试API

现在让我们跳入Python Shell的环境中， 试试Django给你的哪些免费（且自由）的API吧。 若要运行Python Shell, 输入如下命令：

	$python manage.py shell

我们使用这个命令， 而非简单地输入 `python`， 原因是manage.py 设置了 `DJANGO_SETTINGS_MODULE`选项， 允许了Python能够直接获得Django的导入地址。

> **除了 manage.py**
> 如果你不想使用manage.py， 也没有问题。 只需设置`DJANGO_SETTINGS_MODULE` 环境变量指向`mysite.settings`,启动一个纯的Python Shell 环境，然后手动设置Django:
```
	>>>import django
	>>>django.setup()
```
> 如果这引发了一个`AttibuteError`， 你很有可能用着一个与此教程不相符的Django版本。 你需要一个更老的教程或者是切换到一个更新的Django。
> 你必须在与manage.py 相同的目录下运行python, 或者确保这个目录在python的默认目录下， 在这种情况下， `import mysite`才能正常工作。
> 关于此的更多信息， 请参见[django-admin.py 的官方文档](https://docs.djangoproject.com/en/1.7/ref/django-admin/)

现在你就已经在Shell环境中了， 来体验一番数据库API把

```python
>>> from polls.model import Question, Choice 
# 导入我们刚刚写的数据模型类
# 目前系统还没有发生任何问题0u0
>>> Questions.objects.all()
[]
#目前Questions中还没有问题
#创建一个新的问题
#关于时差的支持已经在默认的配置文件中， 所以Django需要为pub_date有一个带有时区信息(tzinfo)的时间属性。 使用 timezone.now()而非datetime.datetime.now()将会得到现在正确的时间。
>>> from django.utils import timezone
>>> q = Question(question_text = "What's new?", pub_date = timezone.now())
#将对象保存到数据库中。 你只需要简单明了地调用save()函数
>>> q.save()
#现在这个问题有了它自己的id。 请注意这个id的值有可能是1L或者1， 这取决于你使用的数据库——其实这并不是什么大问题， 它仅仅表明了你的数据库引擎更喜欢使用python的长整形数
>>> q.id
1

#通过Python的属性来访问数据的值
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2015,2,2,13,0,0,775217,tzinfo = <UTC>)

#通过直接改动数据对象的属性来改动值， 然后调用save()函数
>>> q.question_text = "What's up?"
>>> q.save()

#objects.all() 显示数据库中所有的问题
>>> Question.objects.all() 
[<Question: Question object>]
```

等等。 `<Question: Question object>`  是一个很难看，很没有帮助的显示。 然后我们通过修改Question模型来改正它：

```python
#polls/models.py
from django.db import models

class Question(models.Model):
	#...
	def __str__(self):    #在Python2 中是__unicode__
		return self.question_text

class Choice(models.Model):
	#...
	def __str__(self):    #在Python2 中是__unicode__
		return self.choice_text
```

给你的数据模型添加`__str__()`方法至关重要。 不仅仅是为了你自己的方便， 更是因为在Django自动生成的管理员后台中， 这个将作为数据模型的自动表示方式。

这些都只是普通的Python方法。 出于演示考虑，让我们添加一个特殊的方法：

```python
#polls/models.py
import datetime
from django.db import models
from django.utils import timezone

class Question(models.Model):
	#...
	def was_published_recently(self):
		return self.pub_date >= timezone.now() - datetime.timedelta(days-1)
```

注意新增的`import datetime` 和`from django.utils import timezone`选项， 分别参见Python标准的`datetime`模块和Django的时区相关的`django.utils.timezone`功能。 如果你对于时区功能不够了解的话， 请参考[时区支持相关的官方文档](https://docs.djangoproject.com/en/1.7/intro/tutorial01/)

保存这些更改， 并且重新启动一个Python Shell

```python
>>>from polls.models import Question, Choice
#确保我们新增的__str__()工作正常
>>> Questions.objects.all()
[<Question: Whats up>]
#Django提供了一个功能齐全的基于键值的数据库查找API
>>> Question.objects.filter(id=1)
[<Question: Whats up>]
>>> Question.objects.filter(question_text__startswith='What')
[<Question: Whats up>]

#得到当年发布的问题
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
[<Question: Whats up>]

#查找一个并不存在的ID， 将会引发一个异常
>>> Question.objects.get(id=2)
Traceback (most recent call last):
...
DoesNotExist: Question matching query does not exist.

#通过主键进行查找是一个极其常用的操作， 所以Django提供通了一个主键查找的捷径， 以下命令与Question.objects.get(id=1)作用等同
>>>Question.objects.get(pk=1)
[<Question: Whats up>]

#确保我们自己写的方法成功运行

>>>q = Question.objects.get(pk=1)
>>>q.was_published_recently()
True

#给某个问题制定多个选择。 构造器将会创建一个新的Choice对象， 进行INSERT操作。 把选项加入进入并返回新的Choise对象。 Django创建了一个集合去管理外键关系的“另外一边”（例如一个问题的选项）， 它们可以通过API进行访问
>>> q = Question.objects.get(pk=1)

#显示问题相关的所有的选项——目前还一个都没有
>>> q.choice_set.all()
[]

#创建三个选项
>>>q.choice_set.create(choice_text = "Not much", votes=0)
<Choice: Not much>
>>>q.choice_set.create(choice_text = "The sky", votes=0)
<Choice: The sky>
>>>c = q.choice_set.create(choice_text = "Just hacking again", votes=0)

#Choice对象有API接口来访问它们相关的问题
>>> c.question
<Question: Whats up?>

#反之亦然： Question对象也可以访问其相关的选项
>>> q.choice_set.all()
[<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]
>>> q.choice_set.count()
3

#API按你的需要自动维护数据之间的关系。
#使用双下划线来分割关系
#这在多深的层级都可以使用， 没有限制
#下面的代码寻找所有今年发布的问题的所有选项
>>>Choice.objects.filter(question__pub_date__year=current_year)
[<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]

#让我们使用delete()函数删除其中的一个选项
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()
```

