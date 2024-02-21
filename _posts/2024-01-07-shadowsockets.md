---
title: aws+ss+docker搭建代理服务器
layout: post
date: '2024-01-07 20:30:00 +0800'
categories:
- shadowSocks
---

这里使用的aws是免费领取的一年的，虽然只有1Cpu 1gMem，但带宽很大，几乎没有限制，非常适合拿来做网络io.
其次使用ss+docker部署也是极其简单，两行代码即可搞定，新手友好.

1. *参考教程*
    - [ss官方文档(C实现)](https://shadowsocks.org/doc/deploying.html#docker)
        这里使用了shadowSocks的C实现，完美适配于低端设备和嵌入式设备，占用服务器的内存可以忽略不计，还提供了docker镜像，非常的贴心.
    - [shadowsocks-libev-(ss的C实现的具体源码)](https://github.com/shadowsocks/shadowsocks-libev)

1. 具体部署

    docker环境下

    ``` shell
    docker pull shadowsocks/shadowsocks-libev

    docker run -e PASSWORD=<password> -p<server-port>:8388 -p<server-port>:8388/udp -d shadowsocks/shadowsocks-libev
    ```

1. 客户端连接

    - ss客户但连接配置

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
