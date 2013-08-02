---
layout: post
title: "Python的psutil模块---用于获得处理器和系统相关信息"
description: "psutil是一个用于获得处理器和系统相关信息的模块。"
category: "Python"
tags: [Python, System Status]
---
{% include JB/setup %}

## psutil简介

[`psutil`][1]是个Python模块，它提供了Python接口，可以用来获取信息，包括：

* Process 当前运行的进程
* System 系统（资源使用）信息
* CPU
* Memory 内存
* Disks 磁盘
* Network 网络
* Users 用户

它已经实现了很多功能，包括如下工具所具有的：

* ps
* top
* df
* netstat
* who
* kill
* uptime
* free
* lsof
* ifconfig
* nice
* ionice
* iostat
* iotop
* pidof
* tty
* taskset
* pmap 

所以我们就可以通过Python来实现获取各种系统状态信息的功能。

虽然Python自带的os模块中也有一些接口可以获取系统信息比如`os.get_loadavg()`
之类的，但是还是没有psutil的功能齐全。推荐大家使用一下。

## 安装psutil
推荐使用pip来安装:

    pip install psutil

当然也可以通过源码安装：

1) [下载][2]源码文件

2) 解压：

    tar -zxvf psutil-1.0.1.tar.gz 

3) 安装：
    
    python setup.py install

## 使用psutil
`psutil`的[官方文档][3]提供了很多Example，基本上稍微看一下应该就会了的。

配合详尽的[API文档][4]，结合自己的需求可以使用更多功能。

这里给出一个例子，是我在最近的一个项目中用到：

{% highlight python %}
# -*- coding: utf-8 -*-

import os
import psutil as ps
from datetime import datetime, date
from django.conf import settings
from django.template import loader
from xxx.sendmail import mailto


def monitor_server():
    ''' Monitor the status of CPU, memory of server.
    If status is dangerous, send mail to admins and restart the apache service.
    '''

    SYSTEM_STATUS_LIMIT = {
        'memory': 100,
        'avg_1': 3.9,
        'avg_5': 3.9,
        'disk': 5,
    }

    danger = False

    # Get cpu info
    cpu_percent = ps.cpu_times_percent(interval=0)
    cpu = {
        'user': cpu_percent.user,
        'system': cpu_percent.system,
        'idle': cpu_percent.idle
    }

    # Get system average load
    avg_1, avg_5, avg_15 = os.getloadavg()
    load_avg = {
        'avg_1': avg_1,
        'avg_5': avg_5,
        'avg_15': avg_15
    }

    # Get memory info. Unit is MB.
    mem = ps.virtual_memory()
    memory = {
        'total': mem.total / 1000000,
        'used': mem.used / 1000000,
        'available': mem.available / 1000000,
        'free': mem.free / 1000000,
        'cached': mem.cached / 1000000,
        'percent': mem.percent
    }

    # Get disk info. Unit is GB.
    home = ps.disk_usage('/home')
    disk = {
        'total': home.total / 1000000000,
        'used': home.used / 1000000000,
        'free': home.free / 1000000000,
        'percent': home.percent
    }

    # Check the status of server
    if memory['available'] < SYSTEM_STATUS_LIMIT['memory']:
        danger = True
        message = u'内存快要耗尽！请尽快解决！'
    if load_avg['avg_1'] > SYSTEM_STATUS_LIMIT['avg_1'] and \
        load_avg['avg_5'] > SYSTEM_STATUS_LIMIT['avg_5']:
        danger = True
        message = u'CPU负载很重，服务器剧慢！请尽快查找原因解决'
    if disk['free'] < SYSTEM_STATUS_LIMIT['disk']:
        danger = True
        message = u'硬盘资源快要耗尽，请尽快解决！'

    # Record log info.
    log = u'\n时间: %s \n' % datetime.now()
    log += u'''
    CPU使用率信息:
        User: %s
        System: %s
        Idle: %s
    CPU平均负载信息:
        1 min: %3.2f
        5 min: %3.2f
        15 min: %3.2f
    内存信息:
        Total: %d MB
        Used: %d MB
        Available: %d MB
        Free: %d MB
        Cached: %d MB
        Percent: %3.2f
    磁盘信息（/home目录）：
        Total: %d G
        Used: %d G
        Free: %d G
        Percent: %3.2f
    ''' % (cpu['user'], cpu['system'], cpu['idle'], load_avg['avg_1'], load_avg['avg_5'], load_avg['avg_15'], memory['total'], memory['used'], memory['available'], memory['free'], memory['cached'], memory['percent'], disk['total'], disk['used'], disk['free'], disk['percent'])

    if danger:
        log += message
        subject = u'服务器异常——公平价'
        context = {
            'memory': memory,
            'cpu': cpu,
            'load_avg': load_avg,
            'disk': disk,
            'message': message,
        }
        content = loader.render_to_string('email/monitor_server.html', context)
        admin_mails = [ad[1] for ad in settings.ADMINS]
        mailto(subject, content, admin_mails)


    filename = '/var/server_log/%s' % str(date.today())
    print log
    print filename
    f = open(filename, 'a')
    try:
        f.write(log.encode('utf-8'))
    except:
        pass
    finally:
        f.close()
{% endhighlight %}

这个例子中只是做了简单的系统信息的记录和检查，并及时报警。

可以把这个function通过celery来异步调度，实现定时执行该function，
获取系统信息，达到监控的目的。

欢迎一起探讨。


[1]: http://code.google.com/p/psutil/
[2]: http://code.google.com/p/psutil/downloads/list
[3]: http://code.google.com/p/psutil/
[4]: http://code.google.com/p/psutil/wiki/Documentation
