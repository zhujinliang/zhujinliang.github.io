---
layout: post
title: "processlist中需要引起关注的状态"
category: "MySQL"
date: 2015-06-11
---

转载自[MySQL中文网][imysql.com]，原文请见[这里][processlist thread states]。

一般而言，我们在processlist结果中如果经常能看到某些SQL的话，至少可以说明这些SQL的频率很高，通常需要对这些SQL进行进一步优化。

<!-- more -->

今天我们要说的是，在processlist中，看到哪些运行状态时要引起关注，主要有下面几个：

|状态|	建议|
|----|------|
|copy to tmp table|	执行ALTER TABLE修改表结构时建议：放在凌晨执行或者采用类似pt-osc工具
|Copying to tmp table|	拷贝数据到内存中的临时表，常见于GROUP BY操作时建议：创建适当的索引
|Copying to tmp table on disk|	临时结果集太大，内存中放不下，需要将内存中的临时表拷贝到磁盘上，形成 #sql\*.MYD、#sql\*.MYI（在5.6及更高的版本，临时表可以改成InnoDB引擎了，可以参考选项default_tmp_storage_engine）建议：创建适当的索引，并且适当加大sort_buffer_size/tmp_table_size/max_heap_table_size
|Creating sort index|	当前的SELECT中需要用到临时表在进行ORDER BY排序建议：创建适当的索引
|Creating tmp table|	创建基于内存或磁盘的临时表，当从内存转成磁盘的临时表时，状态会变成：Copying to tmp table on disk建议：创建适当的索引，或者少用UNION、视图(VIEW)、子查询(SUBQUERY)之类的，确实需要用到临时表的时候，可以在session级临时适当调大 tmp_table_size/max_heap_table_size 的值
|Reading from net|	表示server端正通过网络读取客户端发送过来的请求建议：减小客户端发送数据包大小，提高网络带宽/质量
|Sending data|	从server端发送数据到客户端，也有可能是接收存储引擎层返回的数据，再发送给客户端，数据量很大时尤其经常能看见备注：Sending Data不是网络发送，是从硬盘读取，发送到网络是Writing to net建议：通过索引或加上LIMIT，减少需要扫描并且发送给客户端的数据量
|Sorting result|	正在对结果进行排序，类似Creating sort index，不过是正常表，而不是在内存表中进行排序建议：创建适当的索引
|statistics|	进行数据统计以便解析执行计划，如果状态比较经常出现，有可能是磁盘IO性能很差建议：查看当前io性能状态，例如iowait
|Waiting for global read lock|	FLUSH TABLES WITH READ LOCK整等待全局读锁建议：不要对线上业务数据库加上全局读锁，通常是备份引起，可以放在业务低谷期间执行或者放在slave服务器上执行备份
|Waiting for tables,Waiting for table| flush	FLUSH TABLES, ALTER TABLE, RENAME TABLE, REPAIR TABLE, ANALYZE TABLE, OPTIMIZE TABLE等需要刷新表结构并重新打开建议：不要对线上业务数据库执行这些操作，可以放在业务低谷期间执行
|Waiting for lock_type lock|	等待各种类型的锁：见下面。


各种锁：

* Waiting for event metadata lock
* Waiting for global read lock
* Waiting for schema metadata lock
* Waiting for stored function metadata lock
* Waiting for stored procedure metadata lock
* Waiting for table level lock
* Waiting for table metadata lock
* Waiting for trigger metadata lock

建议：比较常见的是上面提到的 global read lock 以及 table metadata lock，建议不要对线上业务数据库执行这些操作，可以放在业务低谷期间执行。

如果是table level lock，通常是因为还在使用MyISAM引擎表，赶紧转投InnoDB引擎吧，别再老顽固了。

更多详情可参考官方手册：[8.14.2 General Thread States][mysql general thread states]

[imysql.com]: http://imysql.com
[processlist thread states]: http://imysql.com/2015/06/10/mysql-faq-processlist-thread-states.shtml
[mysql general thread states]: http://dev.mysql.com/doc/refman/5.6/en/general-thread-states.html
