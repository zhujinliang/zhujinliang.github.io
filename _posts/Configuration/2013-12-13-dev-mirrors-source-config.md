---
layout: post
title: "配置开发环境镜像源"
description: "配置开发环境镜像源"
category: "Configuration"
tags: [Python, pip, Nodejs, Ruby]
---
{% include JB/setup %}

# 配置开发环境镜像源

国内配置开发环境的时候经常会要用到的这几个源, 默认源因为一些众所周知的原因非常慢或者根本上不了. 
还好国内都有镜像, 收集如下。

## Python PyPI (pip)

* 编辑`~/.pip/pip.conf`:

        vim ~/.pip/pip.conf

* 添加**[豆瓣源][douban]**

        [global]
        index-url = http://pypi.douban.com/simple



## Node NPM

* 编辑 `~/.npmrc`

        vim ~/.npmrc

* 添加

        registry = http://registry.cnpmjs.org


## Ruby Gem

* 删除rubygems源，添加[淘宝源][taobao]

        $ gem sources --remove https://rubygems.org/
        $ gem sources -a http://ruby.taobao.org/


[douban]: http://pypi.douban.com
[taobao]: http://ruby.taobao.org/