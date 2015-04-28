---
layout: post
title: "MySQL主从同步机制与配置"
category: "MySQL"
date: 2014-04-28
---

MySQL的Replication是一种多个MySQL的数据库做主从同步的方案，特点是异步，广泛用在各种对MySQL有更高性能，更高可靠性要求的场合。与之对应的另一个技术是同步的MySQL Cluster，但因为比较复杂，使用者较少。 

我们为什么要用MySQL Replication呢？当你的网站一天的独立IP流量能达到100W时，全部流量能达到1000W时，单个服务器，根本无法满足现有需要，100W，1000W就是举个例子。尽管你做了许多的技术上面的措施，比如内存缓存（如memcache），文件缓存啊，对于大数据量表进行分表啊等等，还是网站还是挺慢的（这可能有多方面原因），单个数据库已经远远不能满足需要，需要多个数据库集群做支撑。有了多台数据库做支撑，后面还可以做数据库读写分离。

下图是MySQL官方给出了使用Replication的场景：
![Mysql Replication场景][1]

<!-- more -->

## Replication原理 
  
Mysql 的 Replication 是一个异步的复制过程，从一个MySQL节点（称之为Master）复制到另一个MySQL节点（称之Slave）。在 Master 与 Slave 之间的实现整个复制过程主要由三个线程来完成，其中两个线程（SQL 线程和 I/O 线程）在 Slave 端，另外一个线程（I/O 线程）在 Master 端。 
  
要实现 MySQL 的 Replication ，首先必须打开 Master 端的 Binary Log，因为整个复制过程实际上就是 Slave 从 Master 端获取该日志然后再在自己身上完全顺序的执行日志中所记录的各种操作。 
  
看上去MySQL的Replication原理非常简单，总结一下： 

* 每个从仅可以设置一个主。 
* 主在执行sql之后，记录二进制log文件（bin-log）。 
* 从连接主，并从主获取binlog，存于本地relay-log，并从上次记住的位置起执行sql，一旦遇到错误则停止同步。 
   
从这几条Replication原理来看，可以有这些推论： 

* 主从间的数据库不是实时同步，就算网络连接正常，也存在瞬间，主从数据不一致。 
* 如果主从的网络断开，从会在网络正常后，批量同步。 
* 如果对从进行修改数据，那么很可能从在执行主的bin-log时出现错误而停止同步，这个是很危险的操作。所以一般情况下，非常小心的修改从上的数据。 
* 一个衍生的配置是双主，互为主从配置，只要双方的修改不冲突，可以工作良好。 
* 如果需要多主的话，可以用环形配置，这样任意一个节点的修改都可以同步到所有节点。 

## Replication配置

因为原理比较简单，所以Replication从MySQL 3就支持，并在所有平台下可以工作，多个MySQL节点甚至可以不同平台，不同版本，不同局域网。

做Replication配置包括用户和my.cnf）两处设置。 

现在假设已经有两台服务器已经安装了MySQL，为了避免不必要的麻烦，主从服务器MySQL版本尽量保持一致，这里以MySQL 5.1.48为例，在5.1.7版本之后，有些配置上的差异，后面会提到。

环境：

* 192.168.0.1 （Master）
* 192.168.0.2 (Slave)

1. 登录Master服务器，修改`my.cnf`，添加如下内容:

		
		server-id = 1   # 数据库ID号， 为1时表示为Master,其中master_id必须为1到232–1之间的一个正整数值; 
		binlog-do-db=test # 需要同步的二进制数据库名；如果需要备份多个数据库，那么应该写多行。例如：binlog-do-db=test1
		auto_increment_increment  # 定义下一次AUTO_INCREMENT的步长
		auto_increment_offset=1  # 定义AUTO_INCREMENT的起点值；这两个参数用于避免多master情况下，多个master同时存取字段类型为AUTO_INCREMENT的冲突。其它几个字段为日志文件配置信息
		log-bin=/home/mysql/updatelog # 启用二进制日志；打开该选项才可以通过I/O写到Slave的relay-log,也是可以进行replication的前提;
		log-bin-index=/home/mysql/master-log-bin.index  
		log-error=/home/mysql/master-error.log  
		relay-log=/home/mysql/slave-relay.log  
		relay-log-info-file=/home/mysql/slave-relay-log.info  
		relay-log-index=/home/mysql/slave-relay-log.index  
		

2. 在Master上建立所需要的用户。

		
		mysql>create user 'user1'@'%' identified by '****';
		mysql>grant all on *.* to 'user1'@'%';
		

3. 备份Master上得数据库，并在Slave上导入。具体的备份以及之后的还原方法，请见[这里][2]
4. 登录Slave数据库服务器，测试在Slave服务器上可以通过user1登录Master数据库。

		
		mysql -u user1 -p -h 192.168.0.1
		
5. 修改`my.cnf`;

		
		server-id=2   # 如果以后要再加Slave号接着往后数就OK了；
		log-bin=/home/mysql/master-bin.log  
		log-bin-index=/home/mysql/master-log-bin.index  
		log-error=/home/mysql/master-error.log  
		relay-log=/home/mysql/slave-relay.log  
		relay-log-info-file=/home/mysql/slave-relay-log.info  
		relay-log-index=/home/mysql/slave-relay-log.index 
		master-host=192.168.0.1   # 前面Master主机的IP
		master-user=user1         # 前面Master上创建的user1账号
		master-password=******
		master-port=3306
		master-connect-retry=60   # 如果发现主服务器断线，重新连接的时间差； 
		replicate-do-db=test      # 需要备份的数据库
		log-slave-update     # 从master读到的更新操作都记录到slave二进制日志中
		slave-skip-errors    # 跳过错误，继续执行复
		

6. 两台服务器都重启MySQL。

	
	service mysqld restart
	

7. 登录Master服务器，查看master状态。

		mysql> show master status;
		+-------------------+----------+--------------+------------------+
		| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB |
		+-------------------+----------+--------------+------------------+
		| master-bin.000072 |       98 | test1        |                  |
		+-------------------+----------+--------------+------------------+
		1 row in set (0.00 sec)

	由此可见两者的File、Position存在问题，所要要去Slave上设置对应主库的`Master_Log_File`、`Read_Master_Log_Pos`；

8. 登录Slave服务器，操作：

		
		mysql> change master to Master_Log_File='mysql-bin.000072',Master_Log_Pos=98;
		mysql> slave start;
		mysql> show slave status\G;

		*************************** 1. row ***************************
		Slave_IO_State: Waiting for master to send event
		Master_Host: 192.168.0.1
		Master_User: user1
		Master_Port: 3306
		Connect_Retry: 60
		Master_Log_File: master-bin.000072
		Read_Master_Log_Pos: 98
		Relay_Log_File: slave-relay.000330
		Relay_Log_Pos: 244
		Relay_Master_Log_File: master-bin.000072
		Slave_IO_Running: Yes
		Slave_SQL_Running: Yes
		Replicate_Do_DB: test1
		Replicate_Ignore_DB:
		Replicate_Do_Table:
		Replicate_Ignore_Table:
		Replicate_Wild_Do_Table:
		Replicate_Wild_Ignore_Table:
		
	**注意**：在这里Slave_IO_State:后面有可能没有任何参数，或者是Waitting to connect

到这里基本上算是配置好了。现在测试一下吧，在Master上的数据库test1里面，加张表试一下，看看Slave上面有没有。

**注意**：MySQL版本从5.1.7以后开始就不支持“master-host”类似的参数，所以前面的一些配置需要处理下。
Slave库中的`my.cnf`的配置以下几行需要注释:

	
	master-host=192.168.0.1   # 前面Master主机的IP
	master-user=user1         # 前面Master上创建的user1账号
	master-password=******
	master-port=3306
	master-connect-retry=60   # 如果发现主服务器断线，重新连接的时间差；
	

原本的这些配置，可以通过在Slave库中执行以下语句来解决：

	
	mysql> change master to master_host='192.168.0.1', master_user='user1', master_password='******', master_log_file='mysql-bin.000072', master_log_pos=98;
	



[1]: /assets/images/mysql-replication/mysql-replication.gif
[2]: http://zhujinliang.cn/mysql-dump
