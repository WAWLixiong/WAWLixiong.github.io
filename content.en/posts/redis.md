---
title: redis
description: ""
date: 2023-01-24
tags:
  - 202301
  - redis
  - 缓存与数据库一致性
  - 穿透
  - 雪崩
  - 击穿
categories:
  - 2023
  - redis
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
   - 参考
     - <https://zhuanlan.zhihu.com/p/408515044>
     - <https://zhuanlan.zhihu.com/p/194347153>
   - 数据库+缓存的三个核心问题: 缓存利用率, 并发一致性, 缓存和数据库同时成功三个问题
   - 缓存更新
     - Cache Aside Pattern 缓存调用者在更新数据库时更新缓存, 可控性强，**企业采用**
       - 更新还是删除缓存?
         - 更新: 写多读少的场景很多无效更新, 并发时需要加分布式锁同步数据造成性能降低
         - 删除: 更新数据库时使缓存失效, 查询时再更新缓存, **企业采用**
       - 先操作缓存还是数据库
         - 先删缓存后写数据库
         - 先写数据库后删除缓存, 主从同步延迟下需要延迟双删(及等待主从同步时间预计5s后再删除一次缓存) **企业使用**
           - 问题1：延迟时间要大于「主从复制」的延迟时间
           - 问题2：延迟时间要大于线程 B 读取数据库 + 写入缓存的时间
           - 第二步失败的方案: 通过消息队列异步重试删除或者类似阿里的解决方案从binlog收集更新信息，不需要在业务代码中发送消息队列
         ![redis_cache](/imgs/redis_cache.png)
     - Read/Write Through Pattern 整合为一个服务，由服务保证一致性，调用方不用关心
     - Write Behind Caching Pattern 调用者只操作缓存，异步服务将缓存数据持久化到数据库, 风险太高
   - 缓存穿透
     - 缓存空对象
     - 布隆过滤器: 不存在是绝对的，存在可能误判
   ![cache_penetrate](/imgs/cache_penetrate.png)
   ![cache_penetrate_solution](/imgs/cache_penetrate_solution.png)
   - 缓存雪崩
     - 多级缓存：浏览器缓存, nginx加缓存, redis, jvm本地缓存
   ![cache_collapse](/imgs/cache_collapse.png)
   - 缓存击穿(热点key过期重建耗时问题)
   ![cache_breakdown](/imgs/cache_breakdown.png)
   ![cache_breakdown_solution](/imgs/cache_breakdown_solution.png)
   ![cache_breakdown_solution_with_mutex](/imgs/cache_breakdown_solution_with_mutex.png)
   ![cache_breakdown_solution_with_logical_expire](/imgs/cache_breakdown_solution_with_logical_expire.png)
3. 达人探店
   - list, sortset实现排行榜差异
4. 优惠券秒杀
   - redis计数器, Lua脚本, 分布式锁，三种消息队列
   - 全局唯一ID
   - 优惠券秒杀下单
   - 超卖问题
   - 一人一单
   - 分布式锁
   - redis优化秒杀
   - redis消息队列实现异步秒杀
5. 好友关注
   - set处理关注，取关，共同好友，消息推送功能
6. 附近商户
   - GeoHash
7. 用户签到
   - BitMap的统计功能
8. UV统计
   - HyperLogLog的统计功能
