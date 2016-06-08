title: Redis 订阅发布
date: 2016-06-02 13:03:24
toc: true
tags: [redis]
categories: redis
keywords: [Linux, redis, 配置, configure, 发布订阅, subscribe, publish]
description: redis 发布订阅相关内容，含配置。
---

订阅配置
--------

redis 配置文件，EVENT NOTIFICATION 一节，配置监听响应哪种事件。
这里配置为 `notify-keyspace-events "KA"`。

<!--more-->

```
############################# EVENT NOTIFICATION ##############################

# Redis can notify Pub/Sub clients about events happening in the key space.
# This feature is documented at http://redis.io/topics/notifications
#
# For instance if keyspace events notification is enabled, and a client
# performs a DEL operation on key "foo" stored in the Database 0, two
# messages will be published via Pub/Sub:
#
# PUBLISH __keyspace@0__:foo del
# PUBLISH __keyevent@0__:del foo
#
# It is possible to select the events that Redis will notify among a set
# of classes. Every class is identified by a single character:
#
#  K     Keyspace events, published with __keyspace@<db>__ prefix.
#  E     Keyevent events, published with __keyevent@<db>__ prefix.
#  g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
#  $     String commands
#  l     List commands
#  s     Set commands
#  h     Hash commands
#  z     Sorted set commands
#  x     Expired events (events generated every time a key expires)
#  e     Evicted events (events generated when a key is evicted for maxmemory)
#  A     Alias for g$lshzxe, so that the "AKE" string means all the events.
#
#  The "notify-keyspace-events" takes as argument a string that is composed
#  of zero or multiple characters. The empty string means that notifications
#  are disabled.
#
#  Example: to enable list and generic events, from the point of view of the
#           event name, use:
#
#  notify-keyspace-events Elg
#
#  Example 2: to get the stream of the expired keys subscribing to channel
#             name __keyevent@0__:expired use:
#
#  notify-keyspace-events Ex
#
#  By default all notifications are disabled because most users don't need
#  this feature and the feature has some overhead. Note that if you don't
#  specify at least one of K or E, no events will be delivered.
notify-keyspace-events "KA"
```

CLI 1 进行订阅操作：
```
sunnogo@R04220:~/workshop/redis_new$ redis-cli 
127.0.0.1:6379> PSUBSCRIBE __keyspace@0__:/RT/*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "__keyspace@0__:/RT/*"
3) (integer) 1
```
CLI 2 进行发布操作：
```
127.0.0.1:6379> HSET /RT/1.1.1.1 nexthop 2.2.2.2
(integer) 1
```

CLI 1 中的响应：
```
1) "pmessage"
2) "__keyspace@0__:/RT/*"
3) "__keyspace@0__:/RT/1.1.1.1"
4) "hset"
```
