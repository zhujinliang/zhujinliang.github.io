---
layout: post
title: "Nginx实现TCP反向代理"
category: "Server"
date: 2015-03-05
---



默认Nginx只支持http的反向代理，要想nginx支持tcp的反向代理，还需要在编译时增加tcp代理模块支持，即nginx_tcp_proxy_module。
同时，为了检测后端主机的状态，还需要其他相关模块支持。为了使用完整功能，需要的模块包括：

* ngx_tcp_module
* ngx_tcp_core_module
* ngx_tcp_upstream_module
* ngx_tcp_proxy_module
* ngx_tcp_upstream_ip_hash_module

<!-- more -->

## 安装

下载nginx，解压，配置，编译，安装。

    wget http://nginx.org/download/nginx-1.4.4.tar.gz
    tar zxvf nginx-1.4.4.tar.gz
    cd nginx-1.4.4
    ./configure --add-module=/path/to/nginx_tcp_proxy_module
    make
    make install


## 配置

nginx.conf主配置文件中增加如下配置配置，tcp 的配置和http的配置是同级的。

    http {
        listen 80;
        location /status {
            check_status;
        }
    }

    tcp {
        upstream tcp_backend {

            server 192。168.1.1:1234;
            server 192。168.1.2:1234;
            server 192。168.1.3:1234;

            check interval=3000 rise=2 fall=5 timeout=1000;
        }
        server {
            listen 8888;
            proxy_pass tcp_backend;
        }
    }

* check interval 健康检查，单位是毫秒
* rise 检查几次正常后，将reslserver加入以负载列表中
* fall 检查几次失败后，摘除realserver
* timeout 检查超时时间，单位毫秒

这会出现一个问题，就是tcp连接会掉线。原因在于当服务端关闭连接的时候，客户端不可能立刻发觉连接已经被关闭，需要等到当Nginx在执行check规则时认为服务端链接关闭，此时nginx会关闭与客户端的连接。

保持连接配置：

    http {
        listen 80;
        location /status {
            check_status;
        }
    }

    tcp {
        timeout 1d;
        proxy_read_timeout 10d;
        proxy_send_timeout 10d;
        proxy_connect_timeout 30;

        upstream tcp_backend {

            server 192。168.1.1:1234;
            server 192。168.1.2:1234;
            server 192。168.1.3:1234;

            check interval=3000 rise=2 fall=5 timeout=1000;
        }
        server {
            listen 8888;
            proxy_pass tcp_backend;
            so_keepalive on;
            tcp_nodelay on;
        }
    }

nginx_tcp_proxy_module模块指令具体参见[这里][nginx tcp proxy module readme]

此文主要内容转载自[运维生存时间][ttlsa]，收藏以备后面用到。


[nginx tcp proxy module readme]: http://yaoweibin.github.io/nginx_tcp_proxy_module/README.html
[ttlsa]: http://www.ttlsa.com/nginx/nginx-tcp-proxy/




