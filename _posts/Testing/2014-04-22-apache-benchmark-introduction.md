---
layout: post
title: "Apache Benchmark简介--WEB压力测试工具"
description: "Apache Benchmark，简称`ab` 是 Apache 附带的一个小工具 ， 专门用于 HTTP Server 的 benchmark testing ， 可以同时模拟多个并发请求。"
category: "Testing"
tags: [Testing, WEB压力测试工具, Apache Benchmark]
---
{% include JB/setup %}

## Apache Benchmark介绍

Apache Benchmark，简称`ab` 是 Apache 附带的一个小工具 ， 专门用于 HTTP Server 的 benchmark testing ， 可以同时模拟多个并发请求。

## 安装
安装apache之后，会自带ab。
Fedora, RHEL, Centos环境下安装：

    yum install httpd

安装成功之后，系统中就会带有ab命令，可以直接调用。

## 使用
### 使用简介
输入以下命令：

    ab www.baidu.com/

测试结果如下：


	This is ApacheBench, Version 2.3 <$Revision: 1554214 $>
	Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
	Licensed to The Apache Software Foundation, http://www.apache.org/

	Benchmarking www.baidu.com (be patient).....done


	Server Software:        BWS/1.1
	Server Hostname:        www.baidu.com
	Server Port:            80

	Document Path:          /
	Document Length:        50015 bytes

	Concurrency Level:      1 
	Time taken for tests:   0.072 seconds                /* 整个测试持续的时间 * /
	Complete requests:      1                            /* 完成的请求数量 */ 
	Failed requests:        0                            /* 失败的请求数量 */ 
	Total transferred:      50655 bytes                  /* 整个场景中的网络传输量 */ 
	HTML transferred:       50015 bytes                  /* 整个场景中的 HTML 内容传输量 */ 
	Requests per second:    13.84 [#/sec] (mean)         /* 指标1，相当于 LR 中的 每秒事务数 ， mean 表示这是一个平均值 */
	Time per request:       72.251 [ms] (mean)           /* 指标2，相当于 LR 中的 平均事务响应时间 */ 
	Time per request:       72.251 [ms] (mean, across all concurrent requests)
	Transfer rate:          684.67 [Kbytes/sec] receivde    /* 平均每秒网络上的流量，可以帮助排除是否存在网络流量过大导致响应时间延长的问题 */ 

	Connection Times (ms)       /* 网络上消耗的时间的分解 */ 
	              min  mean[+/-sd] median   max
	Connect:       23   23   0.0     23      23
	Processing:    50   50   0.0     50      50
	Waiting:       24   24   0.0     24      24
	Total:         72   72   0.0     72      72


上边的数据中，`HTML transferred`，`Requests per second`，`Time per request`是我们需要重点关注的，根据这些数据，我们能大概了解Web服务器的性能水平。

结果中相关字段说明：

	Server Software       服务器系统
	Server Hostname       服务器域名
	Server Port	       服务器端口
	Document Path	     访问的路径
	Document Length	   访问的文件大小
	Concurrency Level	 并发请求数，可以理解为同一时间的访问人数
	Time taken fortests   响应时间
	Complete requests	 总共响应次数
	Failed requests	   失败的请求次数
	Write errors	      失败的写入次数
	Total transferred	 传输的总数据量
	HTML transferred	  HTML页面大小
	Requests per second   每秒支持多少人访问
	Time per request	  满足一个请求花费的总时间
	Time per request	  满足所有并发请求中的一个请求花费的总时间
	Transfer rate	     平均每秒收到的字节

最后的数据包括`Connect`,`Processing`,`Waiting`,`Total`字段。这些数据能大致说明测试过程中所需要的时间。其实我们可以只看Total字段中的min，max两列数据值，这两个值分别显示了测试过程中，花费时间最短和最长的时间。

### 使用进阶

`ab`命令常用的参数如下：


	参数     功能解释
	-n     设置ab命令模拟请求的总次数
	-c     设置ab命令模拟请求的并发数
	-t	 设置ab命令模拟请求的时间
	-k	 设置ab命令允许1个http会话响应多个请求
	

这三个参数的用法如下：

* 使用ab命令加上“`-n`”参数模拟1个用户访问百度总共5次

    	ab -n 5 www.baidu.com/

* 使用ab命令加上“`-n`”与“`-c`”参数模拟5个用户同时访问百度总共10次(即每个用户访问2次)

    	ab -c 5 -n 10 www.baidu.com/

* 使用ab命令加上“`-c`”与“`-t`”参数模拟5个用户同时访问百度总共10秒

    	ab -c 5 -t 10 www.baidu.com/

* 使用ab命令加上“`-c`”与“`-t`”附加“`-k`”参数模拟5个用户同时访问百度总共10秒,百度会打开5个并发连接，从而减少web服务器创建新链接所花费的时间。

    ab -c 5 -t 10 -k www.baidu.com/

#### 需要注意的问题

* `ab`命令必须指定**要访问的文件**，如果没指定，那必须得在域名的结尾加上一个反斜杠，例如
`ab www.baidu.com`得改写为`ab www.baidu.com/`

* `ab`命令可能会由于目标web服务器做了相应的过滤处理，导致在某些情况下收不到任何数据，这个时候可以使用“`-H`”参数，来模拟成浏览器发送请求。例如：模拟成Chrome浏览器向百度发送1个请求:

    	ab -H "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-Us) AppleWeb Kit/534.2 (KHTML, like Gecko) Chrome/6.0.447.0 Safari/534.2" www.baidu.com/

* 默认情况下，`ab`没有启用gzip压缩功能，所以压力测试的结果会跟实际情况有很大的偏差。要想让`ab`使用gzip压缩功能，得添加参数 `-H 'Accept-Encoding: gzip'`
* 带参数的压力测试示例

    	ab 'www.xxx.com/?a=1&b=2&c=3'

### 高级使用

`ab`更多的可选参数如下：


	参数    功能解释
	-A    采用base64编码向服务器提供身份验证信息，用法: -A 用户名:密码
	-C	cookie信息，用法: -C key=value
	-d	不显示pecentiles served table
	-e	保存基准测试结果为csv格式的文件
	-g	保存基准测试结果为gunplot或TSV格式的文件
	-h	显示ab可选参数列表
	-H	采用字段值的方式发送头信息和请求
	-i	发送HEAD请求，默认发送GET请求
	-p	通过POST发送数据，用法： -p page=1&key=value
	-P	采用base64编码向服务器提供身份验证信息，用法: -A 用户名:密码
	-q	执行多余100个请求时隐藏掉进度输出
	-s	使用Https协议发送请求，默认使用Http
	-S	隐藏中位数和标准偏差值
	-v	-v 2 及以上将打印警告和信息，-v 3 打印http响应码，-v 4 打印头信息
	-V	显示ab工具的版本号
	-w	采用HTML表格打印结果
	-x	HTML标签属性，使用 -w 参数时，将放置在<table>标签中
	-X	设置代理服务器，用法 -X 192.168.1.1:80
	-y	HTML标签属性，使用 -w 参数时，将放置在<tr>标签中
	-z	HTML标签属性，使用 -w 参数时，将放置在<td>标签中


