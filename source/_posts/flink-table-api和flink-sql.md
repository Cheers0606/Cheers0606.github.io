---
title: flink table api和flink sql
tags: flink sql
overdue: false
categories: flink sql
date: 2021-01-11 14:58:04
updated: 2021-01-11 14:58:04
---
# 表（Table）
+ tableEnvironment可以注册目录Catalog，并可以基于Catalog注册表
+ 表（Table）是有一个"标识符"（identifier）来指定，由3部分组成：Catalog名，数据库（database）名和对象名
+ 表可以是常规的，也可以是虚拟的（视图，view）
+ 常规表（Table）一般可以用来描述外部数据，比如文件、数据库表或消息队列的数据，也可以直接从DataStream转换而来
+ 视图（view）可以从现有表中创建，通常是table API 或者SQL查询的一个结果集

## 创建表
```scala
tableEnv
    .connect(...)       // 定义表的数据来源
    .withFormat(...)    // 定义数据格式化方法
    .withSchema(...)    // 定义表结构
    .createTemporaryTable("myTable"); // 创建临时表（在catalog中注册表）
```
