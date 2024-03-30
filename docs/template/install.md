---
layout: page
title: 本地安装方法
menubar: docs_menu
show_sidebar: false
toc: true
---

[查看文档](https://www.csrhymes.com/bulma-clean-theme/docs/getting-started/installation/)

## 远程主题

直接运行

``` bash
bundle exec jekyll serve
```

## 本地主题

首先进入`Gemfile`，注释给github page的依赖，取消注释本地运行主题依赖

其次进入`_config.yml`，注释远程主题，取消注释本地主题

最后本地运行

``` bash
bundle exec jekyll serve
```
