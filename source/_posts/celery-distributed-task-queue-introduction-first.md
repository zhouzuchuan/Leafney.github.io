---
title: Celery分布式任务队列入门(一)-理论
date: 2017-12-10 23:56:56
updated: 2017-12-11 16:09:45
tags:
    - Celery
categories: 
    - Celery分布式任务队列入门
top: true
---

之前曾在公司的一个分布式爬虫项目中使用 `Celery` 和 `RabbitMQ` 实现过分布式爬虫的功能。最近在整理之前的开发笔记时，看到之前写的关于 `Celery` 的文章，决定趁着有时间再把关于Celery相关的内容好好的整理一番，没想到越写越想把相关的点都理清楚，也就有了这个Celery系列文章。

Celery 是一个简单、灵活且可靠的，处理大量消息的分布式系统，并且提供维护这样一个系统的必需工具。它是一个专注于实时处理的任务队列，同时也支持任务调度。

<!-- more -->

#### 主要模块

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171211/IdE7bgjLE1.png?imageslim)

##### 任务模块 Task

包含异步任务和定时任务。其中，异步任务通常在业务逻辑中被触发并发往任务队列，而定时任务由 Celery Beat 进程周期性地将任务发往任务队列。


##### 消息中间件 Broker

一个消息传输的中间件，可以理解为一个邮箱，作为消费者和生产者之间的桥梁。接收任务生产者发来的消息（即任务），将任务存入队列。Celery 本身不提供队列服务，官方推荐使用 RabbitMQ 和 Redis 等。

##### 任务执行单元 Worker

Worker 是执行任务的处理单元，它实时监控消息队列，获取队列中调度的任务，并执行它。

##### 任务结果存储 Backend

Backend 用于存储任务的执行结果，以供查询。同消息中间件一样，存储也可使用 `RabbitMQ`, `Redis` 和 `MongoDB` 等。

***

#### 系列文章目录

* [Celery分布式任务队列入门(一)-理论](/2017/12/10/celery-distributed-task-queue-introduction-first/)
* [Celery分布式任务队列入门(二)-环境配置](/2017/12/10/celery-distributed-task-queue-introduction-second/)
* [Celery分布式任务队列入门(三)-任务](/2017/12/10/celery-distributed-task-queue-introduction-third/)

未完待续。。。

