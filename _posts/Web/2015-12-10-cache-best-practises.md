---
layout: post
title: "cache 最佳实践"
category: "Web"
date: 2015-12-10
---


# 简介

Web开发中，性能一直都是被大家所重视的一点，然而判断一个网站或系统的性能最直观的就是看网页打开或接口访问的速度。其中提高系统反应速度的一个有效方式就是使用缓存。一个优秀的缓存策略可以大大降低请求响应时间，提高性能。那么下面我结合所负责的系统（日PV10亿+），来谈谈服务器端缓存使用的一些实践。

Web缓存分为很多种，比如数据库缓存、内存缓存、应用缓存、代理服务器缓存、还有我们熟悉的CDN缓存，以及浏览器缓存。其中代理服务器缓存、还有我们熟悉的CDN缓存，以及浏览器缓存属于偏前端的缓存，这里我们主要关注服务器端的缓存：

* DB缓存
* 内存缓存，如Redis、memcache
* 应用缓存

<!-- more -->

## DB缓存
以MySQL为例，MySQL提供了查询缓存的功能，简单的说，查询缓存就是内存中的一块存储区域，其存储了用户的SQL文本以及相关的查询结果。通常情况下，用户下次查询时，如果所使用的SQL文本是相同的，并且自从上次查询后，相关的纪录没有被更新过，此时数据库就直接采用缓存中的内容。
工作流程:

1. 服务器接收SQL,以SQL和一些其他条件为key查找缓存表(额外性能消耗)
2. 如果找到了缓存,则直接返回缓存(性能提升)
3. 如果没有找到缓存,则执行SQL查询,包括原来的SQL解析,优化等.
4. 执行完SQL查询结果以后,将SQL查询结果存入缓存表(额外性能消耗)

### 配置方法

![MySQL 查询缓存][mysql-query-cache]

```
mysql> SHOW VARIABLES LIKE ‘%query_cache%’;
+——————————+———+
| Variable_name | Value |
+——————————+———+
| have_query_cache | YES | –查询缓存是否可用
| query_cache_limit | 1048576 | –可缓存具体查询结果的最大值
| query_cache_min_res_unit | 4096 |
| query_cache_size | 0 | –查询缓存的大小
| query_cache_type | ON | –阻止或是支持查询缓存
| query_cache_wlock_invalidate | OFF |
+——————————+———+
6 rows in set (0.00 sec)
```

配置如下：

```
set global query_cache_type=1;    -- 设置缓存的类型，缓存所有结果
set global query_cache_size=128M;  -- 查询缓存的大小
```


### 最佳实践
查询缓存失效情况：

当某个表正在写入数据,则这个表的缓存(命中检查,缓存写入等)将会处于失效状态.在Innodb中,如果某个事务修改了表,则这个表的缓存在事务提交前都会处于失效状态,在这个事务提交前,这个表的相关查询都无法被缓存。

在高并发系统中，MySQL存储引擎一般采用Innodb，以提高其并发写入能力，在写操作频繁的业务中，查询缓存基本都是失效的，命中率不高，因此，一般线上不推荐开启查询缓存。一位MySQL大牛专门写了文章分析过，建议线上不要开启查询缓存，[线上环境到底要不要开启query cache][mysql-close-query-cache]

讲了这么多，最后不推荐使用查询缓存，伐开心！


## 内存缓存
内存缓存是Web开发中很常见的方案，可以大幅提高请求的响应速度，一般会采用Redis或memcache作为内存缓存使用，两者的区别这里就不展开了。最近Redis火了之后，且支持的数据类型更多，应该用Redis的更多些。

缓存读取流程：

1. 先到缓存中查数据
2. 缓存中不存在则到实际数据源中取，取出来后放入缓存
3. 下次再来取同样信息时则可直接从缓存中获取

缓存更新流程：

1. 更新数据库
2. 使缓存过期或失效，这样会促使下次查询数据时在缓存中查不到而重新从数据库去一次。

### 最佳实践
* 一方面根据SQL语句查询维度缓存数据库查询结果，另一方面可以根据业务需要，按照业务逻辑维度来缓存结果，避免获取一个数据还要多次请求缓存数据。
* 最好有一个统一维护缓存Key的地方，避免多人协作时key冲突，便于管理，以Python实现为例。


```
class CacheKey(object):
    '''
    Cache key setting.
    '''

    customer_token = 'customer_token:{id}'
    login_captcha = 'login_captcha:{phone}'


使用时：
customer_id = 12
customer_token_key = CacheKey.customer_token.format(id=customer_id)
```

* 通过信号或者触发器的方式，监听数据库数据更新，以便于更新缓存。
* 更新缓存时，如果业务中，对短暂地读取脏数据是可以忍受的，建议更新缓存可以采用异步的方式去更新。


## 本地应用缓存
前面使用内存缓存已经可以大幅提高请求响应速度，但由于请求缓存数据需要连接访问Redis获取数据，当业务逻辑中，响应一个外部请求，需要访问Redis次数过多，且缓存的数据量过大时，会产生大量的网络IO开销。我所经历过的一个项目中，有个高频访问的API的逻辑中，需要访问Redis 60多次，获取缓存数据，造成Redis压力过大，且影响到了其他逻辑请求Redis获取缓存数据，而这个API响应时间达500ms，这时单纯的扩展Redis以提高其并发吞吐量已经效果不大。这时我们的本地应用缓存闪亮登场！！！

本地应用缓存，是应用自己存储在本地应用内存中缓存，即应用自己维护一套数据结构，存储在内存中，同时可供其子线程共享访问。由于其将数据存储在本地，所以访问时不存在任何网络开销，速度极快，不存在任何外部组建依赖。

本地应用缓存使用策略如下：
![本地应用缓存使用策略][local-cache-strategy]


本地应用缓存优点：

* 速度快
* 不依赖外部组建、系统，没有网络传输
* 减小对内存缓存，DB的压力

缺点：

* 本地应用缓存只能应用程序自己去访问和修改，不易管理
* 当在多台服务器上部署应用，或者单台多进程起应用时，每个进程的应用自己维护一套应用缓存，当有数据更新时，不能及时的通知所有应用进程，因而不能保证应用缓存中数据实时更新


### 最佳实践
本地应用缓存虽有数据不方便实时更新的缺点，但我们可以通过其他办法来解决，例如采用zookeeper，所有应用的进程监听zookeeper某个节点的数据变更，当有数据更新时，更新zookeeper该节点数据，zookeeper便会通知所有应用进程，进而可以更新该应用缓存数据。该方案在我所负责的项目中已实际验证过可行。


最后附上本地应用缓存Python代码实现，或见[这里][github-local-cache]：


```
import time


def clear_local_cache(local_cache):
    '''
    Clear the contents of the local cache for the current process.
    '''
    local_cache.__clear_local_cache__()


class LocalCache(object):
    __slots__ = ('__storage__', )
    cache_time = 0

    def __init__(self):
        object.__setattr__(self, '__storage__', {})

    def get_cache_time(self):
        '''
        Override this method to set cache time.
        Unite: second
        '''
        return self.cache_time

    def get_data(self, name):
        '''
        Override this method to get data that need cache.
        '''
        return 1

    def _set_cache(self, name, data):
        storage = self.__storage__
        ts = time.time() + self.get_cache_time()
        storage[name] = (ts, data)

    def clear(self, name):
        '''
        Clear local cache of name key.
        '''
        self.__storage__[name] = (0, 0)

    def __clear_local_cache__(self):
        self.__storage__.clear()

    def __iter__(self):
        return iter(self.__storage__.items())

    def __getattr__(self, name):
        storage = self.__storage__
        try:
            ts, data = storage[name]
            if ts < time.time():
                # cache time out and set new cache.
                data = self.get_data(name)
                self._set_cache(name, data)
        except KeyError:
            data = self.get_data(name)
            self._set_cache(name, data)
        return data

    def __setattr__(self, name, value):
        self._set_cache(name, value)

    def __delattr__(self, name):
        try:
            del self.__storage__[name]
        except KeyError:
            raise AttributeError(name)

    def __getitem__(self, name):
        return self.__getattr__(name)

    def __setitem__(self, name, value):
        self.__setattr__(name, value)

    def __delitem__(self, name):
        self.__delattr__(name)



if __name__ == '__main__':
    import random
    LOCAL_CACHE_TIME = 5  # 5 seconds
    class TestLocalCache(LocalCache):
        cache_time = LOCAL_CACHE_TIME

        def get_data(self, name):
            return random.randrange(1, 100)

    test_cache = TestLocalCache()
    print '==========Get data======='
    for i in range(5):
        print i, test_cache[i]

    print '==========Get data from local cache======='
    for i in range(5):
        print i, test_cache[i]

    print '==========Clear local cache=========='
    clear_local_cache(test_cache)
    for i in range(5):
        print i, test_cache[i]

    print '==========Set local cache======='
    for i in range(5):
        test_cache[i] = 1
        print i, test_cache[i]

    print '==========Clear local cache of key 0======='
    test_cache.clear(0)
    for i in range(5):
        print i, test_cache[i]

    print '==========Wait cache expired======'
    time.sleep(LOCAL_CACHE_TIME)
    print '==========Local cache expired======'
    for i in range(5):
        print i, test_cache[i]
```

运行结果如下：

```
==========Get data=======
0 55
1 17
2 98
3 69
4 62
==========Get data from local cache=======
0 55
1 17
2 98
3 69
4 62
==========Clear local cache==========
0 55
1 39
2 74
3 99
4 45
==========Set local cache=======
0 1
1 1
2 1
3 1
4 1
==========Clear local cache of key 0=======
0 20
1 1
2 1
3 1
4 1
==========Wait cache expired======
==========Local cache expired======
0 8
1 26
2 28
3 8
4 91
```



[mysql-query-cache]: http://7ktw62.com1.z0.glb.clouddn.com/img/blog/web/mysql_query_cache.jpg
[mysql-close-query-cache]: http://imysql.com/2014/09/05/mysql-faq-why-close-query-cache.shtml
[local-cache-strategy]: http://7ktw62.com1.z0.glb.clouddn.com/img/blog/web/local_cache_strategy.png
[github-local-cache]: https://github.com/zhujinliang/python-snippets/blob/master/local_cache.py
