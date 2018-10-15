---
title: redis解读
date: 2018-10-10 11:18:21
tags:
- redis
cover: /2018/09/11/anti-climb/cover.jpg
---
### 开始使用Redis
Redis是一个非常流行的基于内存的轻量级键值数据库(key-value database)。redis原生地在内存中实现了多种类型的数据结构，并提供了操作这些数据结构的多种API。除此之外还提供了高性能命令处理,高可靠性/扩展性的架构及数据持久化等特性。通过[DB-Engine complete ranking](https://db-engines.com/en/ranking)查看各数据库使用排名。  
安装细节自行google。安装完成后，bin目录中会有一些可执行的文件。关于他们的介绍如下表  
| 文件名 | 描述 | 备注 |
| ------ | ------ | ------ |
| redis-server | Redis服务端 | |
| redis-sentinel | Redis Sentinel | redis-server的软链接 |
| redis-cli | Redis命令行工具 |  |
| redis-check-rdb | RedisRDB检查工具 |  |
| redis-check-aof | Redis Append Only Files(AOF)检查工具 |  |
| redis-benchmark | Redis基准/性能测试工具 |  |  
启动和停止Redis服务端的步骤如下。
<code>./redis-server redis.conf</code>

也可以将redis-server以守护进程的方式在后台启动Redis，那么可以编辑配置文件将daemonize参数设置为yes。可以通过<code>Ctrl+C(非后台启动)</code>，或者Kill+PID来停止redis服务。其实更加优雅的方式是<code>redis-cli shutdown</code>。可以通过在通过<code>./redis-cli</code>登录之后。通过<code>info指令</code>查看相关redis服务器的信息。可以通过section参数来指定要哪部分信息 例如 <code>info cpu</code>
| 段落名称 | 描述 |
| ------ | ------ |
| Server | 关于Redis服务器的基本信息 |
| Clients| 客户端连接的状态和指标 |
| Memory | 大致的内存消耗指标 |
| Persistence | 数据持久化相关的状态和指标 |
| Stats | 总体的统计数据 |
| Replication | 主从复制相关的状态和指标 |
| CPU | CPU使用情况 |
| Cluster | Redis Cluster的状态 |
| Keyspace | 数据库相关的统计数据 |
构建redis监控应用的常见实践之一就是通过定期使用<code>info</code>命令来获取信息。  
Redis以其高性能而闻名，最大程度的利用了单线程、非阻塞、多路复用的IO模型来快速地处理请求，某些情况下Redis也会创建线程或子进程来执行某些任务。  
Redis通信协议RESP，例如<code>"echo -e '*l\r\n\$4\r\nPING\r\n' | nc 127.0.0.1 6379"</code>  
具体来说：  
+ 1代表这个数组的大小
+ \r\n(CRLF)是每个部分的终结符。
+ $4表示接下来是四个字符注册的字符串
+ PING为字符串本身

总之，客户端发送给redis服务端的命令实际上就是一组字符串组成的RESP数组。之后，服务器按照不同的命令，使用上述五中RESP类型之一对命令进行响应。
### redis支持的数据类型
+ 字符串(String)数据类型
+ 列表(List)数据类型
+ 哈希(hash)数据类型
+ 集合(set)数据类型
+ 有序集合(sorted set)数据类型
+ HyperLogLog数据类型
+ Geo数据类型
+ 键管理

我们无法像使用SQL来操作关系型数据库那样操作Redis。相反，我们需要直接使用API发送数据所对应的命令，来操作想要操作的目标数据。  
+ 字符串(String)类型 
1. SET命令
2. GET命令
3. STRLEN命令
4. SETRANGE命令
5. SETNX命令
6. MSET及MGET命令
7. OBJECT ENCODING命令(查看与键相关联的Redis值对象在内部的编码方式)。注意<code>Object</code>命令还有很多的功能如refcount和idletime等

+ 列表(List)类型
1. LPUSH命令
2. LRANGE命令
3. RPUSH命令
4. LINSERT命令
5. LINDEX命令
6. LPOP命令
7. RPOP命令
8. LTRIM命令
9. LSET命令
10. BLPOP命令
11. BRPOP命令  


redis在内部使用quicklist存储列表对象。有两个配置选项可以调整列表对象的存储逻辑：
1. list-max-ziplist-size：一个列表条目中一个内部节点的最大大小。
2. list-compress-depth：列表压缩策略。如果我们会用到Redis中列表首尾的元素，那么可以利用这个选项来获得更好的压缩比。  

+ 哈希(Hash)类型
1. HMSET命令
2. HMGET命令
3. HSET命令
4. HGET命令
5. HEXISTS命令
6. HGETALL命令(不建议对数量巨大的哈希使用)
7. HDEL命令
8. HSETNX命令
9. HSCAN命令

如果一个哈希的字段非常多，那么执行HGETALL命令时可能会阻塞Redis服务器。这时候可以使用HSCAN命令来增量的获取所有字段和值  

+ 集合(Set)类型  
1. SADD命令
2. SISMEMBER命令
3. SREM命令
4. SCARD命令
5. SSCAN命令
6. SUNION命令
7. SINTER命令
8. SINTERSTORE命令
9. SDIFF命令
10. SDIFFSTORE命令

+ 有序集合(Sorted Set)类型
1. ZADD命令
2. ZREVRANGE命令
3. ZINCRBY命令
4. ZREVRANK命令
5. ZSCORE命令
6. ZUNIONSTORE命令
7. ZINTERSTORE

+ HyperLogLog类型
1. PFADD命令
2. PFCOUNT命令
3. PFMERGE命令  

Redis中HLL算法返回的基数可能不准确(标准差小于1%),因此在决定是否使用HLL时需要进行权衡。HLL实际上是被当做字符串存储的。

+ Geo类型
1. GEOADD命令
2. GEOPOS命令
3. GEORADIUS命令
4. GEODIST命令
5. GEORADIUSBYMEMBER命令  
6. GEOHASH命令

GEOADD设置坐标时，这些坐标会被内部转换成一个52位的GEOHASH。存储在Geo中的坐标和GEOPOS命令返回的坐标直接可能存在细微的差别。

+ 键的管理
1. DEL阻塞
2. UNLINK(4.0以上引入,主要用于解决大Key的异步删除)
3. EXISTS
4. TYPE
5. RENAME命令在目标键已经存在时先将目标键删除再进行重命名。在大KEY重命名时，先用UNLINK对目标键进行删除再进行rename操作
6. KEYS(阻塞、性能问题)

### 数据特性
+ 位图(bitmap)
+ 键的过期时间
+ SORT命令
+ 管道(pipeline)
+ Redis事务
+ 发布订阅(PubSub)
+ 使用Lua脚本
+ 调试Lua脚本  

在某些统计场景中，位图可以节省大量空间(依据场景而定)。Redis的过期时间会被存储为一个绝对的UNIX时间戳。在一个键过期后，当客户端试图访问已过期的键时，Redis会立即将其从内存中删除。Redis这种删除键的方式为被动删除。对于那些已经过期且永远不会再被访问的键Redis还会定期运行一个基于概率的算法来进行主动删除。  
redis事务与关系型数据库事务的区别在于，redis事务没有回滚功能。一般来说，在一个redis事务中可能会出现两种类型的错误。  
1. 命令语法有错误。在这种情况下入队时就能发现存在预发错误，整个事务会快速失败且事务中的所有命令都不会被处理。
2. 命令成功入队，但是执行过程中发生了错误，卫浴发生错误命令之后的将继续执行，而不会回滚。

Redis PubSub中，channel的生命周期而言，如果给定的频道之前未曾被订阅过，那么SUBSCRIBE命令会自动创建频道。特别注意，PUBSUB相关的机制均不支持持久化。消息、频道和PubSub的关系均不能保存到磁盘上，Redis并没有保证消息投递的可靠性机制。Redis中基于PubSub的键空间通知功能，允许客户端订阅Redis频道来接收命令发布或者数据改变的事件。

Redis Lua脚本，Lua脚本是原子执行，可以增强redis事务。实现更为强大的功能和程序逻辑。Redis的Lua脚本和事务都可以将一组操作原子化地执行，一般来说，当我们的业务场景中设计复杂逻辑判断或者循环处理时，最好选择Lua脚本而不是事务。这里也必须注意在执行Lua脚本期间，Redis服务器不能处理任何其他命令。必须保证Lua脚本尽快执行。一般可以配置Lua脚本运行市场的限制。`lua-time-limit`
### 开发案例

Redis占用空间优化，通过修改配置，让redis尽可能以最优结构存储，以及尽量减少键的个数和键的长度
### 复制  
当主从实例之间网络连接通畅且简历了复制关系之后，主实例会把将其受到的写命令转发给从实例执行，已实现主从实例之间的数据同步。  
在Redis的复制机制中，共有两种重新同步机制：部分重新同步和完全重同步。部分重新同步和完全重新同步是有主实例决定的。当一个从实例被提升为主实例时，其他的从实例必须与新的主实例重新同步。通过优化`repl-backlog-size`参数来充分利用部分同步的优势从而优化主从复制的性能

### 持久化  
Redis共有两种数据持久化类型：RDB和AOF。RDB可以看作是Redis在某一个时间点上的快照，非常适合于备份和灾难恢复。AOF则是一个写入操作的日志，将在服务器启动时被重放。注意RDB文件的配置及AOF文件的配置


### 配置高可用和集群
+ Sentinel  
单个哨兵本身可能失效，所以哨兵显然不足以保证高可用，对主实例进行故障迁移的决策是基于仲裁系统的，所以需要至少3个哨兵进程才能构成一个简装的分布式系统来持续监控Redis主实例的状态。如果有多个哨兵进程检测到主实例下线，其中的哨兵进程会选举出来负责推选一个从实例替代原有的主实例。

+ Redis Cluster


### 生产环境部署

### redis管理

### 故障诊断

### redis模块

### redis生态