---
layout: post
title: "SQL基础语法"
date: 2013-05-07
tag: Database
---


### 基本规则
* 数据库中元组对应的是行，属性值对应的是列。

* SQL 对大小写不敏感！

* 大写仅仅是为了区别关键字和普通变量。

* MySQL的SQL语句要求每句后面加分号（；）

### SQL DML 和 DDL
可以把 SQL 分为两个部分：数据操作语言 (DML) 和 数据定义语言 (DDL)。

查询和更新指令构成了 SQL 的 DML 部分：

* SELECT - 从数据库表中获取数据

* UPDATE - 更新数据库表中的数据

* DELETE - 从数据库表中删除数据

* INSERT INTO - 向数据库表中插入数据

SQL 中最重要的 DDL 语句:

* CREATE DATABASE - 创建新数据库

* ALTER DATABASE - 修改数据库

* CREATE TABLE - 创建新表

* ALTER TABLE - 变更（改变）数据库表

* DROP TABLE - 删除表

* CREATE INDEX - 创建索引（搜索键）

* DROP INDEX - 删除索引

<!-- more -->

### SQL SELECT 语句

`SELECT` 语句用于从表中选取数据。

*注释：*SQL 语句对大小写不敏感。SELECT 等效于 select。

结果被存储在一个结果表中（称为结果集）。

*语法：*

    SELECT 列名称 FROM 表名称

以及：

    SELECT * FROM 表名称

从表中选出所有的列。*提示：*星号（ * ) 是选取所有列的快捷方式。

#### SQL SELECT DISTINCT 语句

在表中，可能会包含重复值。这并不成问题，不过，有时您也许希望仅仅列出不同（distinct）的值。

关键词 `DISTINCT` 用于返回唯一不同的值。

*语法：*

    SELECT DISTINCT 列名称 FROM 表名称

如需从 Company" 列中仅选取唯一不同的值，我们需要使用 `SELECT DISTINCT` 语句：

    SELECT DISTINCT Company FROM Orders 

### WHERE 子句

如需有条件地从表中选取数据，可将 WHERE 子句添加到 SELECT 语句。

*语法：*

    SELECT 列名称 FROM 表名称 WHERE 列 运算符 值

下面的*运算符*可在 `WHERE` 子句中使用：

    操作符	    描述
    =	    等于
    <>	    不等于
    >	    大于
    <	    小于
    >=	    大于等于
    <=	    小于等于
    BETWEEN	    在某个范围内
    LIKE	    搜索某种模式

*注释：*在某些版本的 SQL 中，操作符 <> 可以写为 !=。

### 引号的使用

请注意，我们在例子中的条件值周围使用的是单引号。

SQL 使用单引号来环绕文本值（大部分数据库系统也接受双引号）。*如果是数值，请不要使用引号。*

文本值：

这是正确的：

    SELECT * FROM Persons WHERE FirstName='Bush'

这是错误的：

    SELECT * FROM Persons WHERE FirstName=Bush

数值：

这是正确的：

    SELECT * FROM Persons WHERE Year>1965

这是错误的：

    SELECT * FROM Persons WHERE Year>'1965'

### AND 和 OR 运算符

`AND` 和 `OR` 可在 WHERE 子语句中把两个或多个条件结合起来。

如果第一个条件和第二个条件都成立，则 AND 运算符显示一条记录。

如果第一个条件和第二个条件中只要有一个成立，则 OR 运算符显示一条记录。
AND 运算符实例

使用 AND 来显示所有姓为 "Carter" 并且名为 "Thomas" 的人：

    SELECT * FROM Persons WHERE FirstName='Thomas' AND LastName='Carter'

OR 运算符实例

使用 OR 来显示所有姓为 "Carter" 或者名为 "Thomas" 的人：

    SELECT * FROM Persons WHERE firstname='Thomas' OR lastname='Carter'

结合 AND 和 OR 运算符

我们也可以把 AND 和 OR 结合起来（使用圆括号来组成复杂的表达式）:

    SELECT * FROM Persons WHERE (FirstName='Thomas' OR FirstName='William')AND LastName='Carter'

### ORDER BY 语句

`ORDER BY` 语句用于根据指定的列对结果集进行排序。

ORDER BY 语句默认按照*升序*对记录进行排序。

如果您希望按照降序对记录进行排序，可以使用 `DESC` 关键字。

以逆字母顺序显示公司名称：

    SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC

以逆字母顺序显示公司名称，并以数字顺序显示顺序号：

    SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC, OrderNumber ASC

第一列中出现相同的值，才会按照第二列来排序。

### INSERT INTO 语句

`INSERT INTO` 语句用于向表格中插入新的行。

*语法*

    INSERT INTO 表名称 VALUES (值1, 值2,....)

我们也可以指定所要插入数据的列（插入部分列）：

    INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)

### Update 语句

`Update` 语句用于修改表中的数据。WHERE来选择条件。

*语法：*

    UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值

更新某一行中的若干列

对于满足LastName='Wilson'条件的元组（行），我们会修改地址（address），并添加城市名称（city）：

    UPDATE Person SET Address = 'Zhongshan 23', City = 'Nanjing'
    WHERE LastName = 'Wilson'

### DELETE 语句

`DELETE` 语句用于删除表中的行。

*语法*

    DELETE FROM 表名称 WHERE 列名称 = 值

删除某行

"Fred Wilson" 会被删除：

    DELETE FROM Person WHERE LastName = 'Wilson' 

删除所有行

可以在不删除表的情况下删除所有的行。这意味着表的结构、属性和索引都是完整的：

    DELETE FROM table_name

或者：

    DELETE * FROM table_name

### TRUNCATE 语句
删除表内的所有数据，不删表本身，保留表结构：

    TRUNCATE TABLE 表名





