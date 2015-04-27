---
layout: post
title: "Linux iptables设置教程"
description: "Netfilter/iptables（简称为`iptables`）组成Linux平台下的包过滤防火墙，与大多数的Linux软件一样，这个包过滤防火墙是免费的，它可以代替昂贵的商业防火墙解决方案，完成封包过滤、封包重定向和网络地址转换（NAT）等功能。Iptables有利于在 Linux 系统上更好地控制 IP 信息包过滤和防火墙配置。"
category: "Linux"
tags: [iptables, Linux, Server]
---




## iptables简介
Netfilter/iptables（简称为`iptables`）组成Linux平台下的包过滤防火墙，与大多数的Linux软件一样，这个包过滤防火墙是免费的，它可以代替昂贵的商业防火墙解决方案，完成封包过滤、封包重定向和网络地址转换（NAT）等功能。Iptables有利于在 Linux 系统上更好地控制 IP 信息包过滤和防火墙配置。

### iptables优点
1. 它可以配置有状态的防火墙。有状态的防火墙能够指定并记住为发送或接收信息包所建立的连接的状态。防火墙可以从信息包的连接跟踪状态获得该信 息。在决定新的信息包过滤时，防火墙所使用的这些状态信息可以增加其效率和速度。这里有四种有效状态，名称分别为 ESTABLISHED 、 INVALID 、 NEW 和 RELATED 。

2. 它使用户可以完全控制防火墙配置和信息包过滤。您可以定制自己的规则rules来满足您的特定需求，从而只允许您想要的网络流量进入系统。

### iptables服务使用及查看
* 启动iptables

        service iptables start  
* 重启iptables

        service iptables restart  
* 关闭iptables

        service iptables stop  

* 查看当前iptables状态
    
        service iptables status
 
* 配置文件位置

        /etc/sysconfig/iptables
**Note**: `service iptables`命令会自动调用`/etc/rc.d/init.d/iptables`。

## iptables命令用法
`iptables`命令语法：

    iptables [-t表名] <-A| I |D |R > 链名[规则编号] [-i | o 网卡名称] [-p 协议类型] [-s 源IP地址 | 源子网][--sport 源端口号] [-d 目标IP地址 | 目标子网][--dport 目标端口号] <-j 动作> 

一条iptables规则包含如下4个基本元素：表，命令，匹配，目标。

1) 表（table）

[-t table]选项允许使用标准表之外的任何表。表是包含仅处理特定类型信息包的规则和链的信息包过滤表。

有三种可用的表选项：`filter`、`nat`和`mangle`。该选项不是必需的，如果未指定，则**filter用作默认表**。

* `filter`表用于一般的信息包过滤，包含INPUT、OUTPUT和FORWAR链。
    * INPUT:数据包的目地的是LINUX主机本身
    * OUTPUT:数据包由LINUX主机本身发送
    * FORWARD:数据包从一个接口进入，另一个接口发出

* `nat`表用于要转发的信息包，它包含PREROUTING、OUTPUT和POSTROUTING链。
* `mangle`表,如果信息包及其头内进行了任何更改，则使用mangle表。该表包含一些规则来标记用于高级路由的信息包以及PREROUTING和OUTPUT链。

**Note：**我们借助`filter`表来禁止某些IP访问服务器。

2) 命令（command）

`command`部分是iptables命令的最重要部分，它告诉iptables命令要做什么，例如，插入规则、将规则添加到链的末尾或删除规则。主要有如下所示的命令。


    -A 或 --append：指定链名  该或命令将一条规则附加到链的末尾
    -D 或 --delete： 通过用-D指定要匹配的规则或者指定规则在链中的位置编号，该命令从链中删除该规则
    -R 或 --replace： 替换指定链中一条匹配的规则
    -P 或 --policy： 该命令设置链的默认目标，即策略。所有与链中任何规则都不匹配的信息包都将被强制使用此链的策略。
    -N 或 --new-chain： 用命令中所指定的名称创建一个新链。
    -F 或 --flush： 如果指定链名，该命令删除链中的所有规则，如果未指定链名，该命令删除所有链中的所有规则。此参数用于快速清除。
    -X或--delete-chain: 删除指定用户的的定义链，若没有指定链，则删除所有的用户链。

3) 匹配（match）

iptables命令的可选match部分指定信息包与规则匹配所应具有的特征（如源和目的地地址、协议等）。

匹配分为两大类：通用匹配和特定于协议的匹配。下面是一些重要的且常用的通用匹配及其说明（可以使用!符号表示不与该项匹配）：

    -p或--protocol：指定协议，如TCP、UDP、ICMP、用逗号分隔的任何这三种协议的组合列表以及ALL（用于所有协议）。ALL是默认匹配。  
    -s 或 --source：指定源IP地址。该匹配还允许对某一范围内的IP地址进行匹配。默认源匹配与所有IP地址匹配。
    -d 或 --destination：指定目标地址。该匹配还允许对某一范围内IP地址进行匹配。
    --sport：指定源端口或端口范围（source port 源端口）      
    --dport：指定目标端口或端口范围（destination port 目的端口）  
    
4) 目标（target）

目标是由规则指定的操作，对与那些规则匹配的信息包执行这些操作。
下面是常用的一些目标及其示例和说明:

    -j ACCETP： 当信息包与具有ACCEPT目标的规则完全匹配时，会被接受（允许它前往目的地）
    -j DROP： 当信息包与具有DROP目标的规则完全匹配时，会阻塞该信息包，并且不对它做进一步处理。
    -j REJECT： 该目标的工作方式与DROP目标相同，但它比DROP好。和DROP不同，REJECT不会在服务器和客户机上留下死套接字。另外，REJECT将错误消息发回给信息包的发送方。
    -j RETURN： 在规则中设置的RETURN目标让与该规则匹配的信息包停止遍历包含该规则的链。如果链是如INPUT之类的主链，则使用该链的默认策略处理信息包。
    LOG： 表示将包的有关信息记录入日志。

更具体信息的请详见`man iptables`。


## 修改iptables规则
添加或修改iptables规则，并生效的步骤：

1. 添加或修改规则 rules。

    * 方法一：打开`/etc/sysconfig/iptables`文件，添加或修改规则后保存。

    * 方法二：通过命令行的方式添加或修改规则rules。

    例如拒绝IP为100.100.120.120的访问请求，可以在终端中通过如下命令设置：
        
                iptables -A INPUT -s 100.100.120.120 -j DROP

2. 重启iptables服务，使规则生效。

        service iptables restart
3. 查看添加或修改的规则rules是否生效。

        iptables -nL --line-number
        或
        service iptables status
        或
        /etc/rc.d/init.d/iptables status

4. 保存规则rules。

        iptables-save > /etc/sysconfig/iptables
        或
        /etc/rc.d/init.d/iptables save

**Note:**修改/etc/sysconfig/iptables 后先重启iptables服务（重新加载新的规则），然后再调用命令保存规则。若修改规则后，直接调用命令保存，`/etc/sysconfig/iptables`的配置会回滚到上次启动服务的配置，之前的修改就全白费了，所以，切记先重启iptables服务，再保存规则！！！

*建议：*每次修改完之后，确认以及测试通过之后，在`/etc/sysconfig/`目录下备份一份，例如`iptables.bak`以便还原。

## 设置iptables随机启动并加载规则
由于整个系统重启之后，系统的iptables会清空，所以我们还需要把`iptables`服务设置为随机启动。

设置iptables开机自动启动：

    chkconfig iptables on    # 开启
    chkconfig iptables off   # 关闭

设置加载规则：

将之前设置好的规则，备份保存在`/etc/sysconfig/iptables.bak`文件中。使用命令:

    iptables-restore < /etc/sysconfig/iptables.bak

加载规则即可。但为了开机启动之后就加载规则，可以将上述加载规则写入到`/etc/rc.local`文件中。




欲更加深入的了解iptables，请参见[iptables指南](http://www.frozentux.net/iptables-tutorial/cn/iptables-tutorial-cn-1.1.19.html#DRAWBACKSWITHRESTORE)。
