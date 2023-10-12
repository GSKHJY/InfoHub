# Redis

## Redis 特性

- 读写性能优异：Redis能读的速度是110000次/s,写的速度是81000次/s （测试条件见下一节）。
- 数据类型丰富：Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
- 原子性：Redis的所有操作都是原子性的，同时Redis还支持对几个操作全并后的原子性执行。
- 丰富的特性：Redis支持 publish/subscribe, 通知, key 过期等特性。
- 持久化：Redis支持RDB, AOF等持久化方式
- 发布订阅：Redis支持发布/订阅模式
- 分布式：Redis Cluster

## Redis 使用场景

- 缓存
- 排行榜系统
- 限时活动
- 计数器应用
- 社交网络
- 消息队列系统
- 分布式锁

## Redis 架构模式
https://xie.infoq.cn/article/fe070dcadf891d3d641132c36

我行 Rddis 集群标准为3服务器6节点，即每台服务器上起两个节点实例，且进行主备角色配置，达到集群最大可用性。

## Redis 基础数据类型

Redis 有5种基础数据类型：String，List，Hash，Set，ZSet。

### String

String 可以是字符串、整数或浮点数。可以对整个字符串或字符串的一部分进行操作，或对整数或浮点数进行自增或自减操作。

### List

链表结构，可以对链表的两端进行push和pop操作，读取单个或多个元素，根据值查找或删除元素。常用来实现时间轴、消息队列功能。

### Hash

键值对，用于存储对象。

### Set

成员唯一的无序集合。可用于打标签，按赞点踩功能。

### Zset

在 Set 的基础上，每个成员都有一个分数。可用于实现排行榜功能。

## Redis 持久化策略

Redis 主要有两种持久化方式，RDB（Redis DataBase）和 AOF（Append-Only File）。

### RDB

RDB 持久化是把当前进程数据生成快照保存到磁盘上的过程，由于是某一时刻的快照，那么快照中的值要早于或者等于内存中的值。

RDB 有手动触发和自动触发两种触发方式。

手动触发：
- save命令：阻塞当前Redis服务器，直到RDB过程完成为止，对于内存 比较大的实例会造成长时间阻塞，线上环境不建议使用；
- bgsave命令：Redis进程执行fork操作创建子进程，RDB持久化过程由子 进程负责，完成后自动结束。阻塞只发生在fork阶段，一般时间很短。

自动触发：
- redis.conf中配置save m n，即每 m 秒内有 n 次修改时，自动触发 bgsave 生成 rdb 文件；
- 主从复制时，从节点要从主节点进行全量复制时也会触发 bgsave 操作，生成当时的快照发送到从节点；
- 执行 debug reload 命令重新加载 redis 时也会触发 bgsave 操作；
- 默认情况下执行 shutdown 命令时，如果没有开启 aof 持久化，那么也会触发 bgsave 操作。

使用 RDB 模式的优点：
- RDB 文件是某个时间节点的快照，默认使用 LZF 算法进行压缩，压缩后的文件体积远远小于内存大小，适用于备份、全量复制等场景；
- Redis加载RDB文件恢复数据要远远快于 AOF 方式；

使用 RDB 模式的缺点：
- RDB 方式实时性不够，无法做到秒级的持久化；
- 每次调用 bgsave 都需要 fork 子进程，fork 子进程属于重量级操作，频繁执行成本较高；
- RDB 文件是二进制的，没有可读性，AOF 文件在了解其结构的情况下可以手动修改或者补全；
- 版本兼容 RDB 文件问题。

### AOF

AOF 采用写后日志，即先写内存，后写日志。AOF 写回策略共有三种，默认的配置项为 Everysec：



AOF 重写过程是由后台进程 bgrewriteaof 来完成的。主线程 fork 出后台的 bgrewriteaof 子进程，fork 会把主线程的内存拷贝一份给 bgrewriteaof 子进程，这里面就包含了数据库的最新数据。然后，bgrewriteaof 子进程就可以在不影响主线程的情况下，逐一把拷贝的数据写成操作，记入重写日志。

使用 AOF 模式的优点：
- 实时性强，发生故障丢失数据少。

使用 AOF 模式的缺点：
- 日志文件体积大
- 恢复速度慢

## Redis 单节点启停/交互

启动命令：
```shell
./redis-server /opt/redis/redis.conf
```

Redis 默认启动在6379端口，可通过 --port 参数自定义端口号。

命令行交互

使用 redis-cli 连接到 Redis：
```shell
redis-cli -h 127.0.0.1 -p 6379
```

关闭 Redis
```shell
redis-cli shutdown
```

关闭前考虑先使用 bgsave 进行快照备份。

## Redis 集群重启顺序

1. 停止 Redis 从节点 -> 停止 Redis 主节点 -> 启动 Redis 主节点 -> 启动 Redis 从节点。标准启动顺序，不会造成主从节点切换，但会造成业务中断。
2. 停止某个主节点的所有从节点 -> 启动某个主节点所有的从节点 -> 从节点客户端，手动通过 cluster failover 进行主从切换 -> 停止主节点 -> 启动已经切换成备节点的节点。依次对所有主节点进行上述操作。这种启动顺序不会造成业务中断，但会进行主从切换，切换期间可能造成业务响应缓慢。
3. 应急操作或应急场景，可以根据需要，只重启主节点。

查看集群状态信息：
redis-cli -h redis_ip -p redis_port info cluster

## Redis 命令练习
https://try.redis.io

## 慢查询

慢查询参数配置：
# 慢查询存储阈值，单位是微秒
slowlog-log-slower-than 10000
# 慢查询日志最多存放多少条，线上建议调大至1000以上
slowlog-max-len 100

获取慢查询日志：
# 参数n指定条数
slowlog get [n]

## bigkey
 
### bigkey 定义

bigkey是指key对应的value所占的内存空间比较大，例如一个字符串类 型的value可以最大存到512MB，一个列表类型的value最多可以存储232-1个 元素。如果按照数据结构来细分的话，一般分为字符串类型bigkey和非字符 串类型bigkey：

- 字符串类型:体现在单个value值很大，一般认为超过10KB就是 bigkey，但这个值和具体的OPS相关。 
- 非字符串类型:哈希、列表、集合、有序集合，体现在元素个数过多。 

### bigkey 危害

- 内存空间不均匀(平衡)：bigkey会造成节点的内存空间使用不均匀。 
- 超时阻塞：由于Redis单线程的特性，操作bigkey比较耗时，也就意味 着阻塞Redis可能性增大。 
- 网络拥塞：每次获取bigkey产生的网络流量较大，假设一个bigkey为 1MB，每秒访问量为1000，那么每秒产生1000MB的流量，对于普通的千兆 网卡(按照字节算是128MB/s)的服务器来说简直是灭顶之灾，而且一般服务器会采用单机多实例的方式来部署，也就是说一个bigkey可能会对其他实例造成影响。

### 统计 bigkey 分布
redis-cli --bigkeys

### 查看某个key的字节数
debug object key

## 命令行执行内存调整

连接客户端
redis-cli -h redis_ip -p redis_port

查看当前 maxmemory 值：
config get maxmemory

修改 maxmemory 值：
config set maxmemory 8589934592

写入配置文件：
config rewrite

## 缓存问题

- 一致性
- 穿透
- 击穿
- 雪崩
- 污染
- etc.


## 参考文档

https://pdai.tech/md/db/nosql-redis/db-redis-x-rdb-aof.html

## 进阶学习

- 部门云文档 -> 架构运维 -> 《REDIS标准运维》
- 《Redis 开发与运维》
- 小林coding-图解Redis
- db-tutorial Redis教程
