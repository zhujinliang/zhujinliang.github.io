---
layout: post
title: "使用 crontab 定时调度任务"
category: "Linux"
date: 2013-01-04
---


工作中经常会需要定时执行某一个任务的需求，在Linux下，可以非常方便的使用 `crontab` 来定时执行脚本，完成任务。


## 使用示例

* 打开任务列表写入

        crontab -e

* 删除任务列掉

        crontab -r

* 查看任务列表

        crontab -l


## crontab的任务列表

`crontab`的任务列表中每个任务的6个部分参数：

* 分钟（0-59）
* 小时（0-23）
* 天（1-31）
* 月份（1-12）
* 工作日（0-6）
* 命令

        Example of job definition:
        .---------------- minute (0 - 59)
        |  .------------- hour (0 - 23)
        |  |  .---------- day of month (1 - 31)
        |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
        |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
        |  |  |  |  |
        *  *  *  *  * user-name  command to be executed

* 每天各小时的第2分钟执行脚本

        02 * * * * /DIR/test.sh

* 每天凌晨2点关机

        * 02 * * * /sbin/shutdown -h

* 每隔6小时执行脚本

        * */6 * * * /DIR/test.sh

## 相关文件文件

运行日志所在目录：

    /var/log/cron


任务列表文件：

    /var/spool/cron/用户

