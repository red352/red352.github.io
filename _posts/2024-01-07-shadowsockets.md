---
title: ss+docker搭建代理服务器
layout: post
date: 2024-01-07 20:30:00 +0800
categories:
  - Network
---

1. *参考教程*
    - [ss官方文档(C实现)](https://shadowsocks.org/doc/deploying.html#docker)
    - [shadowsocks-libev-(ss的C实现的具体源码)](https://github.com/shadowsocks/shadowsocks-libev)

1. 具体部署

    docker环境下

    ``` shell
    docker pull shadowsocks/shadowsocks-libev

    docker run -e PASSWORD=<password> -p<server-port>:8388 -p<server-port>:8388/udp -d shadowsocks/shadowsocks-libev
    ```

1. 客户端连接

    - ss客户端连接配置

    ``` json
    {
        "server":"my_server_ip",
        "server_port":8388,
        "local_port":1080,
        "password":"barfoo!",
        "method":"aes-256-gcm"
    }
    ```

    - 扩展，使用订阅方式，多个客户端可以方便链接
    具体使用[subconverter](https://github.com/tindy2013/subconverter)
