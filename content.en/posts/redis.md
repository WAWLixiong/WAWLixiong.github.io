---
title: redis
description: ""
date: 2023-01-24
tags:
  - 202301
  - redis
categories:
  - 2023
menu: main
---


## 特征

1. key-value
2. 单线程，每个命令原子性, 6.0多线程对网络请求的处理
3. 低延迟，高性能(内存，io多路复用，良好的编码)
4. 持久化
5. 主从/分片

<!--more-->

## 数据结构

1. redis-cli 下 help @string 或者 redis.io/commands 查询各种命令手册
2. string
3. list
4. hash
   - 实现相同功能hash比json的string占用更少的内存
5. set
6. zset

## 实战

1. 短信登录
   - redis共享session
   ![redis_login](/imgs/redis_login.png)
   - 基于session保存验证码的问题是tomcat水平扩展时不能在多台主机共享
   ![redis_login_2](/imgs/redis_login_2.png)
   ![redis_login_3](/imgs/redis_login_3.png)
2. 商户查询缓存
   - 企业缓存使用技巧, 缓存雪崩，穿透等问题
   - 主动更新缓存
     - Cache Aside Pattern 缓存调用者在更新数据库时更新缓存, 可控性强，**企业采用**
       - 更新还是删除缓存?
         - 更新: 写多读少的场景很多无效更新
         - 删除: 更新数据库时使缓存失效, 查询时再更新缓存, **企业采用**
       - 缓存与数据库操作同时成功或失败
         - 单体系统: 缓存与数据库操作放在一个事务
         - 分布式系统(通过mq通知其他系统删除缓存): 利用TCC等分布式方案
       - 先操作缓存还是数据库
         - 先删缓存后写数据库
         - 先写数据库后缓存, **企业使用**
         ![redis_cache](/imgs/redis_cache.png)
     - Read/Write Through Pattern 整合为一个服务，由服务保证一致性，调用方不用关心
     - Write Behind Caching Pattern 调用者只操作缓存，异步服务将缓存数据持久化到数据库, 风险太高
3. 达人探店
   - list, sortset实现排行榜差异
4. 优惠券秒杀
   - redis计数器, Lua脚本, 分布式锁，三种消息队列
5. 好友关注
   - set处理关注，取关，共同好友，消息推送功能
6. 附近商户
   - GeoHash
7. 用户签到
   - BitMap的统计功能
8. UV统计
   - HyperLogLog的统计功能