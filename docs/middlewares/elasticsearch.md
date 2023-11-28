# Elasticsearch

Elasticsearch（以下简称ES）是一个非结构化的文档型搜索引擎，底层基于 Lucene 开发。Lucene是一个由 Apache 基金会开源的，使用 Java 编写的全文检索库，ES也是用 Java 开发的。ES 将 Lucene 打包成了一个单独的服务，提供了一套简洁的 Restful API ，并具有良好的分布式支持。
ES 发展至今，除了搜索引擎本身之外，还收购了许多其他产品，构建起了一整套生态。旗下产品还包括：Kibana（可视化工具），Beats（单一数据采集器），Logstash（数据采集管道）等，统称为 Elastic Stack。

## Elasticsearch 相关概念

数据概念：
- index：一系列具有相同数据格式（字段数量可以不同）的文档的集合，类似于结构化数据库中“库”的概念。
- type：类比于结构化数据库中“表”。在7.0及之后版本被弱化，一个Index只能包含一个名称为”_doc”的type。
- document：数据的最小单位，以 json 格式存储。

物理概念：
- cluster：集群，可以包含多个 node。
- node：节点，即 Elasticsearch 的一个进程，原则上一台服务器只部署一个node
- shard：分片，每个index可以有多个分片，每个分片存储部分数据，分布在不同的node上，一个node上可以有多个分片


## 倒排索引

![倒排索引](https://p.ipic.vip/b6lf7t.jpeg ':size=80%')

ES 基于 Lucene 开发，Lucene 使用倒排索引来实现快速的全文检索。倒排索引存储在硬盘中。所以使用更快的硬盘，并且避免使用NAS（速度慢，延时大），是 ES 性能调优的一个选项。

倒排索引包含两个部分：
- 单词词典
- 倒排列表

### 单词词典（Term Dictionary）

记录所有文档的单词，记录单词到倒排列表的关联关系。
单词词典一般比较大，可以通过 B+ 树或哈希拉链法实现，以满足高性能的插入与查询。
如果配置了分词器，数据写入索引时会进行格式化（大小写转换，去除特殊符号，分词，etc.），这就要求在查询时对查询条件也进行相同的格式化操作。例如：分词器将 John 转换为了 john，如果在查询时仍使用原始数据 `name=John` 查询，将查询不到任何结果。

### 倒排列表（Posting List）

倒排列表记录了单词对应的文档组合，由倒排索引项组成。
倒排索引项包括：
- 文档ID
- 词频TF：该单词在文档中出现的次数，用于相关性评分
- 位置（Position）：单词在文档中分词的位置（第几个词），用于语句搜索
- 偏移（Offset）：记录单词的开始结束位置（第几个字符到第几个字符），实现高亮显示 

在 mapping 设置中，可以通过 `index_options` 字段控制倒排索引中记录的内容：
- docs：记录 `doc id`
- freqs：记录 `doc id` 和 `term frequencies`
- positions：记录`doc id` / `term frequencies` / `term position`
- offsets：记录`doc id` / `term frequencies` / `term position` / `character offsets`
也可使用 `index: false` 配置来关闭索引。关闭索引后，该字段将不能作为查询条件。

### 不可变性

Lucene 的倒排索引具有不可变性，索引写入磁盘后永远不会被修改。
不可变索引带来的优势：
- 不需要更新索引，所以不需要锁
- 大部分读请求会直接请求内存，带来性能提升
- 其他缓存（如 filter 缓存）在索引生命周期内始终有效

## 插入数据

由于索引具有不可变性，所以当新增数据时，Lucene 通过新增一个倒排索引的方式来解决问题，新增的这个索引被称为**段（segment）**。
数据新增的操作大致可分为3个步骤：
1. 将新的文档写入 ES 的 JVM 内存中，这时这条数据还不能被检索；
2. 每隔一段时间（默认1秒，可配置），JVM内存中的索引被转移到服务器内存中（off-heap），生产一个新的段，这个操作被称为 **refresh**；
3. 每隔一段时间（默认30分钟，可配置），缓存中的段被写入到磁盘中，并生成一个新的提交点（记录了当前索引中存在的所有段的元数据）。写入磁盘后，缓存中的数据被清除，这个操作被称为 **flush**。
由以上步骤可见，ES提供的搜索功能是**近实时（1s）**的。

### 执行间隔

Refresh 和 flush 操作的执行都遵循两个基本逻辑，一是来到了触发时间间隔，一个是数据量达到阈值（对于 refresh 来说是 JVM 内存中的数据，对于 flush 来说是 transaction log 的文件大小），两者都可通过配置文件调整。
transaction log
为保证在 flush 操作的间隙，存储在内存中的数据不会丢失，Lucene 引入了 transaction log 做持久化处理。
每当一个写请求完成后，对应文档会被写入到 transaction log 文件中。执行 flush 操作会清空当前的 transaction log 内容。
当 Elasticsearch 启动的时候， 它会从磁盘中使用最后一个提交点去恢复已知的段，并且会重放 transaction log 中所有在最后一次提交后发生的变更操作。
transaction log 默认每5秒写入硬盘；在每个写请求完成后，也会执行该操作。

## 集群健康度

- 红：至少一个主节点没有分配
- 黄：至少一个副本没有分配
- 绿：主副分片全部正常分配

查看集群健康状态：
`GET  _cat/health?v`

返回第一个未分配 shard 的原因：
`GET _cluster/allocation/explain`


## watermark

ES 使用 watermark 来标记磁盘使用情况，配置位于 `cluster.routing.allocation.disk.watermark`。

watermark 共有三个阶段：low，high 以及 flood_stage

- low：默认85%，超过 low 水位后，ES 不会将分配分片给该节点
- high：默认为90%，超过 high 水位后，ES 尝试将节点上的现有分片重新定位
- flood_stage：默认为95%，进行洪水泛滥阶段后，ES 对节点上分片对应的索引进行强制只读处理。

## 执行明细查看

在 query 上方设置 `profile: true`，来开启 profile API，可以看到 ES 的搜索情况，是如何拆分成底层的 Lucene 请求的，并且会显示每部分的耗时情况，可以用于查询优化以及问题排查：

```json
GET /myIndex/_search
{
  "profile": "true",
  "query": {
    "match": {
      "author": "Tom"
    }
  }
}
```

## ES读写优化设置

写性能优化：
- 只需要聚合不需要搜索的字段，index 设置成 false
- 不需要算分的字段，norms 设置成 false
- 关闭字符串的 dynamic mapping 功能，减少字段数量
- 使用 index_option 控制哪些字段会被添加到倒排索引中，可以一定程度的节约 CPU
- 关闭 _source，减少 IO 操作，但会导致 index 文档与 update 操作失效，适合指标型数据
- 调大 refresh_interval 的数值，减少刷新（refresh）频率，牺牲实时性（需同时调整 indices.memory.index_buffer_size）
- 单个 bulk 请求体的数据量不要太大，官方建议5-15mb
- 写入端的 bulk 请求超时要足够长，建议60s以上
- 写入端尽量将数据轮询打到不同节点
- 索引创建属于计算密集型任务，应该使用固定大小的线程池来配置没，来不及处理的放入队列。
- 线程数应该配置成 CPU 核心数+1，避免过多的上下文切换
- 队列大小可以适当增加，不要过大，否则占用的内存会成为 GC 的负担

读性能优化：
- 尽量使用 Filter Context，利用缓存机制，减少不必要的算分
- 严禁使用`*`开头通配符的 term 查询

## ES内存使用与分配

![ES 内存使用](https://p.ipic.vip/3z37pz.png ':size=100%')

## 我部 ES 整体架构

![ES 架构](https://p.ipic.vip/uf9xr4.png ':size=90%')

## ES 集群重启

1. 禁用分片配置
```json
PUT _cluster/settings
{ 
  "persistent": {
    "cluster.routing.allocation.enable": "primaries"
  }
}
```

2. 停止索引编制并执行同步刷新
```shell
POST _flush/synced
```

3. 杀掉对应 ES 进程
```shell
ps -ef | grep elasticsearch
kill -9 <pid>
```

4. 操作完成后，起应用
```shell
/es7/elasticsearch/bin/elasticsearch -d
```

5. ES 集群健康度为黄后，恢复分片配置
```json
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": null
  }
}
```

## 运维常用命令

查看集群健康情况：
```json
GET _cluster/health
```

查看分片未分配原因：
```json
GET _cluster/allocation/explain
```

查看集群配置：
```json
GET _cluster/settings?include_defaults
```

清除缓存：
```json
POST _cache/clear
```

重新分配分片：
```json
POST _cluster/reroute?retry_failed=true
```

查看内存占用：
```json
GET _cat/nodes?h=name,heapCurrent,fielddataMemory,queryCacheMemory,requestCacheMemory,segmentsMemory&v
```

查看各个索引 fielddataMemory 占用：
```json
GET _stats/fielddata?fields=*
```


查看 JVM：
```json
GET _nodes/stats/jvm?human
```

查看分词结果：
```
GET /analyze_sample/_analyze
{
  "analyzer" : "whitespace",
  "text" : "this is a test"
}
```

## ES 集群同步

![ES集群同步](https://p.ipic.vip/c3ev6t.png ':size=50%')

## 进阶学习

[ES官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/index.html)
