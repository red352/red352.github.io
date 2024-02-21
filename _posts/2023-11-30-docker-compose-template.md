---
title: docker-compose-temp
layout: post
date: '2023-11-30 9:28:34 +0800'
categories:
- Docker
---

``` yml
version: "3"
services:
  maven:
    image: maven:3.9.5
    working_dir: /app
    volumes:
      - $HOME/.m2:/root/.m2 # 这里配置maven配置目录，maven设置github的认证token
      - ./:/app
    command:
      - mvn
      - package
  java:
    depends_on:
      - maven
    build:
      dockerfile: Dockerfile
    # 这里参数需配置系统环境变量
    # 或者直接在这里配置
    environment:
      - mysql_host=192.168.0.201
      - mysql_username=root
      - mysql_password=123456
      - qqmail_key=key
    ports:
      - "8080:8080"
```

``` DockeFile
# 阶段二：运行阶段
FROM openjdk:21-slim-buster

# 设置工作目录
WORKDIR /app

# 从构建阶段复制编译好的 JAR 文件到当前镜像
COPY ./target/*.jar ./app.jar

# 暴露容器的 8080 端口
EXPOSE 8080

# 运行应用程序
CMD java  -jar ./app.jar
```
