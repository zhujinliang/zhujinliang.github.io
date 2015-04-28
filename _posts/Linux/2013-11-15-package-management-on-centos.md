---
layout: post
title: "CentOS下软件包管理"
category: "Linux"
tags: [Linux, rpm, yum]
---


# CentOS下软件包管理

之前一直使用`Ubuntu`，由于工作中经常要接触服务器配置相关工作，使用`CentOS`比较多，
加之之前在某帽子公司时，一直用`Fedora`，`Fedora`和`CentOS`也算是一家人，都是和某帽子公司有渊源，
`Fedora`自从桌面换了Gnome3，效果炫了很多，体验也不错，就是比较吃硬件，于是平时的工作环境便切换到`Fedora`下了，
简单记录软件包安装。

Centos的软件安装大致可以分为两种类型：

```
CentOS                      Ubuntu
rpm文件安装，使用rpm指令        deb文件安装，使用dpkg指令
yum安装                      apt-get安装
```


## rpm指令

### 查询系统装已经安装的软件信息

* 查询系统中已经安装的软件

        rpm -qa 

* 查询一个已经安装的文件属于哪个软件包；

        rpm -qf 文件名的绝对路径

* 查询已安装软件包都安装到**何处**；

        rpm -ql 软件名

* 查询一个已安装软件包的**信息**

        rpm  -qi 软件名

* 查看一下已安装软件的**配置文件**；

        rpm -qc 软件名

* 查看一个已经安装软件的文档安装位置：

        rpm -qd 软件名

* 查看一下已安装软件所依赖的软件包及文件；

        rpm -qR 软件名

<!-- more -->

### 对于未安装的软件包信息查询

* 查看一个软件包的用途、版本等信息；

        rpm -qpi rpm文件

* 查看一件软件包所包含的文件；

        rpm -qpl rpm文件

* 查看软件包的文档所在的位置；

        rpm -qpd rpm文件

* 查看一个软件包的配置文件；

        rpm -qpc rpm文件

* 查看一个软件包的依赖关系

        rpm -qpR rpm文件

 
### 软件包的安装、升级、删除等

* 安装或者升级一个rpm包

        rpm -ivh rpm文件【安装】
        rpm -Uvh rpm文件【更新】

* 删除一个rpm 包

        rpm -e 软件名

 如何需要不管依赖问题，强制删除软件，在如上命令其后加上 `--nodeps`

 

### 签名导入

    rpm --import 签名文件 
    rpm --import RPM-GPG-KEY

 

## yum管理软件
### yum基本概念
**`yum`是什么**

yum = Yellow dog Updater, Modified 
主要功能是更方便的添加/删除/更新RPM包. 
它能自动解决包的依赖性问题. 
它能便于管理大量系统的更新问题

**yum的特点**

* 可以同时配置多个资源库(Repository) 
* 简洁的配置文件`/etc/yum.conf `
* 自动解决增加或删除rpm包时遇到的倚赖性问题 
* 使用方便 
* 保持与RPM数据库的一致性

**yum安装**

CentOS自带(yum-xx.noarch.rpm)

    rpm -ivh yum-xx.noarch.rpm

在第一次启用yum之前首先需要导入系统的RPM-GPG-KEY：

    rpm --import RPM-GPG-KEY

 

### yum指令的使用

当第一次使用yum管理软件时，yum会自动下载所需要的headers放置在/var/cache/yum目录下；

#### rpm包的更新

* 检查可以更新的软件包

        yum check-update

* 更新**所有的软件包**

        yum update

* 更新特定的软件包

        yum update kernel

* 大规模的升级

        yum upgrade

#### rpm包的安装和删除
* rpm包的安装和删除

        yum install xxx【服务名】
        yum remove xxx【服务名】

#### yum缓存的相关信息

* **清楚缓存中rpm包文件**

        yum clean packages

* 清楚缓存中rpm的头文件

        yum clean  headers

* 清除缓存中旧的头文件

        yum clean old headers

* 清除缓存中旧的rpm头文件和包文件

        yum clean all

#### 软件包信息查询

* 列出资源库中所有可以安装或更新的rpm包

        yum list

* 列出资源库中特定的可以安装或更新以及已经安装的rpm包

        yum list firfox*

**注意：**可以在rpm包名中使用通配符,查询类似的rpm包

* 列出资源库中所有可以更新的rpm包

        yum list updates

* **列出已经安装的所有的rpm包**

        yum list installed

* 列出已经安装的但是不包含在资源库中的rpm包

        yum list extras

**注意：**通过如网站下载安装的rpm包

* rpm包信息显示(info参数同list)，列出资源库中所有可以安装或更新的rpm包的信息

        yum info

* 列出资源库中特定的可以安装或更新以及已经安装的rpm包的信息

        yum info firefox*

**注意：**可以在rpm包名中使用匹配符

* 列出资源库中所有可以更新的rpm包的信息

        yum info updates
* 列出已经安装的所有的rpm包的信息

        yum info installed

* 列出已经安装的但是不包含在资源库中的rpm包的信息

        yum info extras

**注意：**通过如网站下载安装的rpm包的信息

* **搜索匹配特定字符的rpm包**

        yum search firofox

* 搜索包含特定文件的rpm包

        yum provides firefox

 

### yum软件源更新
以[网易的镜像][163 mirror]为例。

* 备份原来的源 `/etc/yum.repos.d/CentOS-Base.repo`

        mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

* 下载对应版本repo文件, 放入`/etc/yum.repos.d/`(操作前请做好相应备份)

    * [CentOS5][CentOS5 repo]

    * [CentOS6][CentOS6 repo]

* 运行`yum makecache`生成缓存




[CentOS5 repo]: mirrors.163.com/.help/CentOS5-Base-163.repo
[CentOS6 repo]: mirrors.163.com/.help/CentOS6-Base-163.repo
[163 mirror]: http://mirrors.163.com/.help/centos.html
