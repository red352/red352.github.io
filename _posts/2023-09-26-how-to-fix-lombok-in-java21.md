---
title: 解决lombok在java21中报错的问题
layout: post
date: 2023-09-26 19:30:00 +0800
published: true
tags:
  - java
categories:
  - Java
---


在java21中使用lombok时，编译时报错：

``` java
Class com.sun.tools.javac.tree.JCTree$JCImport does not have member field 'com.sun.tools.javac.tree.JCTree qualid'
```

## 解决方法

使用edge-SNAPSHOT版本lombok
在pom.xml中添加如下依赖：

``` xml
<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>edge-SNAPSHOT</version>
    </dependency>
</dependencies>
<repositories>
    <repository>
        <id>projectlombok.org</id>
        <url>https://projectlombok.org/edge-releases</url>
    </repository>
</repositories>
```
