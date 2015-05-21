---
layout: post
title: "MySQL开启慢查询日志"
category: "MySQL"
date: 2014-03-1
---



## 1.开启慢查询作用

开启慢查询之后，它能记录下所有执行超过`long_query_time`时间的SQL语句, 帮你找到执行慢的SQL, 方便我们对这些SQL进行优化.

## 2.开启慢查询

首先我们先查看MySQL服务器的慢查询状态是否开启.执行如下命令:

	show variables like '%quer%';

得到结果如下图：
![查询配置变量][find config]


可以看到当前`log_slow_queries`状态为OFF, 说明当前并没有开启慢查询.

开启慢查询操作如下:

Linux下找到mysql的配置文件`/etc/my.cnf`, 在`mysqld`下方加入慢查询的配置语句(注意:一定要在[mysqld]下的下方加入)

![配置文件配置][how to config]

`log-slow-queries`: 代表MYSQL慢查询的日志存储目录, 此目录文件一定要有写权限；

`long_query_time`: 最长执行时间. (如图, MSYQL将记录下所有执行时间超过2条的SQL语句, 此处为测试时间, 时间不应太小最好在5-10秒之内, 可以根据自己的标准而定);

参考[MySQL5.5 官方手册][MySQL 5.5 mannual]

配置好以后重新启动一个MySQL服务:

	service mysqld restart

### 记录没有使用索引的慢查询日志

在`/etc/my.cnf`中，上面的慢查询配置下面，增加如下配置。

	log-queries-not-using-indexes


## 3.MySQL慢查日志分析工具mysqldumpslow

使用mysqldumpslow命令可以非常明确的得到各种我们需要的查询语句，对MySQL查询语句的监控、分析、优化是MySQL优化的第一步，也是非常重要的一步。

#### 用法

	mysqldumpslow -t 10 path/mysql-slow.log

这会输出记录次数最多的10条SQL语句，其中：

-s, 是表示按照何种方式排序，c、t、l、r分别是按照记录次数、时间、查询时间、返回的记录数来排序，ac、at、al、ar，表示相应的倒叙；

-t, 是top n的意思，即为返回前面多少条的数据；

-g, 后边可以写一个正则匹配模式，大小写不敏感的；

例如：

	mysqldumpslow -s r -t 10 path/mysql-slow.log

得到返回记录集最多的10个查询。

	mysqldumpslow -s t -t 10 -g “left join” path/mysql-slow.log

得到按照时间排序的前10条里面含有左连接的查询语句。


[find config]:/assets/images/mysql-slow-query/find_config.jpg
[how to config]:/assets/images/mysql-slow-query/config.jpg
[MySQL 5.5 mannual]:http://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html
