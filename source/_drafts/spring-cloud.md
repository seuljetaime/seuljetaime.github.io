---
title: spring cloud
tags:
---

# 几种架构的对比

## 单节点

服务部署到单个servlet容器上，关闭或重启容器时会导致服务中断。

## 集群Cluster

单节点的复制模式，每个节点提供相同服务。 请求由Http服务器（比如Nginx）转发到节点。

**好处：**

+ 代码基本不需修改( 定时任务那些要处理)
+ 能实现高可用，某个服务节点down了，整体还能正常运行

## 分布式Distributed

一个业务拆分成多个业务子系统，各子系统单独部署。比如图片、定时任务这些可以拆开独立部署。分布式一般部署到不同服务器上。分布式的子系统相对来说业务之间的耦合性会高。

分布式的单个业务子系统也可以做集群。

## 微服务

微服务一般都是分布式的，比分布式更强调单个服务的独立性。每个服务可单独访问，每个服务专注于自己的功能，与其他服务松耦合。微服务是粒度更小、更细的分布式。



**好处：**

+ 服务的复用性更高
+ 单个服务的扩展能力更强
+ 某个服务不可用时，不会导致整个系统不可用

**缺点：**

+ 单个请求要经过多次跨进程的REST HTTP调用，性能会差那么一点
+ 为了可靠性，要做额外的处理
+ 工序太多

# Spring Cloud

## 官方介绍

<p align="center"><b>COORDINATE ANYTHING: DISTRIBUTED SYSTEMS SIMPLIFIED</b></p>

Building distributed systems doesn't need to be complex and error-prone. Spring Cloud offers a simple and accessible programming model to the most common distributed system patterns, helping developers build resilient, reliable, and coordinated applications. Spring Cloud is built on top of Spring Boot, making it easy for developers to get started and become productive quickly.

![架构图](/spring-cloud/diagram-distributed-systems.svg)





功能特性：

+ Distributed/versioned configuration
+ Service registration and discovery
+ Routing
+ Service-to-service calls
+ Load balancing
+ Circuit Breakers
+ Global locks
+ Leadership election and cluster state
+ Distributed messaging

## 服务调用



![Eureka](/spring-cloud/eureka-mini-system.jpg)



Eureka Server提供服务注册功能，所有服务都向此中心注册。

当服务需要调用服务时，向此中心获取调用节点。



服务注册参见：

eureka-server、provider、provoder2代码



服务调用参见：

eureka-server、provider、provoder2、ribbon或feign

## 熔断

![熔断](/spring-cloud/HystrixFallback.png)

当某个服务不可用时

## 服务跟踪

**spring-cloud-starter-sleuth**

在Header中添加了

+ x-b3-traceid 整个链路的标识
+ x-b3-spanid 跨度id
+ x-b3-parentspanid 上级跨度id
+ x-b3-sampled 是否取样输出

运行项目：

+ Eureka Server
+ Provider、Provider2
+ Ribbon

请求Ribbon的链接`http://localhost:8080/test`，之后在Ribbon及Provider或Provider2的 Console可以看到log日志。

Ribbon:

```
2018-11-29 17:28:30.273  INFO [ribbon,cd1fedb64ee5e43d,cd1fedb64ee5e43d,true] 2472 --- [nio-8080-exec-9] ication$$EnhancerBySpringCGLIB$$14060552 : ribbon: echo 'Hello World'
```

Provider:

```
2018-11-29 17:28:30.288  INFO [test-service,cd1fedb64ee5e43d,f4d75a843d97e7b2,true] 1256 --- [nio-7001-exec-3] ication$$EnhancerBySpringCGLIB$$85ca892a : provider2: echo 'Hello World'
```

日志中，分为4个值：

+ 服务应用名，比如ribbon
+ Trace ID 整个链路的标识
+ Span ID 这个链路的跨度ID
+ 是否输出信息到zipkin等服务

### 链路图形化监控

参见zipkin-server

# 参考链接

[Spring Cloud](https://spring.io/projects/spring-cloud)

[Spring Cloud优秀项目](http://www.ityouknow.com/springcloud/2018/08/06/spring-cloud-open-source.html)

[翟永超-Spring-Cloud基础教程](http://blog.didispace.com/Spring-Cloud%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B/)