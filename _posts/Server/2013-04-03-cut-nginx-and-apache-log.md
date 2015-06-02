---
layout: post
title: "服务器日志切割脚本"
category: "Server"
date: 2013-04-03
---


## Nginx服务器日志切割

Nginx 是一个非常轻量的 Web 服务器，具有体积小、性能高、速度快等诸多优点。但其生成的日志文件只有一个，不会自动切割日志文件，
长期不去清理，会导致日志文件很大，不便于管理，也不便于出现问题，去查找访问日志。

当然这么点功能难不倒我们强大的shell脚本。


切割脚本：

    #! /bin/bash

    NGINX_LOG_PATH="your_path/nginx/logs"
    NGINX_BACK_LOG_PATH=${NGINX_LOG_PATH}/$(date -d "yesterday" +%Y)/$(date -d "yesterday" +%m)
    NGINX_BACK_LOG_NAME_PREFIX=$(date -d "yesterday" +%Y%m%d)_

    mkdir -p $NGINX_BACK_LOG_PATH

    # Copy old server log
    for LOG_FILE in "site1 site2 site3"
    do
        cp ${NGINX_LOG_PATH}/${LOG_FILE}_access.log ${NGINX_BACK_LOG_PATH}/${NGINX_BACK_LOG_NAME_PREFIX}${LOG_FILE}_access.log
        echo 1> ${NGINX_LOG_PATH}/${LOG_FILE}_access.log
    done

## 定时任务

通过`crontab`将上面的切割日志脚本，加入到系统定时任务当中，具体如下：

    crontab -e
    # 加入以下配置，每天00:00 执行切割nginx日志脚本
    0 0 * * * /your_path/cut_nginx_log.sh

轻松搞定！

如果要做Apache日志的切割也同理，修改日志文件的路径，以及相应的文件名即可。

