---
layout: post
title: "South在Django中使用"
description: "针对django自带的syncdb同步models和数据库的缺陷开发的数据迁移工具。
South可以作为syncdb的替代，South能够检测对models的更改并同步到数据库。"
category: "Django"
tags: [South, Django]
---
{% include JB/setup %}

## South概述
针对django自带的syncdb同步models和数据库的缺陷开发的数据迁移工具。

South可以作为syncdb的替代，South能够检测对models的更改并同步到数据库。

[South的Wiki](http://south.aeracode.org/)

[South的文档](http://south.readthedocs.org/en/latest/)

## South基本用法
新建的项目使用South：

1. 安装完South之后，要在django项目中使用South，先要将South作为一个App导入项目，所以在Django项目的settings.py中的`INSTALL_APP`下添加`south`。

2. Django中本身的app，还是可以通过Django自带的命令来同步到数据库中（包括South app的`south_migrationhistory`表也是此时创建）

        python manage.py syncdb  

3. 对于Django项目中自己创建的app，需要通过以下命令来使用South初始化。

        python manage.py schemamigration yourappname --initial    # 会在youappname目录下面创建一个migrations的子目录，含有migrations包以后每次对models更改后，可以运行以下两条命令同步到数据库     
        python manage.py schemamigration  --auto yourappname      # 检测对models的更改  
        python manage.py migrate yourappname                      # 将更改反应到数据库  

4. 已有的项目中使用South，假设一个已存在的项目（定义了models，创建了相应的数据库，保存了响应的数据），这时想要使用South替代原来的syncdb，只需要一些简单的步骤：

* 先在`INSTALL_APP`里面添加south
* 然后执行以下命令 `convert_to_south`：

        python manage.py syncdb      # syncdb已经被South更改，用来创建south_migrationhistory表  
 
        python manage.py convert_to_south yourappname   # 在yourappname目录下面创建migrations目录以及第一次迁移需要的migration包  

## South进阶

当多人开发项目时，可能另一个人在他自己的数据库上执行了migration，之后又migrate更新到了数据库中，此时会出现migration文件目录下的不一致，此时自己更改了models.py之后，执行migration，产生了migration文件，但当你执行migrate想更新数据库时，会报错，可能的原因是migration文件与数据库中south_migrationhistory的不一致造成的。

若本地的app目录下有部分migrations文件，则解决办法如下：

1. 查看数据库`south_migrationhistory`表中`migration`字段的值对应的和当前要migrate的app下的migration文件夹下的一致的最后一个文件名的。

2. 删除`south_migrationhistory`表中，`migration`字段对应的文件名，在那之后不一致的所有记录。

3. 此时执行migration操作，生成migration文件。
 
        python manager.py schemamigration --auto app_name

4. 执行如下命令,告诉South，之前的所有migration文件已经应用到数据库中，数据库已经更新了（fake，其实就是欺骗South）。`migration_file_num`是现在migration文件夹下的文件中，id号最大的id。

        python manage.py migrate app_name migration_file_num --fake

5. 之后，便可以重新修改models.py文件，然后按一般使用South的方法进行了。

若本地的app目录下有没有migrations文件（一般migrations文件都是不加入到版本控制中的），则解决办法如下：

1. 为该app生成migration文件，一般会在该app目录下创建migrations文件夹，并有初始文件名为0001_initial.py：

        python manage.py schemamigration --init app_name

2. 删除`south_migrationhistory`表中，`app_name`字段为该app，且`migration`字段对应的文件名不是0001_initial.py的所有记录。

3. 执行如下命令,告诉South，之前的所有migration文件已经应用到数据库中，数据库已经更新了（fake，其实就是欺骗South）。

        python manage.py migrate app_name 0001 --fake

4. 之后，便可以重新修改models.py文件，然后按正常使用South的方法进行了。


欢迎指出问题，相互交流学习！
