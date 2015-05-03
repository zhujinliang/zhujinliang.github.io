---
layout: post
title: "Linux下修改DNS服务器"
category: "Linux"
date: 2013-03-10
---



大家平时在上网可能会发现，当访问一个不存在（或被封）的网站时，会自动跳转到一个没打算访问的网站，
而这些网站往往充斥着满屏的广告，最常见的如电信114互联星空网站，这些充斥着广告的页面严重影响了我们的上网体验，
这一现象是由于卑鄙的ISP运营商进行DNS劫持造成的！

而Google提供的免费DNS域名解析系统能够帮助我们解决国内电信网通等ISP运营商的DNS劫持问题！

### DNS简介

DNS（Domain Name System）中文意思为域名解析服务器，它在互联网的作用是把域名转换成为网络可以识别的IP地址。
当用户在浏览器中输入网址域名时，首先就会访问系统设置的DNS域名解析服务器（通常由ISP运营商如电信、联通提供）。
如果该服务器内保存着该域名对应的IP信息，则直接返回该信息供用户访问网站。
否则，就会向上级DNS逐层查找该域名的对应数据。

### DNS劫持

目前国内用户普遍使用的是ISP运营商提供的DNS服务器，这样有着一个巨大的风险，就是DNS劫持,
目前国内ISP运营商普遍采用DNS劫持的方法，干扰用户正常上网，例如，当用户访问一个不存在（或者被封）的网站，
电信运营商就会把用户劫持到一个满屏都是广告的页面，以帮助自己盈利！

### Google的免费DNS域名解析系统

Google于09年底推出了一个名为“Google Public DNS”的域名解析系统，Google表示推出免费DNS的主要目的就是为了
改进网络浏览速度，为此Google对DNS服务器技术进行了改进，通过采用预获取技术提升性能，同时保证DNS服务的安全性和准确性。

Google的免费DNS服务器的IP地址非常容易记忆：“8.8.8.8”和“8.8.4.4”。

对于Google免费DNS服务器，Google公司承诺绝不会重定或者过滤用户所访问的地址，而且绝无广告，
在此推荐遭受到国内ISP运营商DNS劫持之苦的朋友使用。

## Linux下修改DNS服务器地址

1. 编辑`/etc/resolv.conf`文件,将`nameserver`对应的值修改为google提供的DNS地址（或者其他的公共DNS也可以）:

        $vim /etc/resolv.conf
        
        resolv.conf文件内容如下：
        # Generated by NetworkManager
        domain Cisco
        search Cisco
        nameserver 8.8.8.8

2. 重启系统网络服务：

        service network restart

**注意：** 由于每次重新连上网络之后，`/etc/resolv.conf`中得配置就会自动配置成路由器的DNS地址。所以需要重新修改。



## 公共 DNS 服务器IP地址

这里收集了一些公共的DNS服务器IP地址，方便使用。

|名称|DNS 服务器IP 地址|
|----|---------------|
|阿里 AliDNS| 	223.5.5.5; 	223.6.6.6|
|CNNIC SDNS|	1.2.4.8; 	210.2.4.8|
|114 DNS| 	114.114.114.114; 	114.114.115.115|
oneDNS| 	112.124.47.27; 	114.215.126.16|
电信/移动/铁通| 	101.226.4.6; 	218.30.118.6|
DNS 派 联通| 	123.125.81.6; 	140.207.198.6|
Google DNS| 	8.8.8.8; 	8.8.4.4|
OpenDNS| 	208.67.222.222; 	208.67.220.220|
V2EX DNS| 	199.91.73.222; 	178.79.131.110|
OpenerDNS| 	42.120.21.30|
    
