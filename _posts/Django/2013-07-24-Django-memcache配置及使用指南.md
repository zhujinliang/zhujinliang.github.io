---
layout: post
title: "Django+memcache配置及使用指南"
description: "Django+memcache配置及使用指南"
category: "Django"
tags: [Django, Memcache]
---
{% include JB/setup %}

## Django缓存

Django提供了一个稳定的缓存系统让你缓存动态页面的结果，这样在接下来有相同的请求就
可以直接使用缓存中的数据，避免不必要的重复计算。另外Django还提供了不同粒度数据的缓存，
例如：你可以缓存整个页面，也可以缓存某个部分，甚至缓存整个网站。

Django提供了可用的缓存方案，分别如下：

* 内存缓冲 (Memcached)

* 数据库缓存 (Database caching)

* 文件系统缓存 (Filesystem caching)

* 本地内存缓存 (Local-memory caching)

* 仿缓存 (Dummy caching)（供开发时使用）

目前为止Django可得到的最快的最高效的缓存类型是基于内存的缓存框架Memcached，
这里将介绍Django中如何使用Memcached来提高网站的响应速度。

## Memcached

`Memcached` 是一个高性能的分布式的内存对象缓存系统，用于动态Web应用以减轻数据库负载。
它通过在内存里维护一个统一的巨大的hash表，它能够用来存储各种格式的数据，包括图像、
视频、文件以及数据库检索的结果等。简单的说就是将数据调用到内存中，然后从内存中读取，
从而大大提高读取速度，来减少读取数据库的次数，从而提供动态、数据库驱动网站
的速度。

`Memcached`官方网站：[http://memcached.org](http://memcached.org)

## 安装Memcached

* 安装memcached server端

可以选择下载源码编译安装，也可以直接通过yum安装。建议采用后者，方便处理依赖。

    yum install memcached

* 安装memcached client端

Memcached的守护进程（daemon ）是用C写的，但是客户端可以用任何语言来编写，并通过
memcached协议与守护进程通信。这里我们需要用Python实现的Memcached client端的库。

目前，memcached的Python库，主要有：

  * `python-memcached`

  * `cmemcache`

  * `python-libmemcached`

  * `pylibmc`

针对这些客户端的性能测试，网上有人已经测试过了，测试结果，请见
[这里](http://www.iteye.com/topic/341763)。

从各个测试的结果来看 python-memcached的性能最差，pylibmc的最好，而且cmemcache也还不错。
当然选择最好的这个库pylibmc，pylibmc是用c写的一个快速而且小巧的memcached客户端。

由于pylibmc是构建在libmemcached之上，所以还需要先安装libmemcached。

安装`pylibmc`：

    yum install libmemcached

    pip install pylibmc


## 启动memcached

* 启动`memcached`命令：


        memcached -d -m 64 -u www -l 127.0.0.1 -p 11211 -c 1024 -P /tmp/memcached.pid


    -d选项是启动一个守护进程，

    -m是分配给Memcache使用的内存数量，单位是MB，我这里设的是64MB，

    -u是运行Memcache的用户，我这里是www，

    -l是监听的服务器IP地址，如果有多个地址的话，用空格分隔

    -p是设置Memcache监听的端口，我这里设置了11212，最好是1024以上的端口，

    -c选项是最大运行的并发连接数，默认是1024

    -P是设置保存Memcache的PID文件，我这里是保存在 /tmp/memcached.pid，方便查看PID

* 查看`memcached`

    启动成功后，查看进程状态：

        ps -ef | grep memcached

    查看11212端口状态：

        netstat -ntl

## 在Django中配置memcached

最简单的配置，如下：
{% highlight python %}
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211',
    }
}
{% endhighlight %}

目前项目上，采用的是memcached和文件缓存共存的方法，具体配置如下：

{% highlight python %}
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211',
        'TIMEOUT': 18000, # 5 hours
        'OPTIONS': {
            'MAX_ENTRIES': 1000,
            'CULL_FREQUENCY': 3,
        }
    },

    'file_cache': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': '/home/wwwcache',
        'TIMEOUT': 18000, # 5 hours
        'OPTIONS': {
            'MAX_ENTRIES': 10000,
            'CULL_FREQUENCY': 3,
        }
    }
}
{% endhighlight %}

具体使用时，针对数据（例如需要访问数据库多个表，且需要处理数据）的缓存，可以使用default
的memcache缓存，而对于整个页面的缓存，则可以使用文件缓存的方式来完成。

**Note:**更多配置信息在Django文档中，请见[这里](https://docs.djangoproject.com/en/1.4/topics/cache/)

## 使用memcached

* 在Python交互式命令行中测试memcache是否可以使用:


        In [1]: import memcache
        In [2]: mc = memcache.Client(['127.0.0.1:11211'], debug=0)
        In [3]: mc.set('key_a', 'test')
        Out[3]: True
        In [4]: value = mc.get('key_a')
        In [5]: print value
        test       # 得到结果test  
        In [6]: mc.delete('key_a')
        Out[6]: 1

    **Note:**
    django中memcached对key要求不超过250个字符，不能包含空白和控制字符，

* 在Django中测试memcached

    我们可以在Django shell中，以pricebook.cn为例，先进入Django shell调试：
    
        django-admin.py shell --settings=pingjia.settings_pb --pythonpath=.

    然后测试Django中的对cache的配置是否成功：

    {% highlight python %}

    from django.core.cache import cache
    cache.set('key1', 'test')
    value = cache.get('key1')
    print value
    得到结果test

    {% endhighlight %}

* 在Django中使用memcached

    在项目中，我们主要可以用cache来缓存一些数据，或者整个页面。

    (1) 缓存数据。
    具体的方法和上面提到的类似，例如：

    {% highlight python %}

    from django.core.cache import cache
    def about(request):
        if cache.get('key1'):
            data = cache.get('key1')
        else:
            访问数据库，得到data
            cache.set('data')
        ...

    {% endhighlight %}

    每次进来，先查看cache中是否存在，若存在，直接获得该值，若不存在，则访问数据库，
    得到数据之后，将其写入到cache之后，以便后续之用。

    **Note:**还有其他更多cache的方法，请见
    [这里](https://docs.djangoproject.com/en/1.4/topics/cache/#the-low-level-cache-api)。


    (2)缓存整个页面。
    可以使用Django提供的cache_page装饰函数，例如缓存about页面：

    {% highlight python %}

    from django.views.decorators.cache import cache_page

    @cache_page(60*30)
    def about(request):
        ...

    {% endhighlight %}
    
    只需在view函数前加上cache_page装饰函数便可以该页面，cache_page的参数是缓存时间
    这里设的是60s*30，即30分钟。同时也可以指定使用的缓存，通过cache参数来指定，例如：
    `@cache_page(60*30, cache='file_cache')`则指定使用settings文件中CACHES中的`file_cache`
    来缓存，若不指定，默认用`default`的来缓存。