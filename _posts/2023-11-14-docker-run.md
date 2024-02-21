---
title: docker run 命令
layout: post
date: '2023-11-14 00:00:00 +0800'
categories:
- Docker
---

以下是几个docker run的例子

- mysql

  ``` shell

  sudo docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=123456 -e TZ=Asia/Shanghai -v mysql:/var/lib/mysql -p 3306:3306 mysql:latest --bind-address=0.0.0.0

  ```

- redis

  ``` shell
  sudo docker run -d --name redis -p 6379:6379 -v /etc/config/myredis:/usr/local/etc/redis -v redis:/data redis:latest redis-server /usr/local/etc/redis/redis.conf

  ```
