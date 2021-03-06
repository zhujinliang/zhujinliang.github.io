---
layout: post
title: "pip使用简介"
category: "Python"
tags: [pip, Python]
---


## pip简介

`pip`的意思是破蛋而出，继承了python的思维，呵呵～python的包有以egg命名的，意思是蛇蛋，如此便自然而然的明白如此命名的原因。


`pip`用来安装和管理python包的命令，可以代替`easy_install`。


## pip用法

* 安装python包

        $ pip install simplejson  
        [... progress report ...]  
        Successfully installed simplejson  

* 升级包

        $ pip install --upgrade simplejson  
        [... progress report ...]  
        Successfully installed simplejson  

* 删除包

        $ pip uninstall simplejson  
        Uninstalling simplejson:  
          /home/me/env/lib/python2.7/site-packages/simplejson  
          /home/me/env/lib/python2.7/site-packages/simplejson-2.2.1-py2.7.egg-info  
        Proceed (y/n)? y  
          Successfully uninstalled simplejson  


* 从指定文件中安装指定的包、模块

        pip install -r requirements.txt  
          
        requirements.txt中的格式是：  
          
        django-coverage==1.2.2  
        django-permissions==1.0.3  
        django==1.3.1  
        MySQL-python==1.2.3  
        South==0.7.5  

关于pip的更多详细信息，请见[这里](http://www.pip-installer.org/en/latest/)。


