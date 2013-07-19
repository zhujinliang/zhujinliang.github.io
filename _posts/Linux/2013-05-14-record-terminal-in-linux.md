---
layout: post
title: "Record Terminal In Linux"
description: "Record terminal in linux"
category: "Linux"
tags: [Linux]
---
{% include JB/setup %}

在Linux环境下我们常常会碰到这样的场景，需要在终端下给别人演示某些命令的结果，或者需要演示如何在
终端下通过命令解决一个问题，我们通常是会把命令一组一组的写在PPT中，然后现场给其他人敲命令演示。
这种方法是我见到的较为常见的。有时候，演示者可能紧张，或者当时使用的PC环境没有配好，就可能会导致
演示达不到预期的效果。如果，我们能够在演示之前把所有的操作和结果都事先通过某种方法录制下来，那么
演示时只需要播放整个操作过程即可，可以避免现场可能出现的各种意外。

提到录制屏幕，很多人第一个会想到是屏幕录制软件，确实，不管是在Windows下还是Linux，都有很多屏幕
录制软件可供使用，这也不乏是一种解决办法。但是使用屏幕录制软件有一些缺点：

* 得到的是一个视频文件，若录制时间长，视频文件会很大。

* 屏幕录制软件的录制效果不佳，分辨率不能保证，若提高分辨率，则视频文件也会很大。

Linux下有很多方便好用的工具，当然也不乏屏幕录制软件，但是如果仅仅是录制终端的话，大可不必非得用
屏幕录制软件不可。下面介绍两个我自己使用过的录制终端的方法。


## asciiio

ASCII.IO is the simplest way to record your terminal and share the recordings 
with your fellow geeks. Simply record and upload your terminal session with a 
single command, and ASCII.IO will play it back in your browser. 

摘自[ascii.io网站](http://ascii.io)

* Install
{% highlight bash %}

$ curl -sL get.ascii.io | bash

{% endhighlight %}

* Config
{% highlight bash %}

$ asciiio auth

{% endhighlight %}

* How to use
{% highlight bash %}

$ asciiio   # 开始录制
输入...
 
$ exit     # 或者Ctrl + d 退出

{% endhighlight %}
退出之后，asciiio会提示，是否要上传，选择yes，上传，并提示查看的网址。点击网址，在浏览器打开
网页便可以看到刚才录制的终端了。

**Note:**上传需要网络，且只能在网站上查看录制的信息（都需要网络）。

可以在ascii.io上注册一个自己的账号，然后在本地配置账号信息之后，可以在网站上查看自己上传的录制
信息。

## script

我们也可以使用Shell 自带的命令来录制终端，主要使用的命令是script。

* Record
{% highlight bash %}

$ script -t 2> timing.log -a output.session
type commands;
...
..
$ exit

{% endhighlight %}

`timing.log`文件用于存储时序信息，描述每个命令的在何时运行。

`output.session`文件用于存储命令输出。

* Play

借助这两个命令，我们可以回放终端刚才的过程：
{% highlight bash %}

$ scriptreplay timing.log output.session

{% endhighlight %}

