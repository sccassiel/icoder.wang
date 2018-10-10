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
+ 使用字符串类型 
1. SET命令
2. GET命令
3. STRLEN命令
4. SETRANGE命令
5. SETNX命令
6. MSET及MGET命令
7. OBJECT ENCODING命令(查看与键相关联的Redis值对象在内部的编码方式)。注意<code>Object</code>命令还有很多的功能如refcount和idletime等
### 数据特性

### 开发案例

### 复制

### 持久化

### 配置高可用和集群

### 生产环境部署

### redis管理

### 故障诊断

### redis模块

### redis生态