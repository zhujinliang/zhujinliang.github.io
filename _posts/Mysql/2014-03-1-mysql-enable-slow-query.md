---
layout: post
title: "MySQL开启慢查询日志"
category: "MySQL"
date: 2014-03-1
---



## 慢查询作用

它能记录下所有执行超过`long_query_time`时间的SQL语句, 帮你找到执行慢的SQL, 方便我们对这些SQL进行优化.

## 开启慢查询

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

配置好以后重新启动一个MySQL服务:

	service mysqld restart


[find config]:/assets/images/mysql-slow-query/find_config.jpg
[how to config]:/assets/images/mysql-slow-query/config.jpg
