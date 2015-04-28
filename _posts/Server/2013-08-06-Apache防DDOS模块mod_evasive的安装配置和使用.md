---
layout: post
title: "Apache防DDOS模块mod_evasive的安装配置和使用"
category: "Apache"
tags: [DDOS Apache]
---



前些天,网站访问很慢,有很多httpd的进程，从`Apache`的访问日志来看，有几个IP一直在不断的访问网站，导致服务器的负荷太重，怀疑有攻击或者是有爬虫在爬我们网站，便在网上搜解决办法，提供的主要方案是使用mod_evasive模块。

## mod_evasive 介绍
`mod_evasive` 是Apache（httpd)服务器的防DDOS的一个模块。对于WEB服务器来说，是目前比较好的一个防护DDOS攻击的扩展模块。虽然并不能完全防御DDOS攻击，但在一定条件下，还是起到缓服Apache（httpd)服务器的压力。如配合iptables、硬件防火墙等防火墙设备配合使用，可能有更好的效果。

mod_evasive 的官方地址： [http://www.zdziarski.com](http://www.zdziarski.com)

## mod_evasive安装配置
* 下载`mod_evasive`

         wget http://www.zdziarski.com/blog/wp-content/uploads/2010/02/mod_evasive_1.10.1.tar.gz
   
* 编译`mod_evasive`

        tar zxvf mod_evasive_1.10.1.tar.gz
        cd mod_evasive
        /www/wdlinux/apache/bin/apxs -i -a -c mod_evasive20.c
   
* 在Apache中配置`mod_evasive`
    
    在/www/wdlinux/apache/conf/http.conf文件中加入以下配置代码

        <IfModule mod_evasive20.c>
           DOSHashTableSize    3097
           DOSPageCount        5
           DOSSiteCount        50
           DOSPageInterval     1
           DOSSiteInterval     1
           DOSBlockingPeriod   360
        </IfModule>

<!-- more -->
## 配置相关参数

* DOSHashTableSize 3097：

    记录和存放黑名单的哈希表大小，如果服务器访问量很大，可以加大该值。   

* DOSSiteCount 50：

    同一个用户在“单位时间”内可以对同一个网站的访问数，超过该数被列为攻击。
   
* DOSPageCount 2：

    同一个页面在”单位时间“内可以被同一个用户访问的次数，超过该数被列为攻击。  
 
*  DOSPageInterval 1：

    设置DOSPageCount中“单位时间”长度标准（单位秒），默认值为1。   

* DOSSiteInterval 1：

    设置DOSSiteCount中“单位时间”的长度标准（单位秒），默认值为1。   

* DOSSiteInterval 60：

    加入黑名单后拒绝访问时间(秒)，这中间会收到 403 (Forbidden) 的返回。   

* DOSEmailNotify xxxx@gmail.com：

    有IP加入黑名单后通知管理员。   

* DOSSystemCommand "sudo iptables -A INPUT -s %s -j DROP"：

    IP加入黑名单后执行的系统命令，可以直接将该IP加入防火墙，永久封IP。   

* DOSLogDir "/www/wdlinux/httpd-2.2.22/logs/mod_evasive"：

    手动创建目录mod_evasive，攻击日志存放目录，注意这个**目录的权限**，是运行apache程序的用户。锁定机制临时目录。   

* DOSWhiteList 127.0.0.1：

    防范白名单，不阻止白名单IP。

## mod_evasive测试验证

1）防DDOS的模块安装配置好后，我们要验证是否工作，可以用Apache 自带的`ab`工具，系统默认安装在/usr/sbin目录中；
   
        /www/wdlinux/apache/bin/ab -n 1000 -c 50 http://****

**Note：**上面的例子的意思是，要发送数据请求包给指定的网站，总共1000个，每次并发50个；

2）另外一个测试工具就是mod_evasive的解压包的目录中`test.pl`， 修改test.pl IP地址为测试目标网站的IP

        chmod 755 test.pl
        ./test.pl
        HTTP/1.1 200 OK
        HTTP/1.1 200 OK
        HTTP/1.1 200 OK
        HTTP/1.1 200 OK
        HTTP/1.1 200 OK
        HTTP/1.1 200 OK
        HTTP/1.1 200 OK
        HTTP/1.1 403 Forbidden
        HTTP/1.1 403 Forbidden
        HTTP/1.1 403 Forbidden
        HTTP/1.1 403 Forbidden
        HTTP/1.1 403 Forbidden
        HTTP/1.1 403 Forbidden
        HTTP/1.1 403 Forbidden
        HTTP/1.1 403 Forbidden

你可以到你的`/www/wdlinux/httpd-2.2.22/logs/mod_evasive`目录(该目录由`DOSLogDir`设定)下面发现有日志文件
类似文件：dos-192.168.12.201 ，192.168.12.201 表示记录了攻击的IP。