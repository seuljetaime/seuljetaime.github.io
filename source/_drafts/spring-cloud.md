---
title: spring cloud
tags:
---

# 几种技术架构的对比

## 单节点

所有服务都由单个servlet容器提供

## 集群Cluster

单节点的复制模式，每个节点提供相同服务。 请求由负载均衡（nginx）转发到节点。

好处：

+ 代码基本不需修改( 定时任务那些要处理)
+ 

## 分布式Distributed

一个业务拆分成多个业务子系统，各子系统单独部署

垂直扩容

## 微服务

微服务一般都是分布式的，比分布式更强调单个服务的独立性。

分布式：分散压力

微服务：分散能力，压力通过水平扩容的形式

水平扩容

好处：

+ 服务的复用性更高

--独立访问

--独立部署



## 其他

高可用、高并发

## Spring Cloud、 Dubbo对比

Spring Cloud是REST形式， dubbo是

Spring Cloud各个组件更全面，dubbo只有

# Spring Cloud

功能：

+ 服务注册发现
+ 负载均衡
+ 配置中心
+ 断路器
+ 链路监控



超时重试

失效转移

事务回滚



# 参考链接

[Spring Cloud优秀项目](http://www.ityouknow.com/springcloud/2018/08/06/spring-cloud-open-source.html)