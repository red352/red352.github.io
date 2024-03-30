---
title: rsync实现linux文件定时同步
layout: post
date: 2024-02-21 00:00:00 +0800
categories:
  - Linux
image: https://w.wallhaven.cc/full/2y/wallhaven-2ywq6m.jpg
---

使用rsync工具进行文件同步

- 创建脚本文件，脚本如下

    ``` shell
    rsync -avz --progress dir1 dir2 dir3 ubuntu@0.0.0.0:/home/ubuntu/sync
    ```

- 创建定时任务

    ``` shell
    crontab -e
    ```

    ``` shell
    0,30 * * * * bash /root/rsync.sh
    ```

- 查看日志

    ``` shell
    grep CRON /var/log/syslog
    ```
