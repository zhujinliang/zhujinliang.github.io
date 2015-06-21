---
layout: post
title: "MySQL备份和还原"
category: "MySQL" 
date: 2014-04-22
---


MySQL备份和还原,都是利用`mysqldump`、`mysql`和`source`命令来完成的。 

## 数据库的备份与导入

### 数据库的备份

1.导出整个数据库

    mysqldump -u 用户名 -p 数据库名 > 导出的文件名
    mysqldump -u dbadmin -p myblog > /home/jzhu/blog/database_bak/myblog.sql

2.导出一个表，包括结构和数据

    mysqldump -u 用户名 -p 数据库名 表名> 导出的文件名
    mysqldump -u dbadmin -p myblog wp_users> /home/jzhu/blog/database_bak/blog_users.sql

3.导出一个数据库结构

    mysqldump -u dbadmin -p -d --add-drop-table myblog > /home/jzhu/blog/database_bak/blog_struc.sql
    

4.导出数据库一个表结构

    mysqldump -u dbadmin -p -d --add-drop-table myblog  blog_users> /home/jzhu/blog/database_bak/blog_users_struc.sql

5.导出数据库的一个表的数据
    
    mysqldump　-t　数据库名 表名　-uroot　-p　>　xxx.sql　 

6.导出某个表的部分数据

    mysqldump -u用户名 -p密码 数据库名 表名 --where="筛选条件" > 导出文件路径
    mysqldump -uroot -p123456 myblog blog_users --where=" id > 100" > /tmp/blog_users.sql
说明：

* -d 没有数据，只有表结构
* -t 没有表结构，只有数据
* --add-drop-table 在每个create语句之前增加一个drop table
* --where/-w 用来设定数据导出的条件，使用方式和SQL查询命令中中的where基本上相同。

<!-- more -->

### 数据库的导入
用 `mysqldump` 备份出来的文件是一个可以直接倒入的 SQL 脚本，有两种方法可以将数据导入。

### 方法1
1. 进入到导出的sql文件所在的目录

2. 执行命令，导入sql文件，还原数据库

        mysql -u用户名 -p密码 数据库名 < 需要导入的sql文件
    
    例如：
    
        mysql -u root -p ping < ping.sql
        # 提示输入密码，输入即可

例如：

    #mysql -u root -p *****  myblog   < /home/jzhu/blog/database_bak/myblog.sql

这种方法，我以前经常现在很少用了，因为很容易产生乱码，因为：

* 导出数据库时，你如果忘了设置导出字符集的话，在导入的时候，就有可能会出问题.

* 假如，你导出时设置导出时设置了utf8的编码，但是你又把你的数据库现在的字符集改成了gb2312的.这样又会乱码。

### 方法2
用 `source` 语句
1. 登陆数据库

        mysql -u用户名 -p密码

2. 将备份的导出的sql文件导入还原到数据库中

        source ping.sql

    等再次出现"<mysql>"并且没有提示错误即还原成功。

例如：

    mysql -u dbadmin -p
    
    use myblog;
    
    set names utf8;  #这里的字符集根你的将要导入的数据库的字符集一至。
    
    source /home/jzhu/blog/database_bak/myblog.sql;
    


