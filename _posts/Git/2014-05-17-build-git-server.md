---
layout: post
title: "搭建git服务器"
date: 2014-05-17
tag: "git"
---


由于git是分布式版本的，每个人本地都有一份完整的代码，所以其远程仓库实际上和本地仓库没什么不同，远程仓库只是作为一个中央服务器，为了方便交换大家的修改。

GitHub就是一个免费托管开源代码的远程仓库，同时他也提供了付费的私有代码仓库。很多时候，公司或团队不想公开源代码，可以采用github来托管代码。后来也有出来bitbucket，对于私有的仓库也是免费的，但是有团队人数限制，超过之后也要收费。国内也有团队推出了免费的类似的代码托管和分享平台，如：开源中国的[git@osc][git oschina]。但如果出于某些原因，例如还是不放心，希望没有网络情况下，局域网也能访问代码仓库时，那就可以自己搭建一台Git服务器作为私有仓库使用。

接下来介绍如何搭建git服务器。

<!-- more -->

### 搭建git服务器

首先需要准备一台运行Linux的机器，这里以CentOS系列系统为例，其他Ubuntu等也类似，只是安装git的命令不同而已。

假设你已经有sudo权限的用户账号，下面，正式开始安装。

第一步，安装git：

	$ sudo yum install git

第二步，创建一个git用户，用来运行git服务：

	$ sudo useradd git

第三步，创建证书登录：

收集所有需要登录的用户的公钥，就是他们自己的id_rsa.pub文件，把所有公钥导入到/home/git/.ssh/authorized_keys文件里，一行一个。
这样，每次提交的时候，就可以免密码提交了。

其实是和github上，要你输入公有密钥是一样的。

第四步，初始化Git仓库：

先选定一个目录作为Git仓库，假定是sample.git，在/home/git目录下输入命令：

	$ sudo git init --bare sample.git

Git就会创建一个裸仓库，裸仓库没有工作区。

因为服务器上的Git仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的Git仓库通常都以.git结尾。然后，把owner改为git：

	$ sudo chown -R git:git sample.git

第五步，禁用shell登录：

出于安全考虑，第二步创建的git用户不允许登录shell，这可以通过编辑/etc/passwd文件完成。找到类似下面的一行：

	git:x:1001:1001:,,,:/home/git:/bin/bash

改为：

	git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell

这样，git用户可以正常通过ssh使用git，但无法登录shell，因为我们为git用户指定的git-shell每次一登录就自动退出。

第六步，克隆远程仓库：

现在，可以通过`git clone`命令克隆远程仓库了，在各自的电脑上运行：

	$ git clone git@server:/home/git/sample.git
	Cloning into 'sample'...
	warning: You appear to have cloned an empty repository.

这里的`server`可以是域名，也可以是IP地址，即对应的git服务器的地址。
剩下的推送就简单了。


### 将已有代码提交到git服务器

若是自己已经有代码仓库，现在要转移，添加自建的git服务器作为另一个代码仓库则只要在本地的git配置中，添加一个`remote`仓库地址即可。

	$ vim .git/config

	[remote "origin"]
	url = git@github.com:sample/sample.git
	fetch = +refs/heads/*:refs/remotes/origin/*
	# 添加如下代码
	[remote "local"]
	url = git@192.168.5.109:/home/git/sample.git
	fetch = +refs/heads/*:refs/remotes/local/*

在下面添加一个名为`local`的远程仓库，以及对应的地址。接下来在提交的时候，指定仓库提交即可。

	$ git push local master

将本地master分支提交到远程仓库local中。


### 管理公钥

如果团队很小，把每个人的公钥收集起来放到服务器的`/home/git/.ssh/authorized_keys`文件里就是可行的。如果团队有几百号人，就没法这么玩了，这时，可以用[Gitosis][gitosis]来管理公钥。


### 其他

git 支持hook，所以可以在服务器端针对不同的操作，编写一系列的脚本来监控提交的情况，从而做出相应的动作。
结合这个特点，可以做自动化部署。例如，每次提交之后，都自动跑一遍测试脚本。





[git oschina]: http://git.oschina.net/
[gitosis]: https://github.com/res0nat0r/gitosis