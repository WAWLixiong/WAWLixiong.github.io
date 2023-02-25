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
     - 一位符号位，31位时间戳, 32位序列号
     - 技巧：key上设计日期避免单个key超过2^32：```inc:prefix:20230210```
   - 优惠券秒杀下单
   - 超卖问题
   ![optimistic_lock_with_version](/imgs/optimistic_lock_with_version.png)
   ![optimistic_lock_with_version](/imgs/optimistic_lock_with_version.png)
   - 一人一单
     - 判断是否有订单记录
     - 分布式锁控制单个用户
   - 分布式锁解决一人一单
     - 简单实现: 通过set(key, val, nx, timeout), ```key: lock:<业务>:<userId>, value: <threadId> timeout: 30s```
       - ttl 兜底
     - 简版实现的问题:
       - 可重入: 通过redis的hash结构+lua脚本解决
       - 可重试: 利用信号量(`todo`)和PubSub功能实现等待，唤醒，获取锁失败重试
       - 超时续约: 利用watchDog, 每隔一段时间(innerLeaseTime/3), 重置超时时间
       - 主从同步异常: 利用联锁MultiLock, 多个节点没有关系, 通过最大失败数量控制获取锁成功
   - redis优化秒杀
     - 商品信息存入redis，redis通过lua脚本原子的完成是否可以下单，减库存，记录用户信息, 异步的完成订单创建
     - redis队列
       - list模拟: y:支持持久化. d:无法避免消息丢失; 只支持消费一次
       - pub/sub: y:支持多生产者多消费者. d: 不支持持久化; 无法避免消息丢失; 堆积后超时数据丢失;
       - stream: y:消息可回溯; 多消费; 阻塞. d: 有漏读风险
   - redis消息队列实现异步秒杀
   ![redis_mq](/imgs/redis_mq.png)
     - xreadgroup, **>** 获取没有delivered的消息, **0**和其他有效或不完整的IDS获取pedning的消息
     - xpending统计信息

      ```redis
        > XREADGROUP GROUP group55 consumer-123 COUNT 1 STREAMS mystream >
        1) 1) "mystream"
          1) 1) 1) 1526984818136-0  # 消息Id
                1) 1) "duration"    # 消费的数据
                    1) "1532"
                    2) "event-id"
                    3) "5"
                    4) "user-id"
                    5) "7782813"

        // 查看消费者组
        127.0.0.1:6379> XPENDING my_stream  my_group
        2) (integer) 2       # 消费组中，所有消费者的pending消息数量
        3) "1605524657157-0" # pending消息中的，最小消息ID
        4) "1605524665215-0" # pending消息中的，最大消息ID
        5) 1) 1) "my_consumer1"  # 消费者1
              1) "1"             # 有1条待确认消息
          1) 1) "my_consumer2"  # 消费者2
              1) "2"             # 有2条待确认消息

        // 查看单个消费者
        127.0.0.1:6379> XPENDING my_stream  my_group 0 + 10 my_consumer1  # 查询(0, 到最大)，10条消息
        6) 1) "1605524665215-0"  # 待ACK消息ID
          1) "my_consumer1"     # 所属消费者
          2) (integer) 847437   # 消息自从被消费者获取后到现在过去的时间(毫秒) - idle time
          3) (integer) 1        # 消息被获取的次数  - delivery counter
      ```

5. 好友关注
   - set处理关注，取关，共同好友，消息推送功能
   - 消息推送
    [!push](/imgs/push.png)
    [!pull](/imgs/pull.png)
    [!push_and_pull](/imgs/push_and_pull.png)
   - 滚动刷新，采用zset数据结构, zrevrangewithscore max min limit offset count
6. 附近商户
   - GeoHash
7. 用户签到
   - BitMap的统计功能
8. UV统计
   - HyperLogLog的统计功能
