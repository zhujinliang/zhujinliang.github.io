---
layout: post
title: "MySQL用户和权限管理"
category: "MySQL" 
date: 2014-04-20
---

MySQL用户和权限相关的操作介绍。

## 用户相关

### 创建用户

* `CREATE USER`命令:
    
        CREATE USER 'username'@'host' IDENTIFIED BY 'password';

* 说明:

        username - 你将创建的用户名
        host - 指定该用户在哪个主机上可以登陆,如果是本地用户可用localhost, 如果想让该用户可以从任意远程主机登陆,可以使用通配符%.
        password - 该用户的登陆密码,密码可以为空,如果为空则该用户可以不需要密码登陆服务器.

* 例子: 

        CREATE USER 'username'@'localhost' IDENTIFIED BY '123456';
        CREATE USER 'username'@'192.168.1.101' IDENDIFIED BY '123456';
        CREATE USER 'username'@'%' IDENTIFIED BY '123456';
        CREATE USER 'username'@'%' IDENTIFIED BY '';
        CREATE USER 'username'@'%';


### 设置与更改用户密码

* `SET PASSWORD`命令:

        SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');
        # 如果是当前登陆用户用
        SET PASSWORD = PASSWORD("newpassword");

* 例子: 

        SET PASSWORD FOR 'username'@'%' = PASSWORD("123456");

### 删除用户

* `DROP USER`命令: 

        DROP USER 'username'@'host';


## 权限相关

### 授权

* `GRANT`命令:

        GRANT privileges ON databasename.tablename TO 'username'@'host'

* 说明: 

        privileges - 用户的操作权限,如SELECT , INSERT , UPDATE 等(详细列表见该文最后面).如果要授予所的权限则使用ALL.;
        databasename - 数据库名,
        tablename - 表名,如果要授予该用户对所有数据库和表的相应操作权限则可用*表示, 如*.*.

* 例子: 

        GRANT SELECT, INSERT ON test.user TO 'username'@'%';
        GRANT ALL ON *.* TO 'username'@'%';

**注意:** 用以上命令授权的用户不能给其它用户授权,如果想让该用户可以授权,用以下命令:

    GRANT privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION;

**注意：**授权后必须 `FLUSH PRIVILEGES;`否则无法立即生效。 
 

### 撤销用户权限

* `REVOKE`命令: 

        REVOKE privilege ON databasename.tablename FROM 'username'@'host';

* 说明:

        privilege, databasename, tablename - 同授权部分.

* 例子:

        REVOKE SELECT ON *.* FROM 'username'@'%';

**注意:** 

假如你在给用户'username'@'%'授权的时候是这样的(或类似的):

    GRANT SELECT ON test.user TO 'username'@'%';
    
则在使用

    REVOKE SELECT ON *.* FROM 'username'@'%';

命令并**不能撤销**该用户对test数据库中user表的SELECT 操作.

相反,如果授权使用的是

    GRANT SELECT ON *.* TO 'username'@'%';

则

    REVOKE SELECT ON test.user FROM 'username'@'%';

命令也**不能撤销**该用户对test数据库中user表的Select 权限.

具体信息可以用以下命令查看.

    SHOW GRANTS FOR 'username'@'%';


#### 所有权限列表：

privileges是一个用逗号分隔的你想要赋予的权限的列表。你可以指定的权限可以分为三种类型： 


* 数据库/数据表/数据列权限： 

	* Alter: 修改已存在的数据表(例如增加/删除列)和索引。 
	* Create: 建立新的数据库或数据表。 
	* Delete: 删除表的记录。 
	* Drop: 删除数据表或数据库。 
	* INDEX: 建立或删除索引。 
	* Insert: 增加表的记录。 
	* Select: 显示/搜索表的记录。 
	* Update: 修改表中已存在的记录。 

* 全局管理权限： 
 
	* file: 在MySQL服务器上读写文件。 
	* PROCESS: 显示或杀死属于其它用户的服务线程。 
	* RELOAD: 重载访问控制表，刷新日志等。 
	* SHUTDOWN: 关闭MySQL服务。 

* 特别的权限： 
 
	* ALL: 允许做任何事(和root一样)。 
	* USAGE: 只允许登录--其它什么也不允许做。 

`*.*`意味着权限对所有数据库和数据表有效。mysql
`dbName.*`意味着对名为dbName的数据库中的所有数据表有效。 
`dbName.tblName`意味着仅对名为dbName中的名为tblName的数据表有效。
 
user指定可以应用这些权限的用户。在MySQL中，一个用户通过它登录的用户名和用户使用的计算机的主机名/IP地址来指定。这两个值都可以使用%通配符(例如jzhu@%将允许使用用户名jzhu从任何机器上登录以享有你指定的权限)。 
 
password指定了用户连接MySQL服务所用的口令。它被用方括号括起，说明IDENTIFIED BY "<password>"在GRANT命令中是可选项。 
 

