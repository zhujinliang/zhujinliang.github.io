---
layout: post
title: "Django-debug-toolbar"
description: "在调试Django的程序时可以显示各种关于当前request/response的调试信息，以及模版渲染，SQL语句查询等信息。"
category: "Django"
tags: [Django]
---

## 介绍

`Django-debug-toolbar`在调试Django的程序时可以显示各种关于当前request/response的调试信息，以及模版渲染，SQL语句查询等信息。

当前可以在调试面板里显示的信息如下：

* Django版本信息  

* `request time`请求时间  

* `settings.py`文件中的设置信息

* GET/POST/cookie/session变量的信息

* 使用的`template`和`context`，以及template的路径

* `SQL`查询的信息，包括查询语句数量，时间等信息。  

* `signal`信号列表  

* `log`日志信息  

就我使用体会，感觉还是非常方便的，大大便利了Django开发过程中的调试，能够帮助开发者及时找到问题所在。

虽然当程序运行到错误的地方时，若当前settings.py文件中Debug=True，Django也会出现调试页面，显示错误的信息，但是就我个人而言，感觉信息还是不如Django-debug-toolbar中显示的详细，而且，很多时候，我们遇到的问题是，程序没有出错，只是没有按照我们预想的运行，得到预定的数据或者结果而已，这个时候，Django-debug-toolbar就会帮上很大的忙。


*Note*: Django-debug-tool只能在Django1.3以上版本使用。


## 安装

* 利用`pip`安装

	pip install django-debug-toolbar

* [下载](https://pypi.python.org/pypi/django-debug-toolbar)Debug-toolbar`源码`，解压到解压目录下使用以下命令安装

	sudo python setup.py install


## 配置

在settings.py文件（建议添加在本地的settings.py文件或者开发使用的settings.py文件）中添加如下配置信息：

* 安装debug_toolbar的app，在`INSTALLED_APPS`中增加如下行：

{% highlight python %}

INSTALLED_APPS += (
    'debug_toolbar',
)

{% endhighlight %}

* 安装debug_toolbar的中间件，在`MIDDLEWARE_CLASSES`中增加如下行：

{% highlight python %}

MIDDLEWARE_CLASSES += (
    'debug_toolbar.middleware.DebugToolbarMiddleware',
)
{% endhighlight %}

*Note*: 如果你的站点启用了压缩中间件：GZipMiddleware，则必须将这一行放到它的后面。

* 增加`INTERNAL_IPS`设置，添加如下行：

{% highlight python %}

INTERNAL_IPS = ('127.0.0.1', )

{% endhighlight %}

* 设置模板，添加debug_toolbar的模板目录到`TEMPLATE_DIRS`：

{% highlight python %}

TEMPLATE_DIRS += (
    os.path.join(PROJECT_ROOT, "../lib/python2.7/site-packages/debug_toolbar/templates"),
)

{% endhighlight %}


*  右侧panel显示的项目是可以定制的，增加如下配置项，将不需要的项目注释掉即可：

{% highlight python %}

# You can change the ordering of this tuple to customize the order of the panels
# you want to display, or add/remove panels.
DEBUG_TOOLBAR_PANELS = (
    'debug_toolbar.panels.version.VersionDebugPanel',
    'debug_toolbar.panels.timer.TimerDebugPanel',
    'debug_toolbar.panels.settings_vars.SettingsVarsDebugPanel',
    'debug_toolbar.panels.headers.HeaderDebugPanel',
    'debug_toolbar.panels.request_vars.RequestVarsDebugPanel',
    'debug_toolbar.panels.template.TemplateDebugPanel',
    'debug_toolbar.panels.sql.SQLDebugPanel',
    'debug_toolbar.panels.signals.SignalDebugPanel',
    'debug_toolbar.panels.logger.LoggingPanel',
)

{% endhighlight %}


*Note*:还可以有更灵活的配置，请点击[这里](https://pypi.python.org/pypi/django-debug-toolbar)。


## 使用方法

* `DJDT`浮动图标

运行Django应用，便可以在浏览器的右上角看到一个浮动的图标，如下图所示：

![浮动图标](/assets/images/django-debug-toolbar/toolbar-hide.png)


* Debug-toolbar调试面板

点击图标之后，会展开整个调试面板，可以选择相关的信息进行查看，如下图所示：

![调试面板](/assets/images/django-debug-toolbar/toolbar-expand.png)

* 以`template`信息为例

点击Template，会显示此Web页面所用到的模版，以及模版中的context，如下图所示：

![template信息](/assets/images/django-debug-toolbar/template-detail.png)

好了，现在就开始Django-debug-toolbar这件利器的体验之旅吧！



## 参考

下载Django-debug-toolbar以及相关文档请点击[这里](https://pypi.python.org/pypi/django-debug-toolbar)
