---
layout: post
title: "Django-celery配置及使用指南"
description: "Django-celery配置及使用指南"
category: "Django"
tags: [Django, Celery]
---
{% include JB/setup %}

## 介绍
Django-celery用作异步任务管理非常方便，借助Django的admin后台管理，可以管理员可以方便的在后台
添加和管理定时的任务，而不用通过Shell的crontab来实现，非常的直观，且方便修改。

关于Django-celery的介绍及下载，请看[这里](https://pypi.python.org/pypi/django-celery)。

最近在一个项目中用到了Django-celery，之前一直使用的是Django的Database作为celery的Broker，
一直没有运行起来，索性按照文档中的说明，按最简单的方式来配置。具体配置过程，请继续阅读。

## 配置
### 配置RabbitMQ

配置Django-celery使用RabbitMQ作为Broker.

* Install RabbitMQ Server

        # yum install rabbitmq-server

* Start RabbitMQ Server

    运行命令，启动RabbitMQ Server

        # rabbitmq-server -detached

    终端中会显示RabbitMQ Server的运行状态信息。

    以后，可以运行命令查看RabbitMQ Server的状态：

        # rabbitmqctl status

### 配置Django-celery
在Django的settings文件中做如下设置：

* 添加djcelery到INSTALLED_APPS

{% highlight python %}
INSTALLED_APPS = {
  ...
  'djcelery',          # 加入celery
}
{% endhighlight %}


* 配置Django-celery

{% highlight python %}
import djcelery
djcelery.setup_loader()
BROKER_URL = 'amqp://guest:guest@localhost:5672//'
CELERYBEAT_SCHEDULER = 'djcelery.schedulers.DatabaseScheduler'
{% endhighlight %}

## 启动Django-celery

在Django中运行以下命令，在后台运行celery

    $ nohup django-admin.py celeryd worker -B --loglevel=info --settings=settings --pythonpath=. &


若要查看Django-celery是否正在运行，可以通过以下命令来查看：

    $ ps aux | grep celery

若没有运行结果，则表示Django-celery没有运行，可以在此执行上面的运行命令运行celery。

若有运行结果，显示celery的进程号等信息，则表示Django-celery已经运行了。

## 使用Django-celery
使用Django-celery可以在admin后台创建一个任务，在指定的时间执行。
使用步骤：

1. 实现指定功能的task

    需要先通过代码实现task的具体功能，之后便可以在admin后台创建任务的时候选择该task。

    **Note:** 要保证实现的task没有错误，否则后面创建了任务之后，是不会达到预期效果的。
    
    **建议:** 实现一个task之后，最好先在django的shell comand中先调试，保证可以成功运行，并达到预期结果了再执行后面的操作。

2. 创建一个任务

    进入Django的admin后台，进入`Djcelery`栏目，一共有四个选项：

* `Crontabs`

在此选项中，可以创建定时执行的任务需要的定时时间。

* `Intervals`

在此选项中，可以创建间隔执行的任务需要时间间隔。

* `Periodic tasks`

在此选项中，创建一个相应的任务，选择任务，需要定时执行的时间或者时间间隔，并保存。

当然，还有其他一些高级的选项，可以尝试使用。

* `Tasks`

* `Workers`

到此，一个任务便创建成功了。不出意外，创建的任务会如期执行，当然，如果创建的没有
如期执行的，应该依次检查前面的步骤，查看celery是否在运行，创建的task是否报错等等，
一级一级排除错误。
