---
layout: post
title: "AJAX 状态值(readyState)和状态码(status)"
category: "JavaScript"
date: 2014-03-06
---


## AJAX状态值与状态码区别

**AJAX状态值**是指，运行AJAX所经历过的几种状态，无论访问是否成功都将响应的步骤，可以理解成为AJAX运行步骤。如：正在发送，正在响应等，由AJAX对象与服务器交互时所得；使用“ajax.readyState”获得。（由数字1~4单位数字组成）

**AJAX状态码**是指，无论AJAX访问是否成功，由HTTP协议根据所提交的信息，服务器所返回的HTTP头信息代码，该信息使用“ajax.status”所获得；（由数字1XX,2XX三位数字组成，详细查看RFC）
这就是我们在使用AJAX时为什么采用下面的方式判断所获得的信息是否正确的原因。

	if (ajax.readyState == 4 && ajax.status == 200) {
		putData(ajax.responseText);
	}

## AJAX运行步骤与状态值说明

在AJAX实际运行当中，对于访问XMLHttpRequest（XHR）时并不是一次完成的，而是分别经历了多种状态后取得的结果，对于这种状态在AJAX中共有5种，分别是：

* 0 - (未初始化)还没有调用send()方法
* 1 - (载入)已调用send()方法，正在发送请求
* 2 - (载入完成)send()方法执行完成，
* 3 - (交互)正在解析响应内容
* 4 - (完成)响应内容解析完成，可以在客户端调用了

对于上面的状态，其中“0”状态是在定义后自动具有的状态值，而对于成功访问的状态（得到信息）我们大多数采用“4”进行判断。


## AJAX状态码说明
也就是HTTP 状态码，这个应该大家都明白的，这里就简单提一下。

* 1XX：请求收到，继续处理
* 2XX：操作成功收到，分析、接受
* 3XX：完成此请求必须进一步处理
* 4XX：请求包含一个错误语法或不能完成
* 5XX：服务器执行一个完全有效请求失败


## AJAX运行步骤示义图
![Ajax status code](http://zhujinliang.qiniudn.com/img/blog/ajax_status_code.jpg)