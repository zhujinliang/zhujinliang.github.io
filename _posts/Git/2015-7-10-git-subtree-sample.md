---
layout: post
title: "git subtree 基本操作"
category: "git"
date: 2015-07-10
---


# 介绍

有时候多个项目的代码会依赖一些公共的库，一般会把公共的库提取出来单独建一个仓库做版本控制，而对于这些公共的库，我们可能会有这些需求：

* 如何在其他git项目中以子目录的形式导入这些公共的git仓库？
* 如何在其他项目中修改，然后推送回被导入git仓库？
* 如何获取被引用库的最近更新？

前面一篇[git submodule 基本操作](http://zhujinliang.cn/blog/git/2015/06/28/git-submodule-sample.html)的文章，我们介绍了通过`submodule`的方式，来解决这些问题，用过的但不够熟悉的同学可能会发现submodule不好用，如果拉取最新代码之后，忘记执行`git submodule update`更新子目录仓库代码，会导致子目录代码未更新成最新，若直接在上面修改代码，再提交，则会出现问题。由于一些同学对git submodule不熟悉，我所在的公司之前就出现过一些代码合并错误导致的事故。

强大的git提供了另一种`subtree`的方式来解决这些问题，下面简单介绍下基本使用。

<!-- more -->

## 基本使用

### 背景

* project1 是主项目1
* project2 是主项目2
* core是共用模块，是project1和project2都需要依赖的模块目录。

 
我们需要在project1和project2中关联core。

### 1 添加子目录，建立与core项目的关联。
进入到项目仓库目录

```
git remote add -f core git@github.com:zhujinliang/core.git 
git remote -v #查看remote中会增加一个core的远程仓库
git subtree add --prefix=core core master --squash
```
 
语法：

```
git remote add -f <子仓库名> <子仓库地址>
git subtree add --prefix=<子目录名> <子仓库名> <分支> --squash
 
-f 意思是在添加远程仓库之后，立即执行fetch。
--squash意思是把subtree的改动合并成一次commit，这样就不用拉取子项目完整的历史记录。
--prefix之后的=等号也可以用空格，或者-P 。
```
 
### 2 从远程仓库更新子目录core

```
git fetch core master 
git subtree pull --prefix=core core master --squash
```

语法：

```
git fetch <远程仓库名> <分支>
git subtree pull --prefix=<子目录名> <远程分支> <分支> --squash
```
 
 
### 3 从子目录push到远程仓库
 
```
git subtree push —prefix=core core master
```

语法：

```
git subtree push --prefix=<子目录名> <远程分支名> 分支
```


 
## 说明
这里需要注意下`subtree`和`submodule`的方式的区别：

* `subtree`方式，外层仓库的commit店会记录subtree自仓库的改动信息。所以，当外层仓库`git pull`拉取最新代码后，得到的即是最新的代码。
* `submodule` 方式，外层仓库的commit点会记录submodule子仓库的commit点，即子仓库的commit id，而不是具体改动信息。所以，当每次外层仓库`git pull`拉取最新代码之后，需要执行操作git submodule update 更新子仓库代码之后，才是最新代码。


 
 
 