---
title: logstash的docker配置
layout: post
date: '2023-12-01 00:00:00 +0800'
categories:
- Logstash
- Docker
---

- jdbc插件

    ``` config
    /usr/share/logstash/logstash-core/lib/jars/mysql-connector-j-8.0.33.jar
    ```

- 工作流位置

    ``` config
    /usr/share/logstash/pipeline
    ```

- 数据位置

    ``` config
    /usr/share/logstash/data
    ```

- 环境变量

    `XPACK_MONITORING_ELASTICSEARCH_HOSTS`
    `XPACK_MONITORING_ENABLED`
