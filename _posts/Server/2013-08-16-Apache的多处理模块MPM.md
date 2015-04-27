---
layout: post
title: "Apache的多处理模块MPM"
description: "MPM，是Multi -Processing Modules的缩写，就是多处理模块的意思，它是在Apache 2.0中引入的一个概念，其引入目标是将Apache的结构能够模块化，把核心的任务处理作为一个可插拔的模块来运行，这样就很容易根据不同的环境和应用来更有效的优化Apache的运行。它的功能是负责绑定本机网络端口、接受请求，并调度子进程来处理请求。"
category: "Apache"
tags: [MPM, Apache]
---



最近，在开发的网站的服务器由于大量搜索引擎的爬虫访的光顾，某一时段，当请求过多是，Apache会产生大量的httpd进程，多时高达100多个httpd进程，导致服务器的负载直接飙升至100多，内存几乎耗尽，网站瘫痪，服务器也瘫痪，有时连ssh也基本登不上去，只能通过联系服务器商来重启服务器解决。这个问题十分的困扰我们团队，于是，便在查找了关于Apache优化的一些资料，了解到可以在MPM模块上做文章，收集整理了一些关于MPM的资料，以便以后查阅。

## MPM模块简介
　　MPM，是Multi -Processing Modules的缩写，就是多处理模块的意思，它是在Apache 2.0中引入的一个概念，其引入目标是将Apache的结构能够模块化，把核心的任务处理作为一个可插拔的模块来运行，这样就很容易根据不同的环境和应用来更有效的优化Apache的运行。它的功能是负责绑定本机网络端口、接受请求，并调度子进程来处理请求。

　　它是Apache2.X中最影响其性能的，最核心的特性，因为直接决定了Apache的工作方式。所以通过设置MPM模块，可以优化Apache的性能。关于Apache的MPM模块的详细文档，请见[这里](http://www.361way.com/mpm/1052.html)。

## MPM模块类型
　　在安装apache时，我们可以查看哪些MPM模块可以被使用，查看方式如下：

    [root@localhost httpd-2.2.11]# ./configure --help | grep mpm  
      --with-mpm=MPM          Choose the process model for Apache to use.  
                          MPM={beos|event|worker|prefork|mpmt_os2}  

* `beos` 专门针对BeOS优化过的多路处理模块(MPM)

* `event` 一个标准workerMPM的实验性变种

* `worker` 线程型的MPM，实现了一个混合的多线程多处理MPM，允许一个子进程中包含多个线程。

* `prefork` 一个非线程型的、预派生的MPM

* `mpmt_os2` 专门针对OS/2优化过的混合多进程多线程多路处理模块(MPM)

查看当前Apache是编译了哪个类型的MPM模块，找到Apache安装文件所在目录，查看方式如下：

    [root@localhost httpd-2.2.11]# /xxx/httpd-2.2.22/bin/httpd -l
    Compiled in modules:
    core.c
    prefork.c
    http_core.c
    mod_so.c
    ...

　　从显示结果中看出Apache当前的正在工作的MPM是prefork工作方式的MPM，这也是一种缺省的模块，如果需要其它的MPM模块，在编译的时候用--with-mpm指定。

　　日常使用过程中，我们常用的MPM模块的类型为`prefork`和`worker`方式，接下来将具体介绍这两种方式，以及相关的配置。

## prefork MPM
### 概述
　　一个非线程型的、预派生的MPM 。这个多路处理模块(MPM)实现了一个非线程型的、预派生的web服务器，它的工作方式类似于Apache 1.3。它适合于没有线程安全库，需要避免线程兼容性问题的系统。它是要求将每个请求相互独立的情况下最好的MPM，这样若一个请求出现问题就不会影响到其他请求。

　　这个MPM具有很强的自我调节能力，只需要很少的配置调整。最重要的是将MaxClients设置为一个足够大的数值以处理潜在的请求高峰，同时又不能太大，以致需要使用的内存超出物理内存的大小。

### 工作方式
　　由一个单独的控制进程(父进程)产生子进程，然后将这些子进程用于监听请求并作出应答，这些子进程在处理时并不产生线程。另外，Apache总是试图保持一些备用的(spare)或者是空闲的子进程用于迎接即将到来的请求。这样客户端就不需要在得到服务前等候子进程的产生。

### 配置参数
用于配置prefork MPM如何工作的主要参数有：

* StartServers

StartServers设置服务器**启动时**建立的子进程数量。因为子进程数量动态的取决于负载的轻重，所有一般没有必要调整这个参数。不同的MPM默认值也不一样。对于prefork默认值是"5"。

* MinSpareServers

MinSpareServers设置**空闲子进程**的最小数量。

所谓空闲子进程是指没有正在处理请求的子进程。如果当前空闲子进程数少于MinSpareServers ，那么Apache将以最大每秒一个的速度产生新的子进程。只有在非常繁忙机器上才需要调整这个参数。将此参数设的太大通常是一个坏主意。

* MaxSpareServers

MaxSpareServers设置**空闲子进程**的最大数量。

如果当前有超过MaxSpareServers数量的空闲子进程，那么父进程将杀死多余的子进程。只有在非常繁忙机器上才需要调整这个参数。将此参数设的太大通常是一个坏主意。

如果你将该的值设置为比MinSpareServers小，Apache将会自动将其修改成"MinSpareServers+1"。

* MaxClients

MaxClients设置允许同时伺服的最大接入请求数量。任何超过MaxClients限制的请求都将进入等候队列，直到达到ListenBacklog限制的最大值为止。一旦一个链接被释放，队列中的请求将得到服务。

对于非线程型的MPM(也就是prefork)，MaxClients表示可以用于伺服客户端请求的最大子进程数量，默认值是256。若要设置大于这个值，你必须同时增大ServerLimit 。

对于线程型或者混合型的MPM(也就是beos或worker)，MaxClients表示可以用于伺服客户端请求的最大线程数量。线程型的beos的默认值是50。对于混合型的MPM默认值是16(ServerLimit)乘以25(ThreadsPerChild)的结果。因此要将MaxClients增加到超过16个进程才能提供的时候，你必须同时增加ServerLimit的值。

* MaxRequestsPerChild

MaxRequestsPerChild设置每个子进程在其生存期内允许伺服的最大请求数量，默认为10000。到达MaxRequestsPerChild的限制后，子进程将会结束。如果 MaxRequestsPerChild为0，子进程将永远不会结束。

将MaxRequestsPerChild设置成非零值有两个好处：

1） 可以防止(偶然的)内存泄漏无限进行，从而耗尽内存。

2） 给进程一个有限寿命，从而有助于当服务器负载减轻的时候减少活动进程的数量。

* ServerLimit

ServerLimit设置了MaxClients最大允许配置的数值。


配置举例（使用时去掉后面的中文注释）：

    <IfModule mpm_prefork_module>
      StartServers        5 Apache启动时开启的子进程个数
      MinSpareServers     5 最小的空闲子进程
      MaxSpareServers     10 最大的空闲子进程
      MaxClients          40 最大进程数量，以免产生大量的进程占用内存，影响系统性能
      MaxRequestsPerChild  500 子进程在处理了500个请求之后，就会被销毁，然后创建新进程
    </IfModule>

## worker MPM
### 概述

　　这个多路处理模块(MPM)使网络服务器支持混合的多线程多进程。由于使用线程来处理请求，所以可以处理海量请求，而系统资源的开销小于基于进程的MPM。但是，它也使用了多进程，每个进程又有多个线程，以获得基于进程的MPM的稳定性。

　　控制这个MPM的最重要的配置是，控制每个子进程允许建立的线程数的`ThreadsPerChild`，和控制允许建立的总线程数的`MaxClients`。

### 工作方式

　　每个进程可以拥有的线程数量是固定的。服务器会根据负载情况增加或减少进程数量。一个单独的控制进程(父进程)负责子进程的建立。每个子进程可以建立ThreadsPerChild数量的服务线程和一个监听线程，该监听线程监听接入请求并将其传递给服务线程处理和应答。

　　Apache总是试图维持一个备用(spare)或是空闲的服务线程池。这样，客户端无须等待新线程或新进程的建立即可得到处理。初始化时建立的进程数量由StartServers决定。随后父进程检测所有子进程中空闲线程的总数，并新建或结束子进程使空闲线程的总数维持在MinSpareThreads和MaxSpareThreads所指定的范围内。由于这个过程是自动调整的，几乎没有必要修改这些指令的缺省值。可以并行处理的客户端的最大数量取决于MaxClients。

### 配置参数
用于配置worker MPM如何工作的主要参数有：

* StartServers

同prefork MPM中的意思，即初始化时建立的进程数量。
对于worker默认值是"3"。

* MinSpareThreads

MinSpareThreads设置**空闲线程**的最小数量。

* MaxSpareThreads

MaxSpareThreads设置**空闲线程**的最大数量。

* MaxClients

MaxClients设置可以并行处理的客户端的最大数量。

* ServerLimit

ServerLimit设置**活动子进程**数量的上限。该设置必须出现在其他worker MPM设置的前面。

**Note:** 活动子进程的最大数量等于MaxClients除以ThreadsPerChild的值。

* ThreadLimit
ThreadLimit设置所有服务线程总数的上限，它必须大于或等于ThreadsPerChild。该设置必须出现在其他worker MPM设置的前面。

* ThreadsPerChild

ThreadsPerChild设置每个子进程最大能够产生的线程数量

　　 在设置的活动子进程数量之外，还可能有额外的子进程处于"正在中止"的状态但是其中至少有一个服务线程仍然在处理客户端请求，直到到达MaxClients以致结束进程，虽然实际数量会很小。这个行为能够通过以下禁止特别的子进程中止的方法来避免：  
1. 将MaxRequestsPerChild设为"0"。  
2. 将MaxSpareThreads和MaxClients设为相同的值。

配置举例（使用时去掉后面的中文注释）：

    <IfModule mpm_worker_module>
      StartServers          2   Apache启动时开启的子进程个数
      MaxClients          150   最大连接数目
      MinSpareThreads      25   最小空闲线程数
      MaxSpareThreads      75   最大空闲线程数
      ThreadsPerChild      25   每个进程能够创建的线程数
      MaxRequestsPerChild   0   每个子进程能够处理的最多请求数
    </IfModule>

## 小结

　　 Prefork MPM基于非线程模型，由独立的进程创建多个子进程的方式来工作，它在所有情况下都很安全，对运行非线程安全（non-thread-safe）模式的软件如PHP，它是唯一的安全选择。另一方面，prefork用单独的子进程来处理不同的请求，进程之间是彼此独立的，这也使其成为最稳定的MPM之一。但是由于每一个请求都会产生一个新的进程，导致系统资源（尤其是内存）消耗的很快，一旦并发量较大的时候，大量的Apache进程会占用巨大的内存空间。 

　　 Worker MPM基于线程模式，具有内存消耗低（对繁忙的服务很重要）、扩展性在某些特定应用情况下比Prefork更好等优点。在这个模式下，采用的进程和线程混合的形式处理请求。由于使用线程来处理，所以可以处理相对海量的请求，而系统资源的开销要小于基于进程的Prefork模式。

　　 以上两种稳定的MPM方式在非常繁忙的服务器应用下都有些不足。需要根据自己服务器的硬件资源情况来设置。我采用的是安装Apache时默认编译进去的prefork类型，通过配置了参数之后，这几天观察发现httpd的进程数量确实限制住了，至少不会导致服务器宕机，没有响应的后果。关于worker的方式，有待以后再测试一下。

