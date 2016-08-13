---
layout: post
title: "git submodule 基本操作"
category: "git"
date: 2015-06-28
---



# 介绍
有时候多个项目的代码会依赖一些公共的库，一般会把公共的库提取出来单独建一个仓库做版本控制，同时其他项目可以通过包含的方式来使用该库。

引用一段《Git权威指南》的话： 项目的版本库在某些情况虾需要引用其他版本库中的文件，例如公司积累了一套常用的函数库，被多个项目调用，显然这个函数库的代码不能直接放到某个项目的代码中，而是要独立为一个代码库，那么其他项目要调用公共函数库该如何处理呢？分别把公共函数库的文件拷贝到各自的项目中会造成冗余，丢弃了公共函数库的维护历史，这显然不是好的方法。

强大的git 提供了`submodule`的方式来解决这个问题。下面简单介绍下submodule的基本使用。



### 添加submodule子目录
在当前目录中添加submodule子目录依赖

```
git submodule add /path/to/repos/foo.git dest/dir
```

 
### 更新submodule的最新代码
 
```
git submodule init  # 在.git/config中注册submodule信息
git submodule update  # 更新子模块到最新依赖的commit点
 
或
git submodule update —-init —-recursive
```
 
 
<!-- more -->

### 修改了submodule目录提交
进入submodule目录

```
git checkout master  # 切换到submodule模块到master分支，根据情况需要
git add .
git commit -m "Commit message"
git push
```
其实进入submodule目录之后，相当于在一个独立git代码仓库中操作，所以修改和提交代码跟平常一样。


### clone带submodule的仓库
先clone外层项目仓库

```
git clone /path/to/repos/foo.git dest/dir
```
再更新submodule子仓库

```
git submodule init    # 在.git/config中注册submodule信息
git submodule update  # 更新子模块到最新依赖的commit点
```
 
### 一次性clone项目和submodules
clone项目时同时clone关联的submodules

```
git clone —recursive /path/to/repos/foo.git dest/dir
```


### 删除submodule子项目

```
rm .gitmodules # 删除.gitmodules文件中相应配置信息
git rm --cached libs/ # 将子模块所在的文件从git中删除
rm -rf libs/    # 物理删除libs
```

### 更新仓库

```
git pull origin master   # 更新最新master分支
git submodule update     # 根据外层仓库记录的提交信息，更新子目录仓库的代码
```


### 说明
外层仓库的commit点会记录submodule子仓库的commit点，即子仓库的commit id，而不是具体改动信息。所以，当每次外层仓库拉取最新代码之后，需要执行操作`git submodule update` 更新子仓库代码之后，才是最新代码。


 
 
 
 
 
 