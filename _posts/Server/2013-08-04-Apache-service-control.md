---
layout: post
title: "Apache服务控制"
category: "Server"
tags: [Apache, Server]
---



## 停止和重启：
* 方法1：

    杀死httpd进程，只需要kill`父进程`即可。不必对父进程以外的任何进程发送信号。
可以用下面这样的命令来向父进程发送信号：

        kill -TERM `cat /www/wdlinux/apache/logs/httpd.pid`

    当你向httpd发送信号后，你可以这样来读取它的进行过程：

        tail -f /www/wdlinux/apache/logs/error_log


* 方法2：

    使用下面将要描述的httpd二进制可执行文件的 -k 命令行选项：
`stop`、`restart`、`graceful`、`graceful-stop` 。不过推荐使用apachectl控制脚本来向httpd二进制可执行文件传递这些选项。

<!-- more -->
### 立即停止

* 信号：`TERM`

        apachectl -k stop

    发送TERM或stop信号到父进程可以使它**立刻杀死所有子进程**。
这将花费一些时间来杀死所有子进程。然后父进程自己也退出。所有进行中的请求将被强行中止，而且不再接受其它请求。

### 优雅重启
* 信号：`USR1`

        apachectl -k graceful

    USR1或graceful信号使得父进程建议子进程在**完成它们现在的请求后退出**(如果他们没有进行服务，将会立刻退出)。
父进程重新读入配置文件并重新打开日志文件。
每当一个子进程死掉，父进程立刻用新的配置文件产生一个新的子进程并立刻开始伺服新的请求。

### 立即重启

* 信号：`HUP`

        apachectl -k restart

    向父进程发送HUP或restart信号会使它象收到TERM信号一样**杀掉所有的子进程**，不同之处在于**父进程本身并不退出**。
它重新读入配置文件、重新打开日志文件。然后产生一系列新的子进程来继续服务。

### 优雅停止

* 信号：`WINCH`

        apachectl -k graceful-stop

    WINCH或graceful-stop信号使得父进程建议子进程在**完成它们现在的请求后退出**(如果他们没有进行服务，将会立刻退出)。
然后父进程删除PidFile并停止在所有端口上的监听。
父进程仍然继续运行并监视正在处理请求的子进程，一旦所有子进程完成任务并退出或者超过由`GracefulShutdownTimeout`指令规定的时间，父进程将会退出。
在超时的情况下，所有子进程都将接收到TERM信号并被强制退出。
