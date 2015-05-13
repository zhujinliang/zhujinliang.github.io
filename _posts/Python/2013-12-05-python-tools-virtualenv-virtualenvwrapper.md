---
layout: post
title: "python 工具 virtualenv和virtualenvwrapper"
category: "Python"
date: 2013-12-05
---

工欲善其事，必先利其器。Python开发中，环境的隔离非常重要，每个项目中所依赖的包的数量以及相同包版本不同是很常见的，若在同一个python环境中，则会出现各种冲突。

介绍两个python开发工具，virtualenv和virtualenvwrapper，应该是每个python开发者必备的。

## virtualenv简介

[`virtualenv`][virtualenv home]是一个隔离python环境的工具。

### virtualenv安装
利用yum进行安装：

    yum install python-virtualenv

或利用`pip`进行安装：

    pip install virtualenv

### virtualenv的用法

#### virtualenv基本用法

* 创建虚拟环境的目录`DEST_DIR`,指定python版本

        virtualenv DEST_DIR --python=<python2.7>

* 切换到虚拟环境

        source DEST_DIR/bin/activate

* 通过`pip`或`easy_install`安装的软件包都会安装到新建的虚拟的环境中
* 到自己项目中，开发即可
* 退出虚拟环境

        deactivate

#### virtualenv进阶

* 创建虚拟环境时，指定Python版本（当系统中有多个Python版本时非常有用），默认使用的是`/usr/bin/python`

        virtualenv DEST_DIR --python=<python2.7>
        或
        virtualenv DEST_DIR -p <python2.7>

* 创建一个纯净的虚拟环境,虚拟环境中没有访问系统环境中包的权限。

        virtualenv --no-site-packages DEST_DIR


## virtualenvwrapper

[`virtualenvwrapper`][virtualenvwrap home]是virtualenv的扩展。

其中扩展包括：

* 创建和删除虚拟化环境;
* 管理开发工作流，使得在多个项目之间不引起冲突且工作更简单;

特征有：

* 在一个地方组织虚拟环境(~/.virtualenv)；
* 为创建、拷贝及删除环境进行了封装(命令分别为`mkvirtualenv`, `cpvirtualenv`,`rmvirtualenv`)且包括用户配置的hooks;
* 一个单一命令在环境之间切换(`workon ENV`) ;
* 为所有的命令提供用户定义的hooks(User-configurable hooks);
* 可扩展的插件系统

### virtualenvwrapper安装
利用yum进行安装：

    yum install python-virtualenvwrapper

或利用`pip`进行安装：

    pip install virtualenvwrapper

### virtualenvwrap的基本用法

(具体命令列表见[这里][virtualenvwrap command])

* 设置python版本和工作目录

        export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python # (默认使用的python版本)
        export WORKON_HOME=~/.venvs # (工作目录，如果没有默认为~/.virutalenvs)

* 创建virtualenvwrap的虚拟工作环境

        source /usr/local/bin/virtualenvwrapper.sh

* (可选)创建PIP下载软件包的缓存位置

        export PIP_DOWNLOAD_CACHE=$HOME/.pip-downloads

* 为项目PROJECT设置虚拟环境

        mkvirtualenv  PROJECT
        workon  PROJECT

* 退出虚拟环境，输入命令：

        deactivate

### virtualenvwrapper进阶
具体的命令如下：

**管理虚拟环境**

* `mkvirtualenv`
    在`WORKON_HOME`中创建一个虚拟环境。除了-a, -i, -r, 和 -h参数之外的所有参数，都是直接传给virtualenv
的。环境创建成功之后，会自动激活。

        mkvirtualenv [-a project_path] [-i package] [-r requirements_file] [virtualenv options] ENVNAME

    `-r` 参数指定从一个文件进行安装包，类似`pip install -r requirements_file`

    `-a` 参数把新创建的环境关联一个已经存在的项目路径

    `-i` 参数 在环境创建完成之后，安装一个或多个包

* `cpvirtualenv`
    复制一个虚拟环境

        cpvirtualenv ENVNAME [TARGETENVNAME]

* `rmvirtualenv`
    在`WORKON_HOME`目录中删除环境(在删除当前环境之前要先`deactive`)

        rmvirtualenv ENVNAME

* `lsvirtualenv`
    列出所有的环境

* `allvirtualenv`
    在`WORKON_HOME`目录下的所有虚拟环境都执行一个命令

        allvirtualenv [command with arguments]

        $ allvirtualenv pip install -U pip

**控制虚拟环境**

* `workon`
    列出虚拟环境或者切换虚拟环境

        workon [environment_name]

* `deactivate`
    退出虚拟环境到系统环境下

        deactivate

**快速定位到虚拟环境**

* `cdvirtualenv`
    切换当前工作目录到`$VIRTUAL_ENV`

        cdvirtualenv [subdir]

        $ mkvirtualenv env1
        New python executable in env1/bin/python
        Installing distribute.............................................
        ..................................................................
        ..................................................................
        done.
        (env1)$ echo $VIRTUAL_ENV
        /Users/dhellmann/Devel/virtualenvwrapper/tmp/env1
        (env1)$ cdvirtualenv
        (env1)$ pwd
        /Users/dhellmann/Devel/virtualenvwrapper/tmp/env1
        (env1)$ cdvirtualenv bin
        (env1)$ pwd
        /Users/dhellmann/Devel/virtualenvwrapper/tmp/env1/bin

先到这，常见的情况都能应对自如了。



[virtualenv home]: www.virtualenv.org
[virtualenvwrap home]: http://virtualenvwrapper.readthedocs.org/en/latest/
[virtualenvwrap command]:  http://virtualenvwrapper.readthedocs.org/en/latest/command_ref.html 