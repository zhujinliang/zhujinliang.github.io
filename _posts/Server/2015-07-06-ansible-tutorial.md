---
layout: post
title: "Ansible 简明教程"
category: "Server"
date: 2015-07-06
---


### Ansile简明教程

`ansible`是新出现的运维工具是基于Python研发的糅合了众多老牌运维工具的优点实现了批量操作系统配置、批量程序的部署、批量运行命令等功能。

一般指定远程主机列表的inventory文件格式，遵循INI文件风格中括号中的字符为组名。例如：

```
[webservers]  
10.10.2.1:2222  
10.10.2.2 
[dbservers]  
10.10.1.1
10.10.1.2
10.10.1.3
   
如果主机名称遵循相似的命名模式还可以使用列表的方式标识各主机例如  
[webservers]  
www[01:50].example.com  
[dbservers]  
db-[a:f].example.com  
   
inventory参数  
   
ansible基于ssh连接inventory中指定的远程主机时还可以通过参数指定其交互方式常用的参数如下所示  
ansible_ssh_host # 要连接的主机名  
ansible_ssh_port # 端口号默认是22  
ansible_ssh_user # ssh连接时默认使用的用户名  
ansible_ssh_pass # ssh连接时的密码  
ansible_sudo_pass # 使用sudo连接用户是的密码  
ansible_ssh_private_key_file # 秘钥文件如果不想使用ssh-agent管理时可以使用此选项  
ansible_shell_type # shell的类型默认sh  
```


#### 并行性和 shell 命令

命令格式：

```
ansible -i <inventory_file> <pattern_goes_here> -m <module_name> -a <arguments> -f <concurrency> -u <user> --sudo
（--ask-sudo-pass (-K) 如果有 sudo 密码请使用此参数）
-i 指定inventory文件
```

默认情况下，ansible 使用的 module 是 command，这个模块并不支持 shell 变量和管道等，若想使用
shell 来执行模块，请使用-m 参数指定 shell 模块

```
ansible webservers -m shell -a 'date'
```

安装python 安装包pykafka：

```
ansible -i ansible_host webservers -m command -a "/home/app/.venvs/api/bin/pip install pykafka==2.0.3" -u app
```

<!-- more -->

#### 文件传输（copy和file模块）

拷贝本地的/etc/hosts 文件到 webservers 主机组所有主机的/tmp/hosts（空目录除外）

```
ansible webservers -m copy -a "src=/etc/hosts dest=/tmp/" -u app --sudo
```

file 模块允许更改文件的用户及权限

```
ansible webservers -m file -a "dest=/srv/foo/b.txt mode=600 owner=app group=app"
```

使用 file 模块创建目录，类似 mkdir -p

```
ansible webservers -m file -a "dest=/data/log/supervisor mode=755 owner=app group=app state=directory"
```

使用 file 模块删除文件或者目录

```
ansible webservers -m file -a "dest=/path/to/c state=absent"
```



### 实际应用
#### 安装依赖包

给所有webservers 组的服务器，安装python-dateutil 依赖包，采用app用户，并发10。

```
ansible -i ansible_host webservers -m shell -a "/home/app/.venvs/api/bin/pip2.7 install python-dateutil==2.4.2" -u app  -f 10

```



#### 杀掉僵尸进程

查找webservers组的服务器，uwsgi进程的父进程进程号为1的进程，kill掉。

```
ansible -i ansible_host webservers -m shell -a "ps -ef | grep uwsgi | awk '\$3==1{print \$2}' | xargs kill -9"
```

#### 更新配置信息

更新uwsgi配置

```
ansible -i ansible_host webservers -m copy -a "src=/home/app/config/customerapi.ini dest=/etc/supervisor/conf.d/" -u app --sudo
```

检查配置信息是否都已改

```
ansible -i ansible_host webservers -m shell -a "cat /etc/supervisor/conf.d/customerapi.ini | grep 'lazy-apps' " -u app -f 5
```


重启配置信息生效

```
ansible -i ansible_host webservers -m shell -a "supervisorctl -c /etc/supervisor/supervisord.conf update" -u app --sudo -f 5

```

更多Ansible信息，请见[这里](http://docs.ansible.com/ansible/index.html)