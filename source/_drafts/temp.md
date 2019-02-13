---
title: temp
tags:
---



# 索引

优化sql、索引

索引无效

分多次查询

增、删、改操作会更新索引，不要建太多索引。

update时，如果只set某个有索引的字段，则只更新此索引，不会所有索引都更新。



# 缓存

广州局都是多节点，用分布式缓存redis



# 物化视图及定时更新





# Java分区分表

## 分区

对Java应用透明，只需修改sql，让查询语句尽量定位到同个分区上或少量分区上



## 冗余字段、冗余表



## Hibernate
[hibernate-shards](https://github.com/hibernate/hibernate-shards) 最后更新时间2013-5-6

## Mybatis

### 中间件





实时数据

历史数据 ElasticSearch