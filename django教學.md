來原：https://docs.djangoproject.com/en/4.2/

（一）--- 3  
（二）--- 276  
（三）--- 321  
 (四) --- 565  
（五）--- 1315  
 (六) --- 1528  
（七）--- 1975  
 (八) --- 2267  
（九）--- 2302  
（十）- 2535

## 初识 Django  
Django 最初被设计用于具有快速开发需求的新闻类站点，目的是要实现简单快捷的网站开发。以下内容简要介绍了如何使用 Django 实现一个数据库驱动的网络应用。

为了让您充分理解 Django 的工作原理，这份文档为您详细描述了相关的技术细节，不过这并不是一份入门教程或者是参考文档（我们当然也为您准备了这些）。

----

## 设计模型  
Django 无需数据库就可以使用，它提供了 对象关系映射器 通过此技术，你可以使用 Python 代码来描述数据库结构。

你可以使用强大的 数据-模型语句 来描述你的数据模型，这解决了数年以来在数据库模式中的难题。以下是一个简明的例子：


mysite/news/models.py
```
from django.db import models


class Reporter(models.Model):
    full_name = models.CharField(max_length=70)

    def __str__(self):
        return self.full_name


class Article(models.Model):
    pub_date = models.DateField()
    headline = models.CharField(max_length=200)
    content = models.TextField()
    reporter = models.ForeignKey(Reporter, on_delete=models.CASCADE)

    def __str__(self):
        return self.headline
```

---

## 应用数据模型  
接下来，运行 Django 命令行实用程序以自动创建数据库表：
```
$ python manage.py makemigrations
$ python manage.py migrate
```
---
该 makemigrations 命令查找所有可用的模型，为任意一个在数据库中不存在对应数据表的模型创建迁移脚本文件。migrate 命令则运行这些迁移来自动创建数据库表

## 享用便捷的 API  

接下来，你就可以使用一套便捷而丰富的 Python API 访问你的数据。这些 API 是即时创建的，而不用显式的生成代码：

```
# Import the models we created from our "news" app
>>> from news.models import Article, Reporter

# No reporters are in the system yet.
>>> Reporter.objects.all()
<QuerySet []>

# Create a new Reporter.
>>> r = Reporter(full_name="John Smith")

# Save the object into the database. You have to call save() explicitly.
>>> r.save()

# Now it has an ID.
>>> r.id
1

# Now the new reporter is in the database.
>>> Reporter.objects.all()
<QuerySet [<Reporter: John Smith>]>

# Fields are represented as attributes on the Python object.
>>> r.full_name
'John Smith'

# Django provides a rich database lookup API.
>>> Reporter.objects.get(id=1)
<Reporter: John Smith>
>>> Reporter.objects.get(full_name__startswith="John")
<Reporter: John Smith>
>>> Reporter.objects.get(full_name__contains="mith")
<Reporter: John Smith>
>>> Reporter.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Reporter matching query does not exist.

# Create an article.
>>> from datetime import date
>>> a = Article(
...     pub_date=date.today(), headline="Django is cool", content="Yeah.", reporter=r
... )
>>> a.save()

# Now the article is in the database.
>>> Article.objects.all()
<QuerySet [<Article: Django is cool>]>

# Article objects get API access to related Reporter objects.
>>> r = a.reporter
>>> r.full_name
'John Smith'

# And vice versa: Reporter objects get API access to Article objects.
>>> r.article_set.all()
<QuerySet [<Article: Django is cool>]>

# The API follows relationships as far as you need, performing efficient
# JOINs for you behind the scenes.
# This finds all articles by a reporter whose name starts with "John".
>>> Article.objects.filter(reporter__full_name__startswith="John")
<QuerySet [<Article: Django is cool>]>

# Change an object by altering its attributes and calling save().
>>> r.full_name = "Billy Goat"
>>> r.save()

# Delete an object with delete().
>>> r.delete()
```
---

## 一个动态管理接口：并非徒有其表  

当你的模型完成定义，Django 就会自动生成一个专业的生产级 管理接口 ——一个允许认证用户添加、更改和删除对象的 Web 站点。你只需在管理站点上注册你的模型即可：

mysite/news/models.py
```
from django.db import models


class Article(models.Model):
    pub_date = models.DateField()
    headline = models.CharField(max_length=200)
    content = models.TextField()
    reporter = models.ForeignKey(Reporter, on_delete=models.CASCADE)
```
---
mysite/news/admin.py
```
from django.contrib import admin

from . import models

admin.site.register(models.Article)
```
---

这样设计所遵循的理念是，站点编辑人员可以是你的员工、你的客户、或者就是你自己——而你大概不会乐意去废半天劲创建一个只有内容管理功能的后台管理界面。

创建 Django 应用的典型流程是，先建立数据模型，然后搭建管理站点，之后你的员工（或者客户）就可以向网站里填充数据了。后面我们会谈到如何展示这些数据。

---

##  规划 URLs  
简洁优雅的 URL 规划对于一个高质量网络应用来说至关重要。Django 推崇优美的 URL 设计，所以不要把诸如 .php 和 .asp 之类的冗余的后缀放到 URL 里。

为了设计你自己的 URLconf ，你需要创建一个叫做 URLconf 的 Python 模块。这是网站的目录，它包含了一张 URL 和 Python 回调函数之间的映射表。URLconf 也有利于将 Python 代码与 URL 进行解耦（译注：使各个模块分离，独立）。

下面这个 URLconf 适用于前面 Reporter/Article 的例子：

mysite/news/urls.py  
```
from django.urls import path

from . import views

urlpatterns = [
    path("articles/<int:year>/", views.year_archive),
    path("articles/<int:year>/<int:month>/", views.month_archive),
    path("articles/<int:year>/<int:month>/<int:pk>/", views.article_detail),
]
```  
上述代码将 URL 路径映射到了 Python 回调函数（“视图”）。路径字符串使用参数标签从URL中“捕获”相应值。当用户请求页面时，Django 依次遍历路径，直至初次匹配到了请求的 URL。(如果无匹配项，Django 会调用 404 视图。) 这个过程非常快，因为路径在加载时就编译成了正则表达式。

一旦有 URL 路径匹配成功，Django 会调用相应的视图函数。每个视图函数会接受一个请求对象——包含请求元信息——以及在匹配式中获取的参数值。

例如，当用户请求了这样的 URL "/articles/2005/05/39323/"，Django 会调用 news.views.article_detail(request, year=2005, month=5, pk=39323)。

---  

##  编写视图  
视图函数的执行结果只可能有两种：返回一个包含请求页面元素的 HttpResponse 对象，或者是抛出 Http404 这类异常。至于执行过程中的其它的动作则由你决定。

通常来说，一个视图的工作就是：从参数获取数据，装载一个模板，然后将根据获取的数据对模板进行渲染。下面是一个 year_archive 的视图样例：  

mysite/news/views.py  
```
from django.shortcuts import render

from .models import Article


def year_archive(request, year):
    a_list = Article.objects.filter(pub_date__year=year)
    context = {"year": year, "article_list": a_list}
    return render(request, "news/year_archive.html", context)

```
这个例子使用了 Django 模板系统 ，它有着很多强大的功能，而且使用起来足够简单，即使不是程序员也可轻松使用。

---

##  设计模板  

上面的代码加载了 news/year_archive.html 模板。  

Django 允许设置搜索模板路径，这样可以最小化模板之间的冗余。在 Django 设置中，你可以通过 DIRS 参数指定一个路径列表用于检索模板。如果第一个路径中不包含任何模板，就继续检查第二个，以此类推。

让我们假设 news/year_archive.html 模板已经找到。它看起来可能是下面这个样子：

mysite/news/templates/news/year_archive.html
```
{% extends "base.html" %}

{% block title %}Articles for {{ year }}{% endblock %}

{% block content %}
<h1>Articles for {{ year }}</h1>

{% for article in article_list %}
    <p>{{ article.headline }}</p>
    <p>By {{ article.reporter.full_name }}</p>
    <p>Published {{ article.pub_date|date:"F j, Y" }}</p>
{% endfor %}
{% endblock %}
```  
我们看到变量都被双大括号括起来了。 {{ article.headline }} 的意思是“输出 article 的 headline 属性值”。这个“点”还有更多的用途，比如查找字典键值、查找索引和函数调用。

我们注意到 {{ article.pub_date|date:"F j, Y" }} 使用了 Unix 风格的“管道符”（“|”字符）。这是一个模板过滤器，用于过滤变量值。在这里过滤器将一个 Python datetime 对象转化为指定的格式（就像 PHP 中的日期函数那样）。

你可以将多个过滤器连在一起使用。你还可以使用你 自定义的模板过滤器 。你甚至可以自己编写 自定义的模板标签 ，相关的 Python 代码会在使用标签时在后台运行。

Django 使用了 ''模板继承'' 的概念。这就是 {% extends "base.html" %} 的作用。它的含义是''先加载名为 base 的模板，并且用下面的标记块对模板中定义的标记块进行填充''。简而言之，模板继承可以使模板间的冗余内容最小化：每个模板只需包含与其它文档有区别的内容。

下面是 base.html 可能的样子，它使用了 静态文件 ：  

mysite/templates/base.html  
```
{% load static %}
<html>
<head>
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
    <img src="{% static 'images/sitelogo.png' %}" alt="Logo">
    {% block content %}{% endblock %}
</body>
</html>
```
简而言之，它定义了这个网站的外观（利用网站的 logo），并且给子模板们挖好了可以填的”坑“。这就意味着你可以通过修改基础模板以达到重新设计网页的目的。

它还可以让你利用不同的基础模板并重用子模板创建一个网站的多个版本。通过创建不同的基础模板，Django 的创建者已经利用这一技术来创造了明显不同的手机版本的网页。

注意，你并不是非得使用 Django 的模板系统，你可以使用其它你喜欢的模板系统。尽管 Django 的模板系统与其模型层能够集成得很好，但这不意味着你必须使用它。同样，你可以不使用 Django 的数据库 API。你可以用其他的数据库抽象层，像是直接读取 XML 文件，亦或直接读取磁盘文件，你可以使用任何方式。Django 的任何组成——模型、视图和模板——都是独立的。

---

##  这仅是基本入门知识  

以上只是 Django 的功能性概述。Django 还有更多实用的特性：

- 缓存框架 可以与 memcached 或其它后端集成。
- 聚合器框架 可以通过编写一个小型 Python 类来创建 RSS 和 Atom 摘要。
- 功能丰富的自动生成的后台——这份概要只是简单介绍了下。

---


（二）--- 276

##  快速安装指南  

### 安装 Python  

作为一个 Python 网络框架，Django 需要 Python。更多细节请参见 我应该使用哪个版本的 Python 来配合 Django?。Python 包含了一个名为 SQLite 的轻量级数据库，所以你暂时不必自行设置一个数据库。

最新版本的 Python 可以通过访问 https://www.python.org/downloads/ 或者操作系统的包管理工具获取。

你可以通过在shell中键入``python``来验证Python是否已安装；你应该看见如下输出：  
```
Python 3.x.y
[GCC 4.x] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```  
---
### 设置数据库  
此步骤仅在你打算使用诸如 PostgreSQL、MariaDB、MySQL 或者 Oracle 这些大型数据库引擎时需要。

---

### 安装 Django  

安装 Django有以下三种方式：

- 安装正式版本 适合大部分用户。  
- 安装 Django 由你的操作系统发行版提供。  
- 安装最新的开发版本 这个选择是针对那些想要体验最新和最好的特性的爱好者们，并不怕运行全新代码。你在开发版中可能会遇到新的 bug，可以报告给社区团队帮助 Django 开发。此外，第三方发行的软件包也可能不与开发版进行兼容

---  

## 验证  
 若要验证 Django 是否能被 Python 识别，可以在 shell 中输入 python。 然后在 Python 提示符下，尝试导入 Django：  
 ```
>>> import django
>>> print(django.get_version())
4.2
 ```  

 当然了，你也可能安装的是其它版本的 Django。

 ---  
（三）--- 321  

## 编写你的第一个 Django 应用，第 1 部分  

让我们通过示例来学习。

通过这个教程，我们将带着你创建一个基本的投票应用程序。

它将由两部分组成：

- 一个让人们查看和投票的公共站点。
- 一个让你能添加、修改和删除投票的管理站点。

我们假定你已经阅读了 安装 Django。你能知道 Django 已被安装，且安装的是哪个版本，通过在命令提示行输入命令（由 $ 前缀）。

```
$ python -m django --version
```  
如果这行命令输出了一个版本号，证明你已经安装了此版本的 Django；如果你得到的是一个“No module named django”的错误提示，则表明你还未安装。

本教程是为Django 4.2 编写的，它支持 Python 3.8 及以后的版本。如果 Django 版本不匹配，你可以通过本页右下角的版本切换器改到你的 Django 版本的教程，或者将 Django 更新到最新的版本  

---  

## 创建项目  

如果这是你第一次使用 Django 的话，你需要一些初始化设置。也就是说，你需要用一些自动生成的代码配置一个 Django project —— 即一个 Django 项目实例需要的设置项集合，包括数据库配置、Django 配置和应用程序配置。

打开命令行，cd 到一个你想放置你代码的目录，然后运行以下命令：  

```
django-admin startproject mysite

```
这行代码将会在当前目录下创建一个 mysite 目录。

```
备注

你得避免使用 Python 或 Django 的内部保留字来命名你的项目。具体地说，你得避免使用像 django (会和 Django 自己产生冲突)或 test (会和 Python 的内置组件产生冲突)这样的名字。
```  
让我们看看 startproject 创建了些什么：  
```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py

```  

这些目录和文件的用处是：  
- 最外层的 mysite/ 根目录只是你项目的容器， 根目录名称对 Django 没有影响，你可以将它重命名为任何你喜欢的名称。
- manage.py: 一个让你用各种方式管理 Django 项目的命令行工具。你可以阅读 django-admin 和 manage.py 获取所有 manage.py 的细节。
- 里面一层的 mysite/ 目录包含你的项目，它是一个纯 Python 包。它的名字就是当你引用它内部任何东西时需要用到的 Python 包名。 (比如 mysite.urls).
- mysite/__init__.py：一个空文件，告诉 Python 这个目录应该被认为是一个 Python 包。如果你是 Python 初学者，阅读官方文档中的 更多关于包的知识。
- mysite/settings.py：Django 项目的配置文件。如果你想知道这个文件是如何工作的，请查看 Django 配置 了解细节。
- mysite/urls.py：Django 项目的 URL 声明，就像你网站的“目录”。阅读 URL调度器 文档来获取更多关于 URL 的内容。
- mysite/asgi.py：作为你的项目的运行在 ASGI 兼容的 Web 服务器上的入口。阅读 如何使用 ASGI 来部署 了解更多细节。
- mysite/wsgi.py：作为你的项目的运行在 WSGI 兼容的Web服务器上的入口。阅读 如何使用 WSGI 进行部署 了解更多细节。

---  

## 用于开发的简易服务器  

 让我们来确认一下你的 Django 项目是否真的创建成功了。如果你的当前目录不是外层的 mysite 目录的话，请切换到此目录，然后运行下面的命令：  

 ```
 $ python manage.py runserver
 ```  
 你应该会看到如下输出：  
 ```Performing system checks...

System check identified no issues (0 silenced).

You have unapplied migrations; your app may not work properly until they are applied.
Run 'python manage.py migrate' to apply them.

九月 16, 2023 - 15:50:53
Django version 4.2, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
 
 ```  
 ```
 备注

忽略有关未应用最新数据库迁移的警告，稍后我们处理数据库。
 ```  

你已经启动了 Django 开发服务器，这是一个用纯 Python 编写的轻量级网络服务器。我们在 Django 中包含了这个服务器，所以你可以快速开发，而不需要处理配置生产服务器的问题 -- 比如 Apache -- 直到你准备好用于生产。

现在是个提醒你的好时机：千万不要 将这个服务器用于和生产环境相关的任何地方。这个服务器只是为了开发而设计的。（我们在网络框架方面是专家，在网络服务器方面并不是。）

服务器现在正在运行，通过浏览器访问 http://127.0.0.1:8000/ 。你将看到一个“祝贺”页面，有一只火箭正在发射。你成功了！

```
更换端口

默认情况下，runserver 命令会将服务器设置为监听本机内部 IP 的 8000 端口。

如果你想更换服务器的监听端口，请使用命令行参数。举个例子，下面的命令会使服务器监听 8080 端口：

$ python manage.py runserver 8080
如果你想要修改服务器监听的IP，在端口之前输入新的。比如，为了监听所有服务器的公开IP（这你运行 Vagrant 或想要向网络上的其它电脑展示你的成果时很有用），使用：

$ python manage.py runserver 0.0.0.0:8000
关于这个简易服务器的完整信息可以在 runserver 文档中找到。

```  
```
会自动重新加载的服务器 runserver

用于开发的服务器在需要的情况下会对每一次的访问请求重新载入一遍 Python 代码。所以你不需要为了让修改的代码生效而频繁的重新启动服务器。然而，一些动作，比如添加新文件，将不会触发自动重新加载，这时你得自己手动重启服务器。
```


---

## 创建投票应用  
现在你的开发环境——这个“项目” ——已经配置好了，你可以开始干活了。

在 Django 中，每一个应用都是一个 Python 包，并且遵循着相同的约定。Django 自带一个工具，可以帮你生成应用的基础目录结构，这样你就能专心写代码，而不是创建目录了。  

```
项目 VS 应用

项目和应用有什么区别？应用是一个专门做某件事的网络应用程序——比如博客系统，或者公共记录的数据库，或者小型的投票程序。项目则是一个网站使用的配置和应用的集合。项目可以包含很多个应用。应用可以被很多个项目使用。
```

你的应用可以存放在任何 Python 路径 中定义的路径。在这个教程中，我们将在你的 manage.py 同级目录下创建投票应用。这样它就可以作为顶级模块导入，而不是 mysite 的子模块。

请确定你现在处于 manage.py 所在的目录下，然后运行这行命令来创建一个应用：  

```
 python manage.py startapp polls
```  
這將創建一個目錄 polls，其佈局如下：  
```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```  
这个目录结构包括了投票应用的全部内容。  

---

##  编写第一个视图  

让我们开始编写第一个视图吧。打开 polls/views.py，把下面这些 Python 代码输入进去：  
polls/views.py
```
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
这是 Django 中最简单的视图。如果想看见效果，我们需要将一个 URL 映射到它——这就是我们需要 URLconf 的原因了。

要在 polls 目錄中創建 URLconf，請創建一個名為 urls.py 的文件。 您的應用程序目錄現在應如下所示：  
```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    urls.py
    views.py
```  
在 polls/urls.py 中，输入如下代码：  
polls/urls.py
```
from django.urls import path

from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```  
下一步是要在根 URLconf 文件中指定我们创建的 polls.urls 模块。在 mysite/urls.py 文件的 urlpatterns 列表里插入一个 include()， 如下：  
mysite/urls.py  
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("polls/", include("polls.urls")),
    path("admin/", admin.site.urls),
]
```
函数 include() 允许引用其它 URLconfs。每当 Django 遇到 include() 时，它会截断与此项匹配的 URL 的部分，并将剩余的字符串发送到 URLconf 以供进一步处理。

我们设计 include() 的理念是使其可以即插即用。因为投票应用有它自己的 URLconf( polls/urls.py )，他们能够被放在 "/polls/" ， "/fun_polls/" ，"/content/polls/"，或者其他任何路径下，这个应用都能够正常工作。
```
何时使用 include()

当包括其它 URL 模式时你应该总是使用 include() ， admin.site.urls 是唯一例外。
```  
你现在把 index 视图添加进了 URLconf。通过以下命令验证是否正常工作：

```
$ python manage.py runserver
```
用你的浏览器访问 http://localhost:8000/polls/，你应该能够看见 "Hello, world. You're at the polls index." ，这是你在 index 视图中定义的。  

```
没有找到页面?

如果你在这里得到了一个错误页面，检查一下你是不是正访问着http://localhost:8000/polls/ 而不应该是 http://localhost:8000/。
```  


函数 path() 具有四个参数，两个必须参数：route 和 view，两个可选参数：kwargs 和 name。现在，是时候来研究这些参数的含义了。

### path() 参数： route  
route 是一个匹配 URL 的准则（类似正则表达式）。当 Django 响应一个请求时，它会从 urlpatterns 的第一项开始，按顺序依次匹配列表中的项，直到找到匹配的项。

这些准则不会匹配 GET 和 POST 参数或域名。例如，URLconf 在处理请求 https://www.example.com/myapp/ 时，它会尝试匹配 myapp/ 。处理请求 https://www.example.com/myapp/?page=3 时，也只会尝试匹配 myapp/。

### path() 参数： view  
当 Django 找到了一个匹配的准则，就会调用这个特定的视图函数，并传入一个 HttpRequest 对象作为第一个参数，被“捕获”的参数以关键字参数的形式传入。稍后，我们会给出一个例子。

### path() 参数： kwargs  
任意个关键字参数可以作为一个字典传递给目标视图函数。本教程中不会使用这一特性。

### path() 参数： name  
为你的 URL 取名能使你在 Django 的任意地方唯一地引用它，尤其是在模板中。这个有用的特性允许你只改一个文件就能全局地修改某个 URL 模式。

---  

(四)- 565

##  编写你的第一个 Django 应用，第 2 部分  

我们将设置数据库，创建第一个模型，并快速介绍 Django 自动生成的后台界面。

### 数据库配置  

现在，打开 mysite/settings.py 。这是个包含了 Django 项目设置的 Python 模块。

通常，这个配置文件使用 SQLite 作为默认数据库。如果你不熟悉数据库，或者只是想尝试下 Django，这是最简单的选择。Python 内置 SQLite，所以你无需安装额外东西来使用它。当你开始一个真正的项目时，你可能更倾向使用一个更具扩展性的数据库，例如 PostgreSQL，避免中途切换数据库这个令人头疼的问题。

如果你想使用其他数据库，你需要安装合适的 database bindings ，然后改变设置文件中 DATABASES 'default' 项目中的一些键值：  

-  ENGINE -- 可选值有 'django.db.backends.sqlite3'，'django.db.backends.postgresql'，'django.db.backends.mysql'，或 'django.db.backends.oracle'。其它 可用后端。  
- NAME -- 数据库的名称。如果你使用 SQLite，数据库将是你电脑上的一个文件，在这种情况下，NAME 应该是此文件完整的绝对路径，包括文件名。默认值 BASE_DIR / 'db.sqlite3' 将把数据库文件储存在项目的根目录。  

如果你不使用 SQLite，则必须添加一些额外设置，比如 USER 、 PASSWORD 、 HOST 等等。想了解更多数据库设置方面的内容，请看文档：DATABASES 。

```
SQLite 以外的其它数据库

如果你使用了 SQLite 以外的数据库，请确认在使用前已经创建了数据库。你可以通过在你的数据库交互式命令行中使用 "CREATE DATABASE database_name;" 命令来完成这件事。

另外，还要确保该数据库用户中提供 mysite/settings.py 具有 "create database" 权限。这使得自动创建的 test database 能被以后的教程使用。

如果你使用 SQLite，那么你不需要在使用前做任何事——数据库会在需要的时候自动创建。
```  


编辑 mysite/settings.py 文件前，先设置 TIME_ZONE 为你自己时区。

此外，关注一下文件头部的 INSTALLED_APPS 设置项。这里包括了会在你项目中启用的所有 Django 应用。应用能在多个项目中使用，你也可以打包并且发布应用，让别人使用它们。

通常， INSTALLED_APPS 默认包括了以下 Django 的自带应用：  

- django.contrib.admin -- 管理员站点， 你很快就会使用它。
- django.contrib.auth -- 认证授权系统。
- django.contrib.contenttypes -- 内容类型框架。
- django.contrib.sessions -- 会话框架。
- django.contrib.messages -- 消息框架。
- django.contrib.staticfiles -- 管理静态文件的框架。  

这些应用被默认启用是为了给常规项目提供方便。

默认开启的某些应用需要至少一个数据表，所以，在使用他们之前需要在数据库中创建一些表。请执行以下命令：

```
python manage.py migrate
```  
这个 migrate 命令查看 INSTALLED_APPS 配置，并根据 mysite/settings.py 文件中的数据库配置和随应用提供的数据库迁移文件（我们将在后面介绍这些），创建任何必要的数据库表。你会看到它应用的每一个迁移都有一个消息。如果你有兴趣，运行你的数据库的命令行客户端，输入 \dt （PostgreSQL）， SHOW TABLES; （MariaDB，MySQL）， .tables （SQLite）或 SELECT TABLE_NAME FROM USER_TABLES; （Oracle）来显示 Django 创建的表。  
```
写给极简主义者

就像之前说的，为了方便大多数项目，我们默认激活了一些应用，但并不是每个人都需要它们。如果你不需要某个或某些应用，你可以在运行 migrate 前毫无顾虑地从 INSTALLED_APPS 里注释或者删除掉它们。 migrate 命令只会为在 INSTALLED_APPS 里声明了的应用进行数据库迁移。
```
---

## 创建模型  


在 Django 里写一个数据库驱动的 Web 应用的第一步是定义模型 - 也就是数据库结构设计和附加的其它元数据。

```
设计哲学

一个模型就是单个定义你的数据的信息源。模型中包含了不可缺少的数据区域和你存储数据的行为。Django 遵循 DRY 原则。目的就是定义你的数据模型要在一位置上，而且自动从该位置推导一些事情。

来介绍一下迁移 - 举个例子，不像 Ruby On Rails，Django 的迁移代码是由你的模型文件自动生成的，它本质上是个历史记录，Django 可以用它来进行数据库的滚动更新，通过这种方式使其能够和当前的模型匹配。

```  
在这个投票应用中，需要创建两个模型：问题 Question 和选项 Choice。Question 模型包括问题描述和发布时间。Choice 模型有两个字段，选项描述和当前得票数。每个选项属于一个问题。

这些概念可以通过一个 Python 类来描述。按照下面的例子来编辑 polls/models.py 文件：  

polls/models.py  
```
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```
每个模型被表示为 django.db.models.Model 类的子类。每个模型有许多类变量，它们都表示模型里的一个数据库字段。

每个字段都是 Field 类的实例 - 比如，字符字段被表示为 CharField ，日期时间字段被表示为 DateTimeField 。这将告诉 Django 每个字段要处理的数据类型。

每个 Field 类实例变量的名字（例如 question_text 或 pub_date ）也是字段名，所以最好使用对机器友好的格式。你将会在 Python 代码里使用它们，而数据库会将它们作为列名。

你可以使用可选的选项来为 Field 定义一个人类可读的名字。这个功能在很多 Django 内部组成部分中都被使用了，而且作为文档的一部分。如果某个字段没有提供此名称，Django 将会使用对机器友好的名称，也就是变量名。在上面的例子中，我们只为 Question.pub_date 定义了对人类友好的名字。对于模型内的其它字段，它们的机器友好名也会被作为人类友好名使用。

定义某些 Field 类实例需要参数。例如 CharField 需要一个 max_length 参数。这个参数的用处不止于用来定义数据库结构，也用于验证数据，我们稍后将会看到这方面的内容。

Field 也能够接收多个可选参数；在上面的例子中：我们将 votes 的 default 也就是默认值，设为0。

注意在最后，我们使用 ForeignKey 定义了一个关系。这将告诉 Django，每个 Choice 对象都关联到一个 Question 对象。Django 支持所有常用的数据库关系：多对一、多对多和一对一。

---  

## 激活模型  
上面的一小段用于创建模型的代码给了 Django 很多信息，通过这些信息，Django 可以：

- 为这个应用创建数据库 schema（生成 CREATE TABLE 语句）。
- 创建可以与 Question 和 Choice 对象进行交互的 Python 数据库 API。
但是首先得把 polls 应用安装到我们的项目里。

```
设计哲学

Django 应用是“可插拔”的。你可以在多个项目中使用同一个应用。除此之外，你还可以发布自己的应用，因为它们并不会被绑定到当前安装的 Django 上。
```
为了在我们的工程中包含这个应用，我们需要在配置类 INSTALLED_APPS 中添加设置。因为 PollsConfig 类写在文件 polls/apps.py 中，所以它的点式路径是 'polls.apps.PollsConfig'。在文件 mysite/settings.py 中 INSTALLED_APPS 子项添加点式路径后，它看起来像这样：
mysite/settings.py
```
INSTALLED_APPS = [
    "polls.apps.PollsConfig",
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]

```  
现在你的 Django 项目会包含 polls 应用。接着运行下面的命令： 
```
$ python manage.py makemigrations polls
```
你将会看到类似于下面这样的输出：  
```
Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice
```  

通过运行 makemigrations 命令，Django 会检测你对模型文件的修改（在这种情况下，你已经取得了新的），并且把修改的部分储存为一次 迁移。

迁移是 Django 对于模型定义（也就是你的数据库结构）的变化的储存形式 - 它们其实也只是一些你磁盘上的文件。如果你想的话，你可以阅读一下你模型的迁移数据，它被储存在 polls/migrations/0001_initial.py 里。别担心，你不需要每次都阅读迁移文件，但是它们被设计成人类可读的形式，这是为了便于你手动调整 Django 的修改方式。

Django 有一个自动执行数据库迁移并同步管理你的数据库结构的命令 - 这个命令是 migrate，我们马上就会接触它 - 但是首先，让我们看看迁移命令会执行哪些 SQL 语句。sqlmigrate 命令接收一个迁移的名称，然后返回对应的 SQL:  

```
$ python manage.py sqlmigrate polls 0001
```  
你将会看到类似下面这样的输出（我把输出重组成了人类可读的格式）：  
```
BEGIN;
--
-- Create model Question
--
CREATE TABLE "polls_question" (
    "id" bigint NOT NULL PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
    "question_text" varchar(200) NOT NULL,
    "pub_date" timestamp with time zone NOT NULL
);
--
-- Create model Choice
--
CREATE TABLE "polls_choice" (
    "id" bigint NOT NULL PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL,
    "question_id" bigint NOT NULL
);
ALTER TABLE "polls_choice"
  ADD CONSTRAINT "polls_choice_question_id_c5b4b260_fk_polls_question_id"
    FOREIGN KEY ("question_id")
    REFERENCES "polls_question" ("id")
    DEFERRABLE INITIALLY DEFERRED;
CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");

COMMIT;
```  
请注意以下几点：  
- 输出的内容和你使用的数据库有关，上面的输出示例使用的是 PostgreSQL。
- 数据库的表名是由应用名(polls)和模型名的小写形式( question 和 choice)连接而来。（如果需要，你可以自定义此行为。）
- 主键(IDs)会被自动创建。(当然，你也可以自定义。)
- 默认的，Django 会在外键字段名后追加字符串 "_id" 。（同样，这也可以自定义。）
- 外键关系由 FOREIGN KEY 生成。你不用关心 DEFERRABLE 部分，它只是告诉 PostgreSQL，请在事务全都执行完之后再创建外键关系。
- 它是为你正在使用的数据库定制的，因此特定于数据库的字段类型，例如“auto_increment”（MySQL）、“bigint PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY”（PostgreSQL）或“integer primary key autoincrement” `` (SQLite) 会自动为您处理。 字段名称的引用也是如此——例如，使用双引号或单引号。
- 这个 sqlmigrate 命令并没有真正在你的数据库中的执行迁移 - 相反，它只是把命令输出到屏幕上，让你看看 Django 认为需要执行哪些 SQL 语句。这在你想看看 Django 到底准备做什么，或者当你是数据库管理员，需要写脚本来批量处理数据库时会很有用。

如果你感兴趣，你也可以试试运行 python manage.py check ;这个命令帮助你检查项目中的问题，并且在检查过程中不会对数据库进行任何操作。

现在，再次运行 migrate 命令，在数据库里创建新定义的模型的数据表：

```
$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Rendering model states... DONE
  Applying polls.0001_initial... OK
```  
这个 migrate 命令选中所有还没有执行过的迁移（Django 通过在数据库中创建一个特殊的表 django_migrations 来跟踪执行过哪些迁移）并应用在数据库上 - 也就是将你对模型的更改同步到数据库结构上。

迁移是非常强大的功能，它能让你在开发过程中持续的改变数据库结构而不需要重新删除和创建表 - 它专注于使数据库平滑升级而不会丢失数据。我们会在后面的教程中更加深入的学习这部分内容，现在，你只需要记住，改变模型需要这三步：  

- 编辑 models.py 文件，改变模型。
- 运行 python manage.py makemigrations 为模型的改变生成迁移文件。
- 运行 python manage.py migrate 来应用数据库迁移。  

数据库迁移被分解成生成和应用两个命令是为了让你能够在代码控制系统上提交迁移数据并使其能在多个应用里使用；这不仅仅会让开发更加简单，也给别的开发者和生产环境中的使用带来方便。

---


##  初试 API  

现在让我们进入交互式 Python 命令行，尝试一下 Django 为你创建的各种 API。通过以下命令打开 Python 命令行：  

```
python manage.py shell
```
我们使用这个命令而不是简单的使用“python”是因为 manage.py 会设置 DJANGO_SETTINGS_MODULE 环境变量，这个变量会让 Django 根据 mysite/settings.py 文件来设置 Python 包的导入路径。

進入 shell 後，探索數據庫 API：  
```
>>> from polls.models import Choice, Question  # Import the model classes we just wrote.

# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# Create a new Question.
# Support for time zones is enabled in the default settings file, so
# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
# instead of datetime.datetime.now() and it will do the right thing.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=datetime.timezone.utc)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```
等等。<Question: Question object (1)> 对于我们了解这个对象的细节没什么帮助。让我们通过编辑 Question 模型的代码（位于 polls/models.py 中）来修复这个问题。给 Question 和 Choice 增加 __str__() 方法。  

polls/models.py
```
from django.db import models


class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text


class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```  
给模型增加 __str__() 方法是很重要的，这不仅仅能给你在命令行里使用带来方便，Django 自动生成的 admin 里也使用这个方法来表示对象。

让我们再为此模型添加一个自定义方法：
polls/models.py
```
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```  
新加入的 import datetime 和 from django.utils import timezone 分别导入了 Python 的标准 datetime 模块和 Django 中和时区相关的 django.utils.timezone 工具模块。

保存這些更改並再次運行 python manage.py shell 啟動新的 Python 交互式 shell：

```
>>> from polls.models import Choice, Question

# Make sure our __str__() addition worked.
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Django provides a rich database lookup API that's entirely driven by
# keyword arguments.
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith="What")
<QuerySet [<Question: What's up?>]>

# Get the question that was published this year.
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# Request an ID that doesn't exist, this will raise an exception.
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# Lookup by a primary key is the most common case, so Django provides a
# shortcut for primary-key exact lookups.
# The following is identical to Question.objects.get(id=1).
>>> Question.objects.get(pk=1)
<Question: What's up?>

# Make sure our custom method worked.
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# Give the Question a couple of Choices. The create call constructs a new
# Choice object, does the INSERT statement, adds the choice to the set
# of available choices and returns the new Choice object. Django creates
# a set to hold the "other side" of a ForeignKey relation
# (e.g. a question's choice) which can be accessed via the API.
>>> q = Question.objects.get(pk=1)

# Display any choices from the related object set -- none so far.
>>> q.choice_set.all()
<QuerySet []>

# Create three choices.
>>> q.choice_set.create(choice_text="Not much", votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text="The sky", votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text="Just hacking again", votes=0)

# Choice objects have API access to their related Question objects.
>>> c.question
<Question: What's up?>

# And vice versa: Question objects get access to Choice objects.
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# The API automatically follows relationships as far as you need.
# Use double underscores to separate relationships.
# This works as many levels deep as you want; there's no limit.
# Find all Choices for any question whose pub_date is in this year
# (reusing the 'current_year' variable we created above).
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# Let's delete one of the choices. Use delete() for that.
>>> c = q.choice_set.filter(choice_text__startswith="Just hacking")
>>> c.delete()

```

---

## 介绍 Django 管理页面  

```
设计哲学

为你的员工或客户生成一个用户添加，修改和删除内容的后台是一项缺乏创造性和乏味的工作。因此，Django 全自动地根据模型创建后台界面。

Django 产生于一个公众页面和内容发布者页面完全分离的新闻类站点的开发过程中。站点管理人员使用管理系统来添加新闻、事件和体育时讯等，这些添加的内容被显示在公众页面上。Django 通过为站点管理人员创建统一的内容编辑界面解决了这个问题。

管理界面不是为了网站的访问者，而是为管理者准备的。
```
创建一个管理员账号
首先，我们得创建一个能登录管理页面的用户。请运行下面的命令：
```
python manage.py createsuperuser
```  
键入你想要使用的用户名，然后按下回车键：
```
Username: admin
```
然后提示你输入想要使用的邮件地址：  
```
Email address: admin@example.com
```  
最后一步是输入密码。你会被要求输入两次密码，第二次的目的是为了确认第一次输入的确实是你想要的密码。
```
Password: **********
Password (again): *********
Superuser created successfully.
```

---

## 启动开发服务器  
Django 的管理界面默认就是启用的。让我们启动开发服务器，看看它到底是什么样的。

如果开发服务器未启动，用以下命令启动它：
```
python manage.py runserver
```
现在，打开浏览器，转到你本地域名的 “/admin/” 目录， -- 比如 http://127.0.0.1:8000/admin/ 。你应该会看见管理员登录界面：
<img src="https://docs.djangoproject.com/en/4.2/_images/admin01.png">  
因为 翻译 功能默认是开启的，如果你设置了 LANGUAGE_CODE，登录界面将显示你设置的语言（如果 Django 有相应的翻译）。

## 进入管理站点页面  
现在，试着使用你在上一步中创建的超级用户来登录。然后你将会看到 Django 管理页面的索引页：
<img src="https://docs.djangoproject.com/en/4.2/_images/admin02.png">  
你将会看到几种可编辑的内容：组和用户。它们是由 django.contrib.auth 提供的，这是 Django 开发的认证框架。  

## 向管理页面中加入投票应用  
但是我们的投票应用在哪呢？它没在索引页面里显示。

只需要再做一件事：我们得告诉管理，问题 Question 对象需要一个后台接口。打开 polls/admin.py 文件，把它编辑成下面这样：

polls/admin.py
```
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```
## 体验便捷的管理功能  
现在我们向管理页面注册了问题 Question 类。Django 知道它应该被显示在索引页里：
<img src="file:///home/user/%E4%B8%8B%E8%BC%89/django-docs-4.2-zh-hans/_images/admin03t.png">  
点击 "Questions" 。现在看到是问题 "Questions" 对象的列表 "change list" 。这个界面会显示所有数据库里的问题 Question 对象，你可以选择一个来修改。这里现在有我们在上一部分中创建的 “What's up?” 问题。
<img src="file:///home/user/%E4%B8%8B%E8%BC%89/django-docs-4.2-zh-hans/_images/admin04t.png">
点击 “What's up?” 来编辑这个问题（Question）对象：
<img src="file:///home/user/%E4%B8%8B%E8%BC%89/django-docs-4.2-zh-hans/_images/admin05t.png">
注意事项：
- 这个表单是从问题 Question 模型中自动生成的
- 不同的字段类型（日期时间字段 DateTimeField 、字符字段 CharField）会生成对应的 HTML 输入控件。每个类型的字段都知道它们该如何在管理页面里显示自己。
- 每个日期时间字段 DateTimeField 都有 JavaScript 写的快捷按钮。日期有转到今天（Today）的快捷按钮和一个弹出式日历界面。时间有设为现在（Now）的快捷按钮和一个列出常用时间的方便的弹出式列表。  
页面的底部提供了几个选项：
- 保存（Save） - 保存改变，然后返回对象列表。
- 保存并继续编辑（Save and continue editing） - 保存改变，然后重新载入当前对象的修改界面。
- 保存并新增（Save and add another） - 保存改变，然后添加一个新的空对象并载入修改界面。
- 删除（Delete） - 显示一个确认删除页面。  
如果显示的 “发布日期(Date Published)” 和你在 教程 1 里创建它们的时间不一致，这意味着你可能没有正确的设置 TIME_ZONE 。改变设置，然后重新载入页面看看是否显示了正确的值。

通过点击 “今天(Today)” 和 “现在(Now)” 按钮改变 “发布日期(Date Published)”。然后点击 “保存并继续编辑(Save and add another)”按钮。然后点击右上角的 “历史(History)”按钮。你会看到一个列出了所有通过 Django 管理页面对当前对象进行的改变的页面，其中列出了时间戳和进行修改操作的用户名：  
<img src="file:///home/user/%E4%B8%8B%E8%BC%89/django-docs-4.2-zh-hans/_images/admin06t.png">


（五）--- 1030  
## 编写你的第一个 Django 应用，第 3 部分  

我们将继续开发网络投票应用程序，并将着重于创建公共接口——“视图”。

## 概况  
Django 中的视图的概念是「一类具有相同功能和模板的网页的集合」。比如，在一个博客应用中，你可能会创建如下几个视图：
- 博客首页——展示最近的几项内容。
- 内容“详情”页——详细展示某项内容。
- 以年为单位的归档页——展示选中的年份里各个月份创建的内容。
- 以月为单位的归档页——展示选中的月份里各天创建的内容。
- 以天为单位的归档页——展示选中天里创建的所有内容。
- 评论处理器——用于响应为一项内容添加评论的操作。
而在我们的投票应用中，我们需要下列几个视图：
- 问题索引页——展示最近的几个投票问题。
- 问题详情页——展示某个投票的问题和不带结果的选项列表。
- 问题结果页——展示某个投票的结果。
- 投票处理器——用于响应用户为某个问题的特定选项投票的操作。

在 Django 中，网页和其他内容都是从视图派生而来。每一个视图表现为一个 Python 函数（或者说方法，如果是在基于类的视图里的话）。Django 将会根据用户请求的 URL 来选择使用哪个视图（更准确的说，是根据 URL 中域名之后的部分）

在你上网的过程中，很可能看见过像这样美丽的 URL：ME2/Sites/dirmod.htm?sid=&type=gen&mod=Core+Pages&gid=A6CD4967199A42D9B65B1B。别担心，Django 里的 URL 样式 要比这优雅的多！

URL 样式是 URL 的一般形式 - 例如：/newsarchive/<year>/<month>/。

为了将 URL 和视图关联起来，Django 使用了 'URLconfs' 来配置。URLconf 将 URL 模式映射到视图。

本教程只会介绍 URLconf 的基础内容，你可以看看 URL调度器 以获取更多内容。

## 编写更多视图  
现在让我们向 polls/views.py 里添加更多视图。这些视图有一些不同，因为他们接收参数：
polls/views.py
```
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)


def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)


def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```
把这些新视图添加进 polls.urls 模块里，只要添加几个 url() 函数调用就行：
polls/urls.py
```
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path("", views.index, name="index"),
    # ex: /polls/5/
    path("<int:question_id>/", views.detail, name="detail"),
    # ex: /polls/5/results/
    path("<int:question_id>/results/", views.results, name="results"),
    # ex: /polls/5/vote/
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```
然后看看你的浏览器，如果你转到 "/polls/34/" ，Django 将会运行 detail() 方法并且展示你在 URL 里提供的问题 ID。再试试 "/polls/34/vote/" 和 "/polls/34/vote/" ——你将会看到暂时用于占位的结果和投票页。

當有人從您的網站請求頁面時（例如“/polls/34/”），Django 將加載 mysite.urls Python 模塊，因為它是由 ROOT_URLCONF 設置指向的。 它找到名為 urlpatterns 的變量並按順序遍歷模式。 在“polls/”處找到匹配項後，它會刪除匹配文本（“polls/”）並將剩余文本（“34/”）發送到“polls.urls”URLconf 進行進一步處理。 它與 '<int:question_id>/' 匹配，從而導致調用Detail() 視圖，如下所示：

```
detail(request=<HttpRequest object>, question_id=34)
```

问题 question_id=34 来自 <int:question_id>。使用尖括号 "获得" 网址部分后发送给视图函数作为一个关键字参数。字符串的 question_id 部分定义了要使用的名字，用来识别相匹配的模式，而 int 部分是一种转换形式，用来确定应该匹配网址路径的什么模式。冒号 (:) 用来分隔转换形式和模式名。

## 写一个真正有用的视图  

每个视图必须要做的只有两件事：返回一个包含被请求页面内容的 HttpResponse 对象，或者抛出一个异常，比如 Http404 。至于你还想干些什么，随便你。

你的视图可以从数据库里读取记录，可以使用一个模板引擎（比如 Django 自带的，或者其他第三方的），可以生成一个 PDF 文件，可以输出一个 XML，创建一个 ZIP 文件，你可以做任何你想做的事，使用任何你想用的 Python 库。

Django 只要求返回的是一个 HttpResponse ，或者抛出一个异常。

因为 Django 自带的数据库 API 很方便，我们曾在 教程第 2 部分 中学过，所以我们试试在视图里使用它。我们在 index() 函数里插入了一些新内容，让它能展示数据库里以发布日期排序的最近 5 个投票问题，以空格分割：  
polls/views.py
```
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    output = ", ".join([q.question_text for q in latest_question_list])
    return HttpResponse(output)


# Leave the rest of the views (detail, results, vote) unchanged

```  
这里有个问题：页面的设计写死在视图函数的代码里的。如果你想改变页面的样子，你需要编辑 Python 代码。所以让我们使用 Django 的模板系统，只要创建一个视图，就可以将页面的设计从代码中分离出来。

首先，在你的 polls 目录里创建一个 templates 目录。Django 将会在这个目录里查找模板文件。

你项目的 TEMPLATES 配置项描述了 Django 如何载入和渲染模板。默认的设置文件设置了 DjangoTemplates 后端，并将 APP_DIRS 设置成了 True。这一选项将会让 DjangoTemplates 在每个 INSTALLED_APPS 文件夹中寻找 "templates" 子目录。这就是为什么尽管我们没有像在第二部分中那样修改 DIRS 设置，Django 也能正确找到 polls 的模板位置的原因。

在你刚刚创建的 templates 目录里，再创建一个目录 polls，然后在其中新建一个文件 index.html 。换句话说，你的模板文件的路径应该是 polls/templates/polls/index.html 。因为``app_directories`` 模板加载器是通过上述描述的方法运行的，所以 Django 可以引用到 polls/index.html 这一模板了。

```
模板命名空间

虽然我们现在可以将模板文件直接放在 polls/templates 文件夹中（而不是再建立一个 polls 子文件夹），但是这样做不太好。Django 将会选择第一个匹配的模板文件，如果你有一个模板文件正好和另一个应用中的某个模板文件重名，Django 没有办法 区分 它们。我们需要帮助 Django 选择正确的模板，最好的方法就是把他们放入各自的 命名空间 中，也就是把这些模板放入一个和 自身 应用重名的子文件夹里。

```
将下面的代码输入到刚刚创建的模板文件中：
polls/templates/polls/index.html
```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```  
然后，让我们更新一下 polls/views.py 里的 index 视图来使用模板：
polls/views.py
```
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    template = loader.get_template("polls/index.html")
    context = {
        "latest_question_list": latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```
上述代码的作用是，载入 polls/index.html 模板文件，并且向它传递一个上下文(context)。这个上下文是一个字典，它将模板内的变量映射为 Python 对象。

用你的浏览器访问 "/polls/" ，你将会看见一个无序列表，列出了我们在 教程第 2 部分 中添加的 “What's up” 投票问题，链接指向这个投票的详情页。

## 一个快捷函数： render()  
「载入模板，填充上下文，再返回由它生成的 HttpResponse 对象」是一个非常常用的操作流程。于是 Django 提供了一个快捷函数，我们用它来重写 index() 视图：
polls/views.py
```
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    context = {"latest_question_list": latest_question_list}
    return render(request, "polls/index.html", context)
```
注意到，我们不再需要导入 loader 和 HttpResponse 。不过如果你还有其他函数（比如说 detail, results, 和 vote ）需要用到它的话，就需要保持 HttpResponse 的导入。

The render() function takes the request object as its first argument, a template name as its second argument and a dictionary as its optional third argument. It returns an HttpResponse object of the given template rendered with the given context.

##  抛出 404 错误  

现在，我们来处理投票详情视图——它会显示指定投票的问题标题。下面是这个视图的代码：
polls/views.py
```
from django.http import Http404
from django.shortcuts import render

from .models import Question


# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, "polls/detail.html", {"question": question})
```
这里有个新原则。如果指定问题 ID 所对应的问题不存在，这个视图就会抛出一个 Http404 异常。

我们稍后再讨论你需要在 polls/detail.html 里输入什么，但是如果你想试试上面这段代码是否正常工作的话，你可以暂时把下面这段输进去：
polls/templates/polls/detail.html
```
{{ question }}
```
这样你就能测试了。

## 一个快捷函数： get_object_or_404()  
尝试用 get() 函数获取一个对象，如果不存在就抛出 Http404 错误也是一个普遍的流程。Django 也提供了一个快捷函数，下面是修改后的详情 detail() 视图代码：

polls/views.py
```
from django.shortcuts import get_object_or_404, render

from .models import Question


# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/detail.html", {"question": question})
```
get_object_or_404() 函數將 Django 模型作為其第一個參數和任意數量的關鍵字參數，並將其傳遞給模型管理器的 get() 函數。 如果該對像不存在，則會引發 Http404。

```
设计哲学

为什么我们使用辅助函数 get_object_or_404() 而不是自己捕获 ObjectDoesNotExist 异常呢？还有，为什么模型 API 不直接抛出 ObjectDoesNotExist 而是抛出 Http404 呢？

因为这样做会增加模型层和视图层的耦合性。指导 Django 设计的最重要的思想之一就是要保证松散耦合。一些受控的耦合将会被包含在 django.shortcuts 模块中。
```
也有 get_list_or_404() 函数，工作原理和 get_object_or_404() 一样，除了 get() 函数被换成了 filter() 函数。如果列表为空的话会抛出 Http404 异常。

## 使用模板系统  
回过头去看看我们的 detail() 视图。它向模板传递了上下文变量 question 。下面是 polls/detail.html 模板里正式的代码：
polls/templates/polls/detail.html
```
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```
模板系统统一使用点符号来访问变量的属性。在示例 {{ question.question_text }} 中，首先 Django 尝试对 question 对象使用字典查找（也就是使用 obj.get(str) 操作），如果失败了就尝试属性查找（也就是 obj.str 操作），结果是成功了。如果这一操作也失败的话，将会尝试列表查找（也就是 obj[int] 操作）。

在 {% for %} 循环中发生的函数调用：question.choice_set.all 被解释为 Python 代码 question.choice_set.all() ，将会返回一个可迭代的 Choice 对象，这一对象可以在 {% for %} 标签内部使用。

## 去除模板中的硬编码 URL  
还记得吗，我们在 polls/index.html 里编写投票链接时，链接是硬编码的：
```
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```
问题在于，硬编码和强耦合的链接，对于一个包含很多应用的项目来说，修改起来是十分困难的。然而，因为你在 polls.urls 的 url() 函数中通过 name 参数为 URL 定义了名字，你可以使用 {% url %} 标签代替它：
```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```
这个标签的工作方式是在 polls.urls 模块的 URL 定义中寻具有指定名字的条目。你可以回忆一下，具有名字 'detail' 的 URL 是在如下语句中定义的：
```
...
# the 'name' value as called by the {% url %} template tag
path("<int:question_id>/", views.detail, name="detail"),
...
```
如果你想改变投票详情视图的 URL，比如想改成 polls/specifics/12/ ，你不用在模板里修改任何东西（包括其它模板），只要在 polls/urls.py 里稍微修改一下就行：
```
...
# added the word 'specifics'
path("specifics/<int:question_id>/", views.detail, name="detail"),
...
```
##  为 URL 名称添加命名空间  
教程项目只有一个应用，polls 。在一个真实的 Django 项目中，可能会有五个，十个，二十个，甚至更多应用。Django 如何分辨重名的 URL 呢？举个例子，polls 应用有 detail 视图，可能另一个博客应用也有同名的视图。Django 如何知道 {% url %} 标签到底对应哪一个应用的 URL 呢？

答案是：在根 URLconf 中添加命名空间。在 polls/urls.py 文件中稍作修改，加上 app_name 设置命名空间：
polls/urls.py
```
from django.urls import path

from . import views

app_name = "polls"
urlpatterns = [
    path("", views.index, name="index"),
    path("<int:question_id>/", views.detail, name="detail"),
    path("<int:question_id>/results/", views.results, name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```
现在，编辑 polls/index.html 文件，从：
polls/templates/polls/index.html
```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```
修改为指向具有命名空间的详细视图：
polls/templates/polls/index.html
```
¶
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```
当你对你写的视图感到满意后，请阅读 教程的第 4 部分 了解基础的表单处理和通用视图。
（五）- 1315

## 编写你的第一个 Django 应用，第 4 部分  
我们将继续网络投票的应用，并将重点放在表单处理和精简我们的代码上。

## 编写一个简单的表单  

让我们更新一下在上一个教程中编写的投票详细页面的模板 ("polls/detail.html") ，让它包含一个 HTML <form> 元素：
polls/templates/polls/detail.html¶
```
<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
<fieldset>
    <legend><h1>{{ question.question_text }}</h1></legend>
    {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
    {% for choice in question.choice_set.all %}
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
    {% endfor %}
</fieldset>
<input type="submit" value="Vote">
</form>
```
简要说明：  
- 上面的模板在 Question 的每个 Choice 前添加一个单选按钮。 每个单选按钮的 value 属性是对应的各个 Choice 的 ID。每个单选按钮的 name 是 "choice" 。这意味着，当有人选择一个单选按钮并提交表单提交时，它将发送一个 POST 数据 choice=# ，其中# 为选择的 Choice 的 ID。这是 HTML 表单的基本概念。
- 我们将表单的 action 设置为 {% url 'polls:vote' question.id %}，并设置 method="post"。使用 method="post" （而不是 method="get" ）是非常重要的，因为提交这个表单的行为将改变服务器端的数据。当你创建一个改变服务器端数据的表单时，使用 method="post"。这不是 Django 的特定技巧；这是优秀的网站开发技巧。
- forloop.counter 指示 for 标签已经循环多少次。
- 由于我们创建一个 POST 表单（它具有修改数据的作用），所以我们需要小心跨站点请求伪造。 谢天谢地，你不必太过担心，因为 Django 自带了一个非常有用的防御系统。 简而言之，所有针对内部 URL 的 POST 表单都应该使用 {% csrf_token %} 模板标签。
现在，让我们来创建一个 Django 视图来处理提交的数据。记住，在 教程第 3 部分 中，我们为投票应用创建了一个 URLconf ，包含这一行：
polls/urls.py
```
path("<int:question_id>/vote/", views.vote, name="vote"),
```
我们还创建了一个 vote() 函数的虚拟实现。让我们来创建一个真实的版本。 将下面的代码添加到 polls/views.py ：
polls/views.py
```
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question


# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST["choice"])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(
            request,
            "polls/detail.html",
            {
                "question": question,
                "error_message": "You didn't select a choice.",
            },
        )
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse("polls:results", args=(question.id,)))
```
以上代码中有些内容还未在本教程中提到过：

- request.POST 是一个类字典对象，让你可以通过关键字的名字获取提交的数据。 这个例子中， request.POST['choice'] 以字符串形式返回选择的 Choice 的 ID。 request.POST 的值永远是字符串。

- 注意，Django 还以同样的方式提供 request.GET 用于访问 GET 数据 —— 但我们在代码中显式地使用 request.POST ，以保证数据只能通过 POST 调用改动。

- 如果在 request.POST['choice'] 数据中没有提供 choice ， POST 将引发一个 KeyError 。上面的代码检查 KeyError ，如果没有给出 choice 将重新显示 Question 表单和一个错误信息。

- 在增加 Choice 的得票数之后，代码返回一个 HttpResponseRedirect 而不是常用的 HttpResponse 、 HttpResponseRedirect 只接收一个参数：用户将要被重定向的 URL（请继续看下去，我们将会解释如何构造这个例子中的 URL）。

- 正如上面的 Python 注释指出的，在成功处理 POST 数据后，你应该总是返回一个 HttpResponseRedirect。这不是 Django 的特殊要求，这是那些优秀网站在开发实践中形成的共识。

- 在这个例子中，我们在 HttpResponseRedirect 的构造函数中使用 reverse() 函数。这个函数避免了我们在视图函数中硬编码 URL。它需要我们给出我们想要跳转的视图的名字和该视图所对应的 URL 模式中需要给该视图提供的参数。 在本例中，使用在 教程第 3 部分 中设定的 URLconf， reverse() 调用将返回一个这样的字符串：

```
"/polls/3/results/"
```
其中 3 是 question.id 的值。重定向的 URL 将调用 'results' 视图来显示最终的页面。
正如在 教程第 3 部分 中提到的，HttpRequest 是一个 HttpRequest 对象。

当有人对 Question 进行投票后， vote() 视图将请求重定向到 Question 的结果界面。让我们来编写这个视图：
polls/views.py
```
from django.shortcuts import get_object_or_404, render


def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/results.html", {"question": question})
```
这和 教程第 3 部分 中的 detail() 视图几乎一模一样。唯一的不同是模板的名字。 我们将在稍后解决这个冗余问题。

现在，创建一个 polls/results.html 模板：
polls/templates/polls/results.html

```<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>

```
现在，在你的浏览器中访问 /polls/1/ 然后为 Question 投票。你应该看到一个投票结果页面，并且在你每次投票之后都会更新。 如果你提交时没有选择任何 Choice，你应该看到错误信息。

```
备注

我们的 vote() 视图代码有一个小问题。代码首先从数据库中获取了 selected_choice 对象，接着计算 vote 的新值，最后把值存回数据库。如果网站有两个方可同时投票在 同一时间 ，可能会导致问题。同样的值，42，会被 votes 返回。然后，对于两个用户，新值43计算完毕，并被保存，但是期望值是44。

这个问题被称为 竞争条件 。
```

##  使用通用视图：代码还是少点好  

detail() （在 教程第 3 部分 中）和 results() 视图都很精简 —— 并且，像上面提到的那样，存在冗余问题。用来显示一个投票列表的 index() 视图（也在 教程第 3 部分 中）和它们类似。

这些视图反映基本的网络开发中的一个常见情况：根据 URL 中的参数从数据库中获取数据、载入模板文件然后返回渲染后的模板。 由于这种情况特别常见，Django 提供一种快捷方式，叫做 “通用视图” 系统。

通用视图将常见的模式抽象化，可以使你在编写应用时甚至不需要编写Python代码。

让我们将我们的投票应用转换成使用通用视图系统，这样我们可以删除许多我们的代码。我们仅仅需要做以下几步来完成转换，我们将：
1. 转换 URLconf。
2. 删除一些旧的、不再需要的视图。
3. 基于 Django 的通用视图引入新的视图。
请继续阅读来了解详细信息。
```
为什么要重构代码？

一般来说，当编写一个 Django 应用时，你应该先评估一下通用视图是否可以解决你的问题，你应该在一开始使用它，而不是进行到一半时重构代码。本教程目前为止是有意将重点放在以“艰难的方式”编写视图，这是为将重点放在核心概念上。

就像在使用计算器之前你需要掌握基础数学一样。
```

##  改良 URLconf  
首先，打开 polls/urls.py 这个 URLconf 并将它修改成：
polls/urls.py
```
from django.urls import path

from . import views

app_name = "polls"
urlpatterns = [
    path("", views.IndexView.as_view(), name="index"),
    path("<int:pk>/", views.DetailView.as_view(), name="detail"),
    path("<int:pk>/results/", views.ResultsView.as_view(), name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]

```
注意，第二个和第三个匹配准则中，路径字符串中匹配模式的名称已经由 <question_id> 改为 <pk>。

## 改良视图  

下一步，我们将删除旧的 index, detail, 和 results 视图，并用 Django 的通用视图代替。打开 polls/views.py 文件，并将它修改成：  

polls/views.py
```
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by("-pub_date")[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = "polls/detail.html"


class ResultsView(generic.DetailView):
    model = Question
    template_name = "polls/results.html"


def vote(request, question_id):
    ...  # same as above, no changes needed.
```
我们在这里使用两个通用视图： ListView 和 DetailView 。这两个视图分别抽象“显示一个对象列表”和“显示一个特定类型对象的详细信息页面”这两种概念。

- 每个通用视图需要知道它将作用于哪个模型。 这由 model 属性提供。
- DetailView 期望从 URL 中捕获名为 "pk" 的主键值，所以我们为通用视图把 question_id 改成 pk 。

默认情况下，通用视图 DetailView 使用一个叫做 <app name>/<model name>_detail.html 的模板。在我们的例子中，它将使用 "polls/question_detail.html" 模板。template_name 属性是用来告诉 Django 使用一个指定的模板名字，而不是自动生成的默认名字。 我们也为 results 列表视图指定了 template_name —— 这确保 results 视图和 detail 视图在渲染时具有不同的外观，即使它们在后台都是同一个 DetailView 。

类似地，ListView 使用一个叫做 <app name>/<model name>_list.html 的默认模板；我们使用 template_name 来告诉 ListView 使用我们创建的已经存在的 "polls/index.html" 模板。

在之前的教程中，提供模板文件时都带有一个包含 question 和 latest_question_list 变量的 context。对于 DetailView ， question 变量会自动提供—— 因为我们使用 Django 的模型（Question）， Django 能够为 context 变量决定一个合适的名字。然而对于 ListView， 自动生成的 context 变量是 question_list。为了覆盖这个行为，我们提供 context_object_name 属性，表示我们想使用 latest_question_list。作为一种替换方案，你可以改变你的模板来匹配新的 context 变量 —— 这是一种更便捷的方法，告诉 Django 使用你想使用的变量名。

启动服务器，使用一下基于通用视图的新投票应用。

(六)- 1528  
## 编写你的第一个 Django 应用，第 5 部分  

我们已经建立了一个网络投票应用程序，现在我们将为它创建一些自动化测试。

## 自动化测试简介  

### 自动化测试是什么？
测试代码，是用来检查你的代码能否正常运行的程序。

测试在不同的层次中都存在。有些测试只关注某个很小的细节（某个模型的某个方法的返回值是否满足预期？），而另一些测试可能检查对某个软件的一系列操作（某一用户输入序列是否造成了预期的结果？）。其实这和我们在 教程第 2 部分，里做的并没有什么不同，我们使用 shell 来测试某一方法的功能，或者运行某个应用并输入数据来检查它的行为。

真正不同的地方在于，自动化 测试是由某个系统帮你自动完成的。当你创建好了一系列测试，每次修改应用代码后，就可以自动检查出修改后的代码是否还像你曾经预期的那样正常工作。你不需要花费大量时间来进行手动测试。

### 为什么你需要写测试  
但是，为什么需要测试呢？又为什么是现在呢？

你可能觉得学 Python/Django 对你来说已经很满足了，再学一些新东西的话看起来有点负担过重并且没什么必要。毕竟，我们的投票应用看起来已经完美工作了。写一些自动测试并不能让它工作的更好。如果写一个投票应用是你想用 Django 完成的唯一工作，那你确实没必要学写测试。但是如果你还想写更复杂的项目，现在就是学习测试写法的最好时机了。

#### 测试将节约你的时间  
在某种程度上，能够「判断出代码是否正常工作」的测试，就称得上是个令人满意的了。在更复杂的应用程序中，组件之间可能会有数十个复杂的交互。

对其中某一组件的改变，也有可能会造成意想不到的结果。判断「代码是否正常工作」意味着你需要用大量的数据来完整的测试全部代码的功能，以确保你的小修改没有对应用整体造成破坏——这太费时间了。

尤其是当你发现自动化测试能在几秒钟之内帮你完成这件事时，就更会觉得手动测试实在是太浪费时间了。当某人写出错误的代码时，自动化测试还能帮助你定位错误代码的位置。

有时候你会觉得，和富有创造性和生产力的业务代码比起来，编写枯燥的测试代码实在是太无聊了，特别是当你知道你的代码完全没有问题的时候。

然而，编写测试还是要比花费几个小时手动测试你的应用，或者为了找到某个小错误而胡乱翻看代码要有意义的多。  

### 测试不仅能发现错误，而且能预防错误  
「测试是开发的对立面」，这种思想是不对的。

如果没有测试，整个应用的行为意图会变得更加的不清晰。甚至当你在看自己写的代码时也是这样，有时候你需要仔细研读一段代码才能搞清楚它有什么用。

而测试的出现改变了这种情况。测试就好像是从内部仔细检查你的代码，当有些地方出错时，这些地方将会变得很显眼——就算你自己没有意识到那里写错了

### 测试使你的代码更有吸引力  
你也许遇到过这种情况：你编写了一个绝赞的软件，但是其他开发者看都不看它一眼，因为它缺少测试。没有测试的代码不值得信任。 Django 最初开发者之一的 Jacob Kaplan-Moss 说过：“项目规划时没有包含测试是不科学的。”

其他的开发者希望在正式使用你的代码前看到它通过了测试，这是你需要写测试的另一个重要原因。

### 测试有利于团队协作
前面的几点都是从单人开发的角度来说的。复杂的应用可能由团队维护。测试的存在保证了协作者不会不小心破坏了了你的代码（也保证你不会不小心弄坏他们的）。如果你想作为一个 Django 程序员谋生的话，你必须擅长编写测试！

## 基础测试策略  
有好几种不同的方法可以写测试。

一些开发者遵循 "测试驱动" 的开发原则，他们在写代码之前先写测试。这种方法看起来有点反直觉，但事实上，这和大多数人日常的做法是相吻合的。我们会先描述一个问题，然后写代码来解决它。「测试驱动」的开发方法只是将问题的描述抽象为了 Python 的测试样例。

更普遍的情况是，一个刚接触自动化测试的新手更倾向于先写代码，然后再写测试。虽然提前写测试可能更好，但是晚点写起码也比没有强。

有时候很难决定从哪里开始下手写测试。如果你才写了几千行 Python 代码，选择从哪里开始写测试确实不怎么简单。如果是这种情况，那么在你下次修改代码（比如加新功能，或者修复 Bug）之前写个测试是比较合理且有效的。

所以，我们现在就开始写吧。

## 开始写我们的第一个测试  

### 首先得有个 Bug  
幸运的是，我们的 polls 应用现在就有一个小 bug 需要被修复：我们的要求是如果 Question 是在一天之内发布的， Question.was_published_recently() 方法将会返回 True ，然而现在这个方法在 Question 的 pub_date 字段比当前时间还晚时也会返回 True（这是个 Bug）。

用djadmin:`shell`命令确认一下这个方法的日期bug

```
python manage.py shell
```
```
>>> import datetime
>>> from django.utils import timezone
>>> from polls.models import Question
>>> # create a Question instance with pub_date 30 days in the future
>>> future_question = Question(pub_date=timezone.now() + datetime.timedelta(days=30))
>>> # was it published recently?
>>> future_question.was_published_recently()
True
```
因为将来发生的是肯定不是最近发生的，所以代码明显是错误的。
## 创建一个测试来暴露这个 bug  
我们刚刚在 shell 里做的测试也就是自动化测试应该做的工作。所以我们来把它改写成自动化的吧。

按照惯例，Django 应用的测试应该写在应用的 tests.py 文件里。测试系统会自动的在所有以 tests 开头的文件里寻找并执行测试代码。

将下面的代码写入 polls 应用里的 tests.py 文件内：
polls/tests.py
```
import datetime

from django.test import TestCase
from django.utils import timezone

from .models import Question


class QuestionModelTests(TestCase):
    def test_was_published_recently_with_future_question(self):
        """
        was_published_recently() returns False for questions whose pub_date
        is in the future.
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertIs(future_question.was_published_recently(), False)
```
我们创建了一个 django.test.TestCase 的子类，并添加了一个方法，此方法创建一个 pub_date 时未来某天的 Question 实例。然后检查它的 was_published_recently() 方法的返回值——它 应该 是 False。

### 运行测试  
在终端中，我们通过输入以下代码运行测试:

```
python manage.py test polls
```
and you'll see something like:
```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_was_published_recently_with_future_question (polls.tests.QuestionModelTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/path/to/mysite/polls/tests.py", line 16, in test_was_published_recently_with_future_question
    self.assertIs(future_question.was_published_recently(), False)
AssertionError: True is not False

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
Destroying test database for alias 'default'...
```
```
不一样的错误？

若在此处你得到了一个 NameError 错误，你可能漏了 第二步 中将 datetime 和 timezone 导入 polls/model.py 的步骤。复制这些语句，然后试着重新运行测试。
```
发生了什么呢？以下是自动化测试的运行过程：
- python manage.py test polls 将会寻找 polls 应用里的测试代码
- 它找到了 django.test.TestCase 的一个子类
- 它创建一个特殊的数据库供测试使用
- 它在类中寻找测试方法——以 test 开头的方法。
- 在 test_was_published_recently_with_future_question 方法中，它创建了一个 pub_date 值为 30 天后的 Question 实例。
- 接着使用 assertls() 方法，发现 was_published_recently() 返回了 True，而我们期望它返回 False。

测试系统通知我们哪些测试样例失败了，和造成测试失败的代码所在的行号。

## 修复这个 bug  
我们早已知道，当 pub_date 为未来某天时， Question.was_published_recently() 应该返回 False。我们修改 models.py 里的方法，让它只在日期是过去式的时候才返回 True：
polls/models.py
```
def was_published_recently(self):
    now = timezone.now()
    return now - datetime.timedelta(days=1) <= self.pub_date <= now
```
and run the test again:
```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
```
发现 bug 后，我们编写了能够暴露这个 bug 的自动化测试。在修复 bug 之后，我们的代码顺利的通过了测试。

将来，我们的应用可能会出现其他的问题，但是我们可以肯定的是，一定不会再次出现这个 bug，因为只要运行一遍测试，就会立刻收到警告。我们可以认为应用的这一小部分代码永远是安全的。

### 更全面的测试  

我们已经搞定一小部分了，现在可以考虑全面的测试 was_published_recently() 这个方法以确定它的安全性，然后就可以把这个方法稳定下来了。事实上，在修复一个 bug 时不小心引入另一个 bug 会是非常令人尴尬的。

我们在上次写的类里再增加两个测试，来更全面的测试这个方法：
polls/tests.py
```
def test_was_published_recently_with_old_question(self):
    """
    was_published_recently() returns False for questions whose pub_date
    is older than 1 day.
    """
    time = timezone.now() - datetime.timedelta(days=1, seconds=1)
    old_question = Question(pub_date=time)
    self.assertIs(old_question.was_published_recently(), False)


def test_was_published_recently_with_recent_question(self):
    """
    was_published_recently() returns True for questions whose pub_date
    is within the last day.
    """
    time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
    recent_question = Question(pub_date=time)
    self.assertIs(recent_question.was_published_recently(), True)
```
现在，我们有三个测试来确保 Question.was_published_recently() 方法对于过去，最近，和将来的三种情况都返回正确的值。

再次申明，尽管 polls 现在是个小型的应用，但是无论它以后变得到多么复杂，无论他和其他代码如何交互，我们可以在一定程度上保证我们为之编写测试的方法将按照预期的方式运行。

## 测试视图  
我们的投票应用对所有问题都一视同仁：它将会发布所有的问题，也包括那些 pub_date 字段值是未来的问题。我们应该改善这一点。如果 pub_date 设置为未来某天，这应该被解释为这个问题将在所填写的时间点才被发布，而在之前是不可见的。

### 针对视图的测试  
为了修复上述 bug ，我们这次先编写测试，然后再去改代码。事实上，这是一个「测试驱动」开发模式的实例，但其实这两者的顺序不太重要。

在我们的第一个测试中，我们关注代码的内部行为。我们通过模拟用户使用浏览器访问被测试的应用来检查代码行为是否符合预期。

在我们动手之前，先看看需要用到的工具们。

### Django 测试工具之 Client  
Django 提供了一个供测试使用的 Client 来模拟用户和视图层代码的交互。我们能在 tests.py 甚至是 shell 中使用它。

我们依照惯例从 shell 开始，首先我们要做一些在 tests.py 里不是必须的准备工作。第一步是在 shell 中配置测试环境:

```
python manage.py shell
```
```
>>> from django.test.utils import setup_test_environment
>>> setup_test_environment()
```
setup_test_environment() 安装了一个模板渲染器，这将使我们能够检查响应上的一些额外属性，如 response.context，否则将无法使用此功能。请注意，这个方法 不会 建立一个测试数据库，所以下面的内容将针对现有的数据库运行，输出结果可能略有不同，这取决于你已经创建了哪些问题。如果你在 settings.py 中的 TIME_ZONE 不正确，你可能会得到意外的结果。如果你不记得之前的配置，请在继续之前检查

接下來我們需要導入測試客戶端類（稍後在tests.py中我們將使用django.test.TestCase類，它帶有自己的客戶端，所以這不是必需的）：

```
>>> from django.test import Client
>>> # create an instance of the client for our use
>>> client = Client()
```
準備好後，我們可以要求客戶為我們做一些工作：
```
>>> # get a response from '/'
>>> response = client.get("/")
Not Found: /
>>> # we should expect a 404 from that address; if you instead see an
>>> # "Invalid HTTP_HOST header" error and a 400 response, you probably
>>> # omitted the setup_test_environment() call described earlier.
>>> response.status_code
404
>>> # on the other hand we should expect to find something at '/polls/'
>>> # we'll use 'reverse()' rather than a hardcoded URL
>>> from django.urls import reverse
>>> response = client.get(reverse("polls:index"))
>>> response.status_code
200
>>> response.content
b'\n    <ul>\n    \n        <li><a href="/polls/1/">What&#x27;s up?</a></li>\n    \n    </ul>\n\n'
>>> response.context["latest_question_list"]
<QuerySet [<Question: What's up?>]>
```

## 改善视图代码  

现在的投票列表会显示将来的投票（ pub_date 值是未来的某天)。我们来修复这个问题。

在 教程的第 4 部分 里，我们介绍了基于 ListView 的视图类：
polls/views.py
```
class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by("-pub_date")[:5]
```
我们需要改进 get_queryset() 方法，让他它能通过将 Question 的 pub_data 属性与 timezone.now() 相比较来判断是否应该显示此 Question。首先我们需要一行 import 语句：
polls/views.py
```
from django.utils import timezone
```
然后我们把 get_queryset 方法改写成下面这样：
polls/views.py
```
def get_queryset(self):
    """
    Return the last five published questions (not including those set to be
    published in the future).
    """
    return Question.objects.filter(pub_date__lte=timezone.now()).order_by("-pub_date")[
        :5
    ]
```
Question.objects.filter(pub_date__lte=timezone.now()) 返回一個查詢集，其中包含 pub_date 小於或等於（即早於或等於）timezone.now 的問題。

## 测试新视图  
启动服务器、在浏览器中载入站点、创建一些发布时间在过去和将来的 Questions ，然后检验只有已经发布的 Questions 会展示出来，现在你可以对自己感到满意了。你不想每次修改可能与这相关的代码时都重复这样做 —— 所以让我们基于以上 shell 会话中的内容，再编写一个测试

将下面的代码添加到 polls/tests.py ：
polls/tests.py
```
from django.urls import reverse
```
然后我们写一个公用的快捷函数用于创建投票问题，再为视图创建一个测试类：
polls/tests.py
```
def create_question(question_text, days):
    """
    Create a question with the given `question_text` and published the
    given number of `days` offset to now (negative for questions published
    in the past, positive for questions that have yet to be published).
    """
    time = timezone.now() + datetime.timedelta(days=days)
    return Question.objects.create(question_text=question_text, pub_date=time)


class QuestionIndexViewTests(TestCase):
    def test_no_questions(self):
        """
        If no questions exist, an appropriate message is displayed.
        """
        response = self.client.get(reverse("polls:index"))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "No polls are available.")
        self.assertQuerySetEqual(response.context["latest_question_list"], [])

    def test_past_question(self):
        """
        Questions with a pub_date in the past are displayed on the
        index page.
        """
        question = create_question(question_text="Past question.", days=-30)
        response = self.client.get(reverse("polls:index"))
        self.assertQuerySetEqual(
            response.context["latest_question_list"],
            [question],
        )

    def test_future_question(self):
        """
        Questions with a pub_date in the future aren't displayed on
        the index page.
        """
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse("polls:index"))
        self.assertContains(response, "No polls are available.")
        self.assertQuerySetEqual(response.context["latest_question_list"], [])

    def test_future_question_and_past_question(self):
        """
        Even if both past and future questions exist, only past questions
        are displayed.
        """
        question = create_question(question_text="Past question.", days=-30)
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse("polls:index"))
        self.assertQuerySetEqual(
            response.context["latest_question_list"],
            [question],
        )

    def test_two_past_questions(self):
        """
        The questions index page may display multiple questions.
        """
        question1 = create_question(question_text="Past question 1.", days=-30)
        question2 = create_question(question_text="Past question 2.", days=-5)
        response = self.client.get(reverse("polls:index"))
        self.assertQuerySetEqual(
            response.context["latest_question_list"],
            [question2, question1],
        )
```
让我们更详细地看下以上这些内容。

首先是一个快捷函数 create_question，它封装了创建投票的流程，减少了重复代码。

test_no_questions doesn't create any questions, but checks the message: "No polls are available." and verifies the latest_question_list is empty. Note that the django.test.TestCase class provides some additional assertion methods. In these examples, we use assertContains() and assertQuerySetEqual().

在 test_past_question 方法中，我们创建了一个投票并检查它是否出现在列表中。

在 test_future_question 中，我们创建 pub_date 在未来某天的投票。数据库会在每次调用测试方法前被重置，所以第一个投票已经没了，所以主页中应该没有任何投票。

剩下的那些也都差不多。实际上，测试就是假装一些管理员的输入，然后通过用户端的表现是否符合预期来判断新加入的改变是否破坏了原有的系统状态。  

## 测试 DetailView  
我们的工作似乎已经很完美了？不，还有一个问题：就算在发布日期时未来的那些投票不会在目录页 index 里出现，但是如果用户知道或者猜到正确的 URL ，还是可以访问到它们。所以我们得在 DetailView 里增加一些约束：
polls/views.py
```
class DetailView(generic.DetailView):
    ...

    def get_queryset(self):
        """
        Excludes any questions that aren't published yet.
        """
        return Question.objects.filter(pub_date__lte=timezone.now())
```
然后，我们应该增加一些测试来检验 pub_date 在过去的 Question 能够被显示出来，而 pub_date 在未来的则不可以：
polls/tests.py
```
class QuestionDetailViewTests(TestCase):
    def test_future_question(self):
        """
        The detail view of a question with a pub_date in the future
        returns a 404 not found.
        """
        future_question = create_question(question_text="Future question.", days=5)
        url = reverse("polls:detail", args=(future_question.id,))
        response = self.client.get(url)
        self.assertEqual(response.status_code, 404)

    def test_past_question(self):
        """
        The detail view of a question with a pub_date in the past
        displays the question's text.
        """
        past_question = create_question(question_text="Past Question.", days=-5)
        url = reverse("polls:detail", args=(past_question.id,))
        response = self.client.get(url)
        self.assertContains(response, past_question.question_text)
```
##  更多的测试思路  
我们应该给 ResultsView 也增加一个类似的 get_queryset 方法，并且为它创建测试。这和我们之前干的差不多，事实上，基本就是重复一遍。

我们还可以从各个方面改进投票应用，但是测试会一直伴随我们。比方说，在目录页上显示一个没有选项 Choices 的投票问题就没什么意义。我们可以检查并排除这样的投票题。测试可以创建一个没有选项的投票，然后检查它是否被显示在目录上。当然也要创建一个有选项的投票，然后确认它确实被显示了。

恩，也许你想让管理员能在目录上看见未被发布的那些投票，但是普通用户看不到。不管怎么说，如果你想要增加一个新功能，那么同时一定要为它编写测试。不过你是先写代码还是先写测试那就随你了。

在未来的某个时刻，你一定会去查看测试代码，然后开始怀疑：「这么多的测试不会使代码越来越复杂吗？」。别着急，我们马上就会谈到这一点。

## 当需要测试的时候，测试用例越多越好  

貌似我们的测试多的快要失去控制了。按照这样发展下去，测试代码就要变得比应用的实际代码还要多了。而且测试代码大多都是重复且不优雅的，特别是在和业务代码比起来的时候，这种感觉更加明显。

但是这没关系！ 就让测试代码继续肆意增长吧。大部分情况下，你写完一个测试之后就可以忘掉它了。在你继续开发的过程中，它会一直默默无闻地为你做贡献的。

但有时测试也需要更新。想象一下如果我们修改了视图，只显示有选项的那些投票，那么只前写的很多测试就都会失败。但这也明确地告诉了我们哪些测试需要被更新，所以测试也会测试自己。

最坏的情况是，当你继续开发的时候，发现之前的一些测试现在看来是多余的。但是这也不是什么问题，多做些测试也 不错

如果你对测试有个整体规划，那么它们就几乎不会变得混乱。下面有几条好的建议：

- 对于每个模型和视图都建立单独的 TestClass
- 每个测试方法只测试一个功能
- 给每个测试方法起个能描述其功能的名字

##  深入代码测试  
在本教程中，我们仅仅是了解了测试的基础知识。你能做的还有很多，而且世界上有很多有用的工具来帮你完成这些有意义的事。

举个例子，在上述的测试中，我们已经从代码逻辑和视图响应的角度检查了应用的输出，现在你可以从一个更加 "in-browser" 的角度来检查最终渲染出的 HTML 是否符合预期，使用 Selenium 可以很轻松的完成这件事。这个工具不仅可以测试 Django 框架里的代码，还可以检查其他部分，比如说你的 JavaScript。它假装成是一个正在和你站点进行交互的浏览器，就好像有个真人在访问网站一样！Django 它提供了 LiveServerTestCase 来和 Selenium 这样的工具进行交互。

如果你在开发一个很复杂的应用的话，你也许想在每次提交代码时自动运行测试，也就是我们所说的持续集成 continuous integration ，这样就能实现质量控制的自动化，起码是部分自动化。

一个找出代码中未被测试部分的方法是检查代码覆盖率。它有助于找出代码中的薄弱部分和无用部分。如果你无法测试一段代码，通常说明这段代码需要被重构或者删除。

（七）--- 1975  
## 编写你的第一个 Django 应用，第 6 部分  

本教程从 教程第 5 部分 结束的地方开始。我们已经建立了一个经过测试的网络投票应用程序，现在我们将添加一个样式表和一个图像。

除了服务端生成的 HTML 以外，网络应用通常需要一些额外的文件——比如图片，脚本和样式表——来帮助渲染网络页面。在 Django 中，我们把这些文件统称为“静态文件”。

对于小项目来说，这个问题没什么大不了的，因为你可以把这些静态文件随便放在哪，只要服务程序能够找到它们就行。然而在大项目——特别是由好几个应用组成的大项目——中，处理不同应用所需要的静态文件的工作就显得有点麻烦了。

这就是 django.contrib.staticfiles 存在的意义：它将各个应用的静态文件（和一些你指明的目录里的文件）统一收集起来，这样一来，在生产环境中，这些文件就会集中在一个便于分发的地方。

## 自定义 应用 的界面和风格  
首先，在你的 polls 目录下创建一个名为 static 的目录。Django 将在该目录下查找静态文件，这种方式和 Diango 在 polls/templates/ 目录下查找 template 的方式类似。

Django 的 STATICFILES_FINDERS 设置包含了一系列的查找器，它们知道去哪里找到 static 文件。AppDirectoriesFinder 是默认查找器中的一个，它会在每个 INSTALLED_APPS 中指定的应用的子文件中寻找名称为 static 的特定文件夹，就像我们在 polls 中刚创建的那个一样。管理后台采用相同的目录结构管理它的静态文件。

在你刚创建的 static 文件夹中创建一个名为 polls 的文件夹，再在 polls 文件夹中创建一个名为 style.css 的文件。换句话说，你的样式表路径应是 polls/static/polls/style.css。因为 AppDirectoriesFinder 的存在，你可以在 Django 中以 polls/style.css 的形式引用此文件，类似你引用模板路径的方式。

```
静态文件命名空间

虽然我们 可以 像管理模板文件一样，把 static 文件直接放入 polls/static （而不是创建另一个名为 polls 的子文件夹），不过这实际上是一个很蠢的做法。Django 只会使用第一个找到的静态文件。如果你在 其它 应用中有一个相同名字的静态文件，Django 将无法区分它们。我们需要指引 Django 选择正确的静态文件，而最好的方式就是把它们放入各自的 命名空间 。也就是把这些静态文件放入 另一个 与应用名相同的目录中。
```
将以下代码放入样式表(polls/static/polls/style.css)：
polls/static/polls/style.css¶
```
li a {
    color: green;
}
```
下一步，在 polls/templates/polls/index.html 的文件头添加以下内容：
polls/templates/polls/index.html
```
{% load static %}

<link rel="stylesheet" href="{% static 'polls/style.css' %}">
```
{% static %} 模板标签会生成静态文件的绝对路径。

这就是你开发所需要做的所有事情了。

启动服务器(如果它正在运行中，重新启动一次):

```
python manage.py runserver
```

##  添加一个背景图  

接下来，我们将为图像创建一个子目录。 在 polls/static/polls/ 目录中创建 images 子目录。 在此目录中，添加您想用作背景的任何图像文件。 出于本教程的目的，我们使用了一个名为“background.png”的文件，它的完整路径为“polls/static/polls/images/background.png”。

然后，在样式表中添加对图像的引用（polls/static/polls/style.css）：
polls/static/polls/style.css
```
body {
    background: white url("images/background.png") no-repeat;
}
```
浏览器重载 http://localhost:8000/polls/，你将在屏幕的左上角见到这张背景图。
```
警告

{% static %} 模板标签在静态文件（例如样式表）中是不可用的，因为它们不是由 Django 生成的。你应该始终使用 相对路径 在你的静态文件之间相互引用，因为这样你可以更改 STATIC_URL （由 static 模板标签使用来生成 URL），而无需修改大量的静态文件。
```
##  编写你的第一个 Django 应用，第 7 部分  

### 自定义后台表单  

通过 admin.site.register(Question) 注册 Question 模型，Django 能够构建一个默认的表单用于展示。通常来说，你期望能自定义表单的外观和工作方式。你可以在注册模型时将这些设置告诉 Django。

让我们通过重排列表单上的字段来看看它是怎么工作的。用以下内容替换 admin.site.register(Question)：
polls/admin.py
```
from django.contrib import admin

from .models import Question


class QuestionAdmin(admin.ModelAdmin):
    fields = ["pub_date", "question_text"]


admin.site.register(Question, QuestionAdmin)
```
你需要遵循以下流程——创建一个模型后台类，接着将其作为第二个参数传给 admin.site.register() ——在你需要修改模型的后台管理选项时这么做。

以上修改使得 "Publication date" 字段显示在 "Question" 字段之前：
<img src= "https://docs.djangoproject.com/en/4.2/_images/admin07.png">  
这在只有两个字段时显得没啥卵用，但对于拥有数十个字段的表单来说，为表单选择一个直观的排序方法就显得你的针很细了。

说到拥有数十个字段的表单，你可能更期望将表单分为几个字段集：
polls/admin.py
```
from django.contrib import admin

from .models import Question


class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None, {"fields": ["question_text"]}),
        ("Date information", {"fields": ["pub_date"]}),
    ]


admin.site.register(Question, QuestionAdmin)
```
fieldsets 元组中的第一个元素是字段集的标题。以下是我们的表单现在的样子：
<img src="https://docs.djangoproject.com/en/4.2/_images/admin08t.png">

## 添加关联的对象  
好了，现在我们有了投票的后台页。不过，一个 Question 有多个 Choice，但后台页却没有显示多个选项。

好了。

有两个方法可以解决这个问题。第一个就是仿照我们向后台注册 Question 一样注册 Choice ：
polls/admin.py
```
from django.contrib import admin

from .models import Choice, Question

# ...
admin.site.register(Choice)
```
现在 "Choices" 在 Django 后台页中是一个可用的选项了。“添加选项”的表单看起来像这样：
<img src="https://docs.djangoproject.com/en/4.2/_images/admin09.png">  
在这个表单中，"Question" 字段是一个包含数据库中所有投票的选择框。Django 知道要将 ForeignKey 在后台中以选择框 <select> 的形式展示。此时，我们只有一个投票。

还请注意“问题”旁边的“添加另一个问题”链接。每个与另一个具有`ForeignKey``关系的对象都可以免费获得此链接。当你点击“添加另一个问题”时，你会看到一个带有“添加问题”表单的弹出窗口。如果你在该窗口中添加问题并点击“保存”，Django会将问题保存到数据库中，并将其动态添加为你正在查看的“添加选项”表单上的选定选项。

不过，这是一种很低效地添加“选项”的方法。更好的办法是在你创建“投票”对象时直接添加好几个选项。让我们实现它。

移除调用 register() 注册 Choice 模型的代码。随后，像这样修改 Question 的注册代码：
polls/admin.py
```
from django.contrib import admin

from .models import Choice, Question


class ChoiceInline(admin.StackedInline):
    model = Choice
    extra = 3


class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None, {"fields": ["question_text"]}),
        ("Date information", {"fields": ["pub_date"], "classes": ["collapse"]}),
    ]
    inlines = [ChoiceInline]


admin.site.register(Question, QuestionAdmin)
```
这会告诉 Django：“Choice 对象要在 Question 后台页面编辑。默认提供 3 个足够的选项字段。”

加载“添加投票”页面来看看它长啥样：
<img src="https://docs.djangoproject.com/en/4.2/_images/admin10t.png">
它看起来像这样：有三个关联的选项插槽——由 extra 定义，且每次你返回任意已创建的对象的“修改”页面时，你会见到三个新的插槽。

在三个插槽的末端，你会看到一个“添加新选项”的按钮。如果你单击它，一个新的插槽会被添加。如果你想移除已有的插槽，可以点击插槽右上角的X。以下图片展示了一个已添加的插槽：
<img src="https://docs.djangoproject.com/en/4.2/_images/admin14t.png">
不过，仍然有点小问题。它占据了大量的屏幕区域来显示所有关联的 Choice 对象的字段。对于这个问题，Django 提供了一种表格式的单行显示关联对象的方法。要使用它，只需按如下形式修改 ChoiceInline 申明：
polls/admin.py
```
class ChoiceInline(admin.TabularInline):
    ...
```
通过 TabularInline （替代 StackedInline ），关联对象以一种表格式的方式展示，显得更加紧凑：
<img src="https://docs.djangoproject.com/en/4.2/_images/admin11t.png">
请注意，有一个额外的“删除？”列，允许删除使用“添加另一个选项”按钮添加的行和已保存的行。

## 自定义后台更改列表  

现在投票的后台页看起来很不错，让我们对“更改列表”页面进行一些调整——改成一个能展示系统中所有投票的页面。

以下是它此时的外观：

<img src="https://docs.djangoproject.com/en/4.2/_images/admin04t.png">
默认情况下，Django 显示每个对象的 str() 返回的值。但有时如果我们能够显示单个字段，它会更有帮助。为此，使用 list_display 后台选项，它是一个包含要显示的字段名的元组，在更改列表页中以列的形式展示这个对象：
polls/admin.py
```
class QuestionAdmin(admin.ModelAdmin):
    # ...
    list_display = ["question_text", "pub_date"]
```
另外，让我们把 教程第 2 部分 中的 was_published_recently() 方法也加上：
polls/admin.py
```
class QuestionAdmin(admin.ModelAdmin):
    # ...
    list_display = ["question_text", "pub_date", "was_published_recently"]
```
现在修改投票的列表页看起来像这样：
<img src="https://docs.djangoproject.com/en/4.2/_images/admin12t.png">
你可以点击列标题来对这些行进行排序——除了 was_published_recently 这个列，因为没有实现排序方法。顺便看下这个列的标题 was_published_recently，默认就是方法名（用空格替换下划线），该列的每行都以字符串形式展示出处。

你可以通过在该方法上（在 polls/models.py 中）使用 display() 装饰器来改进，如下图所示：
polls/models.py
```
from django.contrib import admin


class Question(models.Model):
    # ...
    @admin.display(
        boolean=True,
        ordering="pub_date",
        description="Published recently?",
    )
    def was_published_recently(self):
        now = timezone.now()
        return now - datetime.timedelta(days=1) <= self.pub_date <= now
```
更多关于可通过装饰器设置的属性的信息，请参见 list_display。

再次编辑文件 polls/admin.py，优化 Question 变更页：过滤器，使用 list_filter。将以下代码添加至 QuestionAdmin：

```
list_filter = ["pub_date"]
```
这样做添加了一个“过滤器”侧边栏，允许人们以 pub_date 字段来过滤列表：
在列表的顶部增加一个搜索框。当输入待搜项时，Django 将搜索 question_text 字段。你可以使用任意多的字段——由于后台使用 LIKE 来查询数据，将待搜索的字段数限制为一个不会出问题大小，会便于数据库进行查询操作。

现在是给你的修改列表页增加分页功能的好时机。默认每页显示 100 项。变更页分页, 搜索框, 过滤器, 日期层次结构, 和 列标题排序 均以你期望的方式合作运行。


##  自定义后台界面和风格  
在每个后台页顶部显示“Django 管理员”显得很滑稽。这只是一串占位文本。

不过，你可以通过 Django 的模板系统来修改。Django 的后台由自己驱动，且它的交互接口采用 Django 自己的模板系统。

##  自定义你的 工程的 模板  
在你的工程目录（指包含 manage.py 的那个文件夹）内创建一个名为 templates 的目录。模板可放在你系统中任何 Django 能找到的位置。（谁启动了 Django，Django 就以他的用户身份运行。）不过，把你的模板放在工程内会带来很大便利，推荐你这样做。

打开你的设置文件（mysite/settings.py，牢记），在 TEMPLATES 设置中添加 DIRS 选项：
mysite/settings.py
```
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [BASE_DIR / "templates"],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
    },
]
```
DIRS 是一个包含多个系统目录的文件列表，用于在载入 Django 模板时使用，是一个待搜索路径。

```
组织模板

就像静态文件一样，我们 可以 把所有的模板文件放在一个大模板目录内，这样它也能工作的很好。但是，属于特定应用的模板文件最好放在应用所属的模板目录（例如 polls/templates），而不是工程的模板目录（templates）。我们会在 创建可复用的应用教程 中讨论 为什么 我们要这样做。
```
現在在範本內建立一個名為 admin 的目錄，並將範本 admin/base_site.html 從 Django 本身原始碼中的預設 Django 管理範本目錄 (django/contrib/admin/templates) 複製到該目錄中。

接着，用你网页站点的名字编辑替换文件内的 {{ site_header|default:_('Django administration') }} （包含大括号）。完成后，你应该看到如下代码：

```
{% block branding %}
<h1 id="site-name"><a href="{% url 'admin:index' %}">Polls Administration</a></h1>
{% endblock %}
```

我们会用这个方法来教你复写模板。在一个实际工程中，你可能更期望使用 django.contrib.admin.AdminSite.site_header 来进行简单的定制。

这个模板文件包含很多类似 {% block branding %} 和 {{ title }} 的文本。 {% 和 {{ 标签是 Django 模板语言的一部分。当 Django 渲染 admin/base_site.html 时，这个模板语言会被求值，生成最终的网页，就像我们在 教程第 3 部分 所学的一样。

注意，所有的 Django 默认后台模板均可被复写。若要复写模板，像你修改 base_site.html 一样修改其它文件——先将其从默认目录中拷贝到你的自定义目录，再做修改。

##  自定义你 应用的 模板  
机智的同学可能会问： DIRS 默认是空的，Django 是怎么找到默认的后台模板的？因为 APP_DIRS 被置为 True，Django 会自动在每个应用包内递归查找 templates/ 子目录（不要忘了 django.contrib.admin 也是一个应用）。

我们的投票应用不是非常复杂，所以无需自定义后台模板。不过，如果它变的更加复杂，需要修改 Django 的标准后台模板功能时，修改 应用 的模板会比 工程 的更加明智。这样，在其它工程包含这个投票应用时，可以确保它总是能找到需要的自定义模板文件。

## 自定义后台主页  
在类似的说明中，你可能想要自定义 Django 后台索引页的外观。

默认情况下，它展示了所有配置在 INSTALLED_APPS 中，已通过后台应用注册，按拼音排序的应用。你可能想对这个页面的布局做重大的修改。毕竟，索引页是后台的重要页面，它应该便于使用。

需要自定义的模板是 admin/index.html。（像上一节修改 admin/base_site.html 那样修改此文件——从默认目录中拷贝此文件至自定义模板目录）。打开此文件，你将看到它使用了一个叫做 app_list 的模板变量。这个变量包含了每个安装的 Django 应用。你可以用任何你期望的硬编码链接（链接至特定对象的管理页）替代使用这个变量。

(八) - 2267

##  寫你的第一個 Django 應用程序，第 8 部分  

本教程從教程 7 結束的地方開始。 我們已經建立了網路投票應用程序，現在將查看第三方軟體包。 Django 的優勢之一是豐富的第三方包生態系統。 它們是社群開發的軟體包，可用於快速改進應用程式的功能集。

本教學將展示如何新增常用的第三方套件 Django Debug Toolbar。 近年來，Django 調試工具列在 Django 開發者調查中名列最常用第三方包的前三名。

##  安裝 Django 調試工具列
Django 偵錯工具列是調試 Django Web 應用程式的有用工具。 它是由 Jazzband 組織維護的第三方軟體包。 工具列可協助您了解應用程式的功能並識別問題。 它透過提供面板來實現這一點，這些面板提供有關當前請求和回應的偵錯資訊。

要安裝工具列等第三方應用程序，您需要在啟動的虛擬環境中執行以下命令來安裝軟體包。 這與我們之前安裝 Django 的步驟類似。

```
$ python -m pip 安裝 django-debug-toolbar
```

與 Django 整合的第三方包需要一些安裝後設定才能將它們與您的專案整合。 通常，您需要將套件的 Django 應用程式新增至 INSTALLED_APPS 設定中。 有些套件需要其他更改，例如添加到 URLconf (urls.py)。

Django 調試工具列需要幾個設定步驟。 請按照安裝指南中的說明進行操作。 本教程中不再重複這些步驟，因為作為第三方包，它可能會根據 Django 的時間表單獨更改。

安裝後，當您刷新民意調查應用程式時，您應該能夠在瀏覽器視窗的右側看到 DjDT“句柄”。 按一下它可開啟偵錯工具列並使用每個面板中的工具。 有關面板顯示內容的更多信息，請參閱面板文檔頁面。

##  獲得他人的幫助

有時您會遇到問題，例如工具列可能無法呈現。 當發生這種情況並且您無法自行解決問題時，您可以選擇其他方法。

如果問題出在特定軟體包上，請檢查該軟體包的文件中是否有常見問題的故障排除。 例如，Django 偵錯工具列有一個提示部分，概述了故障排除選項。

可能很難知道應該使用哪些第三方軟體包。 這取決於您的需求和目標。 有時使用處於 alpha 狀態的套件是很好的。 其他時候，您需要知道它已準備好投入生產。 Adam Johnson 發表了一篇博文，概述了使軟體包「維護良好」的一系列特徵。 Django Packages 顯示其中一些特徵的數據，例如上次更新包的時間。

正如亞當在他的帖子中指出的那樣，當其中一個問題的答案是「否」時，那就是一個做出貢獻的機會。

（九）--- 2302

##  进阶指南：如何编写可重用程序  

这个进阶教程从 教程第 8 部分 结束的地方继续讲起。我们将会把我们的网络投票应用放进一个独立的 Python 包中，以便你在新的项目中重用它或将它与他人分享。

如果你尚未完成教程 1-7，我们推荐你先浏览一遍教程，这样你的样例工程会和下面的一致。

### 可重用性很重要

设计，构建，测试以及维护一个 web 应用要做很多的工作。很多 Python 以及 Django 项目都有一些常见问题。如果我们能保存并利用这些重复的工作岂不是更好？

可重用性是 Python 的根本。The Python Package Index (PyPI) 有许大量的包，都可被用在你自己的 Python 项目中。同样可以在 Django Packages 中查找已发布的可重用应用，也可将其引入到你的项目中。Django 本身也是一个 Python 包，也就是说你可以将已有的 Python 包或 Django 应用并入你的项目。你只需要编写属于你的那部分即可。

假设你现在创建了一个新的项目，并且需要一个类似我们之前做的投票应用。你该如何复用这个应用呢？庆幸的是，其实你已经知道了一些。在 教程 1，我们使用过 include 从项目级别的 URLconf 分割出 polls。在本教程中，我们将进一步使这个应用易用于新的项目中，并发布给其他人安装使用。

```
包？应用？

一个 package 提供了一组关联的 Python 代码的简单复用方式。一个包（“模块”）包含了一个或多个 Python 代码文件。

一个包通过 import foo.bar 或 from foo import bar 的形式导入。一个目录（例如 polls）要成为一个包，它必须包含一个特定的文件 __init__.py，即便这个文件是空的。

Django 应用 仅仅是专用于 Django 项目的 Python 包。应用会按照 Django 规则，创建好 models, tests, urls, 以及 views 等子模块。

稍后，我们将解释术语 打包 ——为了方便其它人安装 Python 包的处理流程。我知道，这可能会使你感到一点点迷惑。

```

### 你的项目和可复用应用  
通过前面的教程，我们的工程应该看起来像这样:

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
    polls/
        __init__.py
        admin.py
        apps.py
        migrations/
            __init__.py
            0001_initial.py
        models.py
        static/
            polls/
                images/
                    background.gif
                style.css
        templates/
            polls/
                detail.html
                index.html
                results.html
        tests.py
        urls.py
        views.py
    templates/
        admin/
            base_site.html

```

你在 教程 7 中创建了 mysite/templates，在 教程 3 中创建了 polls/templates。现在也许更清楚为什么我们选择为项目和应用程序设置单独的模板目录：所有属于 polls 应用程序的部分都在 polls 中。这使得应用程序自成一体，更容易放到一个新项目中。

目录 polls 现在可以被拷贝至一个新的 Django 工程，且立刻被复用。不过现在还不是发布它的时候。为了这样做，我们需要打包这个应用，便于其他人安装它。

### 安装必须环境  

Python 打包的現況有點混亂，工具繁多。 在本教程中，我們將使用 setuptools 來建立我們的套件。 它是推薦的打包工具（與分發叉合併）。 我們還將使用 pip 來安裝和卸載它。 您現在應該安裝這兩個軟體包。 如果需要協助，可以參考如何使用pip安裝Django。 您可以用同樣的方式安裝setuptools。

### 打包你的应用  

Python 的 打包 将以一种特殊的格式组织你的应用，意在方便安装和使用这个应用。Django 本身就被打包成类似的形式。对于一个小应用，例如 polls，这不会太难。

1. 首先，在你的 Django 项目目录外创建一个名为 django-polls 的文件夹，用于盛放 polls。

```
为你的应用选择一个名字

当为你的包选一个名字时，避免使用像 PyPI 这样已存在的包名，否则会导致冲突。当你创建你的发布包时，可以在模块名前增加 django- 前缀，这是一个很常用也很有用的避免包名冲突的方法。同时也有助于他人在寻找 Django 应用时确认你的 app 是 Django 独有的。

应用标签（指用点分隔的包名的最后一部分）在 INSTALLED_APPS 中 必须 是独一无二的。避免使用任何与 Django contrib packages 文档中相同的标签名，比如 auth，admin，messages。

```

2. 将 polls 目录移入 django-polls 目录。

3. 创建一个名为 django-polls/README.rst 的文件，包含以下内容：

django-polls/README.rst
```
=====
Polls
=====

Polls is a Django app to conduct web-based polls. For each question,
visitors can choose between a fixed number of answers.

Detailed documentation is in the "docs" directory.

Quick start
-----------

1. Add "polls" to your INSTALLED_APPS setting like this::

    INSTALLED_APPS = [
        ...,
        "polls",
    ]

2. Include the polls URLconf in your project urls.py like this::

    path("polls/", include("polls.urls")),

3. Run ``python manage.py migrate`` to create the polls models.

4. Start the development server and visit http://127.0.0.1:8000/admin/
   to create a poll (you'll need the Admin app enabled).

5. Visit http://127.0.0.1:8000/polls/ to participate in the poll.
```

4. 创建一个 django-polls/LICENSE 文件。选择一个非本教程使用的授权协议，但是要足以说明发布代码没有授权证书是 不可能的 。Django 和很多兼容 Django 的应用是以 BSD 授权协议发布的；不过，你可以自己选择一个授权协议。只要确定你选择的协议能够限制未来会使用你的代码的人。

5. 接下来我们将创建 pyproject.toml、setup.cfg 和 setup.py 文件，创建 django-polls/pyproject.toml、django-polls/setup.cfg 和 django-polls/setup.py 文件，内容如下：
django-polls/pyproject.toml
```

[build-system]
requires = ['setuptools>=40.8.0']
build-backend = 'setuptools.build_meta'
```

django-polls/setup.cfg
```
[metadata]
name = django-polls
version = 0.1
description = A Django app to conduct web-based polls.
long_description = file: README.rst
url = https://www.example.com/
author = Your Name
author_email = yourname@example.com
license = BSD-3-Clause  # Example license
classifiers =
    Environment :: Web Environment
    Framework :: Django
    Framework :: Django :: X.Y  # Replace "X.Y" as appropriate
    Intended Audience :: Developers
    License :: OSI Approved :: BSD License
    Operating System :: OS Independent
    Programming Language :: Python
    Programming Language :: Python :: 3
    Programming Language :: Python :: 3 :: Only
    Programming Language :: Python :: 3.8
    Programming Language :: Python :: 3.9
    Topic :: Internet :: WWW/HTTP
    Topic :: Internet :: WWW/HTTP :: Dynamic Content

[options]
include_package_data = true
packages = find:
python_requires = >=3.8
install_requires =
    Django >= X.Y  # Replace "X.Y" as appropriate
```
django-polls/setup.py
```
from setuptools import setup

setup()
```
6. 默认情况下，包中仅包含 Python 模块和包。 要包含其他文件，我们需要创建一个 MANIFEST.in 文件。 上一步中提到的 setuptools 文档更详细地讨论了这个文件。 要包含模板、README.rst 和我们的 LICENSE 文件，创建一个文件 django-polls/MANIFEST.in ，其内容如下：
django-polls/MANIFEST.in
```
include LICENSE
include README.rst
recursive-include polls/static *
recursive-include polls/templates *
```
7. 在应用中包含详细文档是可选的，但我们推荐你这样做。创建一个空目录 django-polls/docs 用于未来编写文档。额外添加一行至 django-polls/MANIFEST.in：
```
recursive-include docs *
```
注意，现在 docs 目录不会被加入你的应用包，除非你往这个目录加几个文件。许多 Django 应用也提供他们的在线文档通过类似 readthedocs.org 这样的网站。

8. 试着用 python setup.py sdist 构建你的应用包（在 django-polls 目录运行）。这将创建一个名为 dist 的目录，并构建你的新应用包，django-polls-0.1.tar.gz。

##  使用你自己的包名  
由于我们把 polls 目录移出了项目，所以它无法工作了。我们现在要通过安装我们的新 django-polls 应用来修复这个问题

```
作为用户库安装

以下步骤将 django-polls 以用户库的形式安装。与安装整个系统的软件包相比，用户安装具有许多优点，例如可在没有管理员访问权的系统上使用，以及防止应用包影响系统服务和其他用户。

请注意，按用户安装仍然会影响以该用户身份运行的系统工具的行为，因此使用虚拟环境是更可靠的解决方案（请参见下文）。
```
1. 为了安装这个包，使用 pip (你早已 安装 pip, 对吗？):

```
python -m pip install --user django-polls/dist/django-polls-0.1.tar.gz
```

2. 幸运的话，你的 Django 项目应该再一次正确运行。启动服务器确认这一点。

3. 通过 pip 卸载包:
```
python -m pip uninstall django-polls
```

## 发布你的应用  
现在，你已经对 django-polls 完成了打包和测试，准备好向世界分享它！如果这不是一个例子应用，你现在就可以这样做。

- 通过邮件将你的包发送给朋友。
- 将这个包上传至你的网站。
- 将你的包发布至公共仓库，比如 the Python Package Index (PyPI)。 packaging.python.org 有一个不错的 教程 说明如何发布至公共仓库。

## 通过虚拟环境安装 Python 包  
早些时候，我们以用户库的形式安装了投票应用。这样做有一些缺点。

- 修改用户库会影响你系统上的其他 Python 软件。
- 你将不能运行此包的多个版本（或者其它用有相同包名的包）。


通常，只有在维护多个 Django 项目时才会出现这些情况。当这样做时，最好的解决方法是使用 venv。使用此工具，你可以维护多个隔离的 Python 环境，每个环境都有其自己的库和包命名空间的副本。

（十）- 2535

## 下一步看什么  

且对继续使用 Django 感兴趣。不过，你读的是整体文档的精简版（实际上，如果你逐字阅读了此文档，你已经阅读了整体文档的 5%）。

那么下一步做什么？

不错，我们已经通过边学边做成为了 Django 的死忠粉了。此时，你应该已经知道如何开始你自己的工程，且会到处搜索其它文档了。想知道更多技巧的话，请回到文档页。

为了使 Django 的文档达到更易使用，更清晰，更加全面的目的，我们付出了巨大的努力。本文档的剩余部分将更详细地介绍此文档的工作方式，以便你可以充分利用它。

（是的，这就是传说中的说明文档的说明文档。请放心，我们不打算写一篇关于如何阅读此文档的文档。）

##  查找文档  

Django 有 许多 文档——差不多由 450,000 字（英文单词）——所以查找你需要的文档可能需要点技巧。有一个好的起点 索引。我们还建议使用内置搜索功能。

或者你可以只是四处看看！

##  文档是如何组成  

Django的主要文档以“块”的形式划分，用于满足不同的需求：

- 介绍文档 是为刚接触 Django 或一般的网络开发人员设计的。它并不涉及任何深入的内容，只是以高度概括的方式介绍了如何以 Django “风格”开发应用。

- 另一方面，The 主题指南 则深入介绍 Django 的各个部分。那里有更多完整的关于 Django的 模板系统, 模板引擎, 表单框架 和其它东西的信息。

- 这可能是你会花费你大部分时间的地方。如果你详细阅读了这些指南，你就能了解几乎所有关于 Django 的知识。

- Web 开发通常是广而不深的——问题通常跨越多个领域。我们已经写了一系列的文档 how-to 指引 用于回答常见的“为什么我会……？”系列问题。在这里，你可以找到关于 通过 Django 生成 PDF，定义自定义模板标签 的文档，当然，还有其它的文档。


- 这个引导和怎么做的文档不会覆盖 Django 中每个可用的类，函数和方法——在你想学的时候，你会发现实在是太多了。作为替代，每个类，函数，方法和模块的细节在 参考 中介绍。那是你未来查找某个函数的细节或其它你需要的东西的地方。

- 如果你对部署一个公用的工程感兴趣，我们有介绍各种部署设置的文档 几个指引 和介绍你几个你需要了解的东西的文档 部署清单。

- 最后，这里有一些与大部分开发者无关的“专业”文档。包含 发布说明 和 内部文档 ，用于向那些想向 Django 提交代码的专业人士。还有一份文档 不适合放到其它地方的一点内容 。

## 这个文档是如何更新的  

就像 Django 的源码每天都被更新和提升一样，我们的文档也会持续优化。我们因为以下几个原因优化文档：

- 更正内容，如更正语法/拼写错误。
- 向已有的需要被扩展的某个章节添加介绍信息或例子。
- 记录尚未记录的 Django 功能。 （这些功能的列-表正在缩小，但仍然存在。）
- 在新特性被添加时，或 Django 的 API 或行为有变化时，会添加新文档。
Django 的文档以和它的代码一样，以代码版本管理系统方式进行管理。它被保存在 git 仓库的 docs 目录内。每份在线文档都是仓库内的一份独立文本文件。

##  从哪里获取这个  

你可以以好几种形式阅读 Django 的文档。他们按照优先顺序排列：

##在网络上  
Django 最新的在线版文档位于 https://docs.djangoproject.com/en/dev/。这些网页由 Django 的源码控制系统中的纯文本文件自动生成。这意味着它们展示了 Django “最新最好” 的修改——它们包含最新的更正和补充，并讨论了最新的Django功能，这些功能只可供Django开发版的用户使用。（参见以下关于“不同版本之间的差异”的介绍。）

为提高文档质量，你可以选择在 工单系统 中提交变更，修正以及建议，为此我们将十分欣喜。 Django 的开发者们会积极的监控工单系统，并使用你的反馈为大家改善文档。

值得一提的是，工单(ticket)应该明确地关联到文档，而不是询问笼统的技术支持问题。 如果你需要针对你的 Django 配置寻求帮助，尝试联系 django-users 邮件组 或者 #django IRC channel 。

## 纯文本形式  
离线阅读，或仅仅是为了方便，你可以阅读 Djano 文档的纯文本形式。

如果你正在使用的是 Django 的某个正式发布版，注意有一个代码压缩包，包含了 docs/ 目录，内含这个版本的完整文档。

如果你使用的是 Django 的开发版（也就是 main 分支），docs/ 目录下包含了所有的文档。你可以更新你的 Git checkout 来获取最新的变化。

一种没啥技术含量的利用纯文本文档的方式是使用 Unix 的 grep 工具在文档中全局中搜索一个短语。举个例子，接下来会向你展示 Django 文档中所有提到这个特定短语 "max_length" 的地方：

```
$ grep -r max_length /path/to/django/docs/
```

## 以本地网页形式阅读  
经过几步操作，你可以获得一份网页文档的副本：
- Django 文档使用了一个叫做 Sphinx 的系统将纯文本转换为网页。你可以通过 Sphinx 的官方网站或 pip 来下载和安装它：
```
python -m pip install Sphinx
```
-  接着，使用其中的 Makefile 工具将文档转换为网页：
```
$ cd path/to/django/docs
$ make html
```
你需要为此安装 GNU Make 工具。
如果你是 Windows 系统，你应该使用其中的批处理文件
```
cd path\to\django\docs
make.bat html
```
- 这个 HTML 文档将会被放置在 docs/_build/html。

## 版本之间的差异  
Git 仓库主分支中的文本文档包含了 “最新和最大” 的变更和新增。这些变化包括针对 Django 下一个 特性版本 的新特性文档。因此，值得指出的是，我们的政策是突出 Django 的最新变化和新增功能。

我们遵循以下原则：

- https://docs.djangoproject.com/en/dev/ 上的开发文档来自 main 分支。这些文档对应于最新的功能发布，加上之后框架中增加／修改的任何功能。
- 当我们为 Django 的开发版本添加功能时，我们会在相同的 Git commit 事务中更新文档。
- 为区分文档中修改或新增的内容，我们使用短语 : "New in Django Development version" 来表示其属于还未发布的开发版，使用 "New in version X.Y" 来表示其属于已经发布的某个版本。
- 文档修复和改进可能会被移植到最后一个发布分支，由合并决定，然而，一旦Django的某个版本被：ref:`不再支持<supported-versions-policy>'，该版本的文档将不会得到任何进一步的更新。
- 主文檔網頁包含先前版本文件的連結。 確保您使用的文件版本與您正在使用的 Django 版本相對應！

---