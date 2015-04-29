---
layout: post
title: "配置开发环境镜像源"
category: "Configuration"
---

# 配置开发环境镜像源

国内配置开发环境的时候经常会要用到的这几个源, 默认源因为一些众所周知的原因非常慢或者根本上不了. 
还好国内都有镜像, 收集如下。

## 系统

* [阿里云开源镜像站][aliyun]

* [网易开源镜像站][wangyi]


## Python PyPI (pip)

* 编辑`~/.pip/pip.conf`:

        vim ~/.pip/pip.conf

* 添加**[豆瓣python源][douban_python]**

        [global]
        index-url = http://pypi.douban.com/simple

<!-- more -->

## Node NPM

* 编辑 `~/.npmrc`

        vim ~/.npmrc

* 添加[淘宝npm源][taobao_npm]

        registry = https://npm.taobao.org


## Ruby Gem

* 删除rubygems源，添加[淘宝ruby源][taobao_ruby]

        $ gem sources --remove https://rubygems.org/
        $ gem sources -a http://ruby.taobao.org/


[aliyun]: http://mirrors.aliyun.com
[wangyi]: http://mirrors.163.com
[douban_python]: http://pypi.douban.com
[taobao_ruby]: http://ruby.taobao.org
[taobao_npm]: https://npm.taobao.org