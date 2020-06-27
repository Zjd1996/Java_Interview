# NoSql

## 入门和概述

### 互联网大型网站架构

技术架构演变：

- 访问数据量不大，单个数据库可以应付
  - 数据量太大，一个机器放不下
  - 数据的索引也占空间，一个机器的内存放不下
  - 访问量（读写混合）一个实例不能承受

- Memcahced+MySQL+垂直拆分![image-20200603204610803](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200603204610803.png)
- MySQL主从，读写分离![image-20200603204754843](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200603204754843.png)
- 分表分库+水平拆分+mysql集群：主从的写压力增大！![image-20200603205027100](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200603205027100.png)
- MySQL的扩展性瓶颈，大文件
- 如今：![image-20200603205218898](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200603205218898.png)
- 为什么用NoSQL：
  - 巨量数据，云计算大数据的分析 MySQL不再适用

### 是什么

Not Only SQL，泛指非关系型的数据库。这类型的数据存储不需要固定的模式，无需多余操作就可以横向扩展。

### 能干嘛

- 易扩展，数据之间无关系，容易扩展，无形之间在架构层面上也带来了可扩展能力。
- 大数据量高性能，得益于无关性，数据库结构简单
- 多样灵活的数据模型，无需事先为要存储的数据建立字段，随时可以存储自定义的数据格式 。关系数据库增删字段非常麻烦。

### 产品

Redis、Memcache、Mongdb

### 重点

K-V、cache、persistent

3V+3高：

大数据时代的3V：

- 海量Volume
- 多样Variety
- 实时Velocity

互联网需求的3高：

- 高并发
- 高可扩
- 高性能

### 当下的NoSQL经典应用

sql+nosql一起。

阿里巴巴中文站商品信息。

- 商品基本信息：冷数据，固定数据：关系型数据库mysql/oracle，淘宝的mysql的引擎重写过，淘宝在去O化，（去IOE，IBM小型机、Oracle数据库、EMC存储设备
- 描述、详情、评价（多文字）：文档数据库MongDB（多文字信息描述类，IO压力）
- 商品图片：分布式的文件系统，淘宝的TFS、Google的GFS、Hadoop的HDFS
- 商品关键字：搜索引擎、ISearch淘宝内用
- 商品的波段性的热点高频信息：内存数据库，Tair、Redis、Memcache
- 商品交易、价格计算、积分累计：外部接口

难点：

- 数据类型多样性
- 数据源多样性和变化重构
- 数据源改造而数据服务平台不需要大面积重构

解决办法：

- EAI和统一数据平台服务层
- UDSL：统一数据层
  - 映射
  - API
  - 热点缓存平台

### NoSQL数据模型简介

以电商客户、订单、订购、地址模型为例：

![image-20200604133050408](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200604133050408.png)

NoSQL：不需要建表、数据方便拆分、合并。（聚合模型）

- 高并发操作不建议有关联查询的，互联网公司用冗余数据来避免关联查询
- 分布式事务是支持不了太多的并发的（需要强一致）
- NoSQL不需要join了，KV解决

聚合模型：

- KV
- Bson（Binary Json）

- 列簇![image-20200604134502995](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200604134502995.png)
- 图形![image-20200604134542297](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200604134542297.png)

### NoSQL数据库的四大分类

#### KV

- 新浪：BerkeleyDB+redis
- 美团：redis+taie
- 阿里-百度：memcache+redis

#### 文档型数据库（bson比较多）

- CouchDB
- MongoDB
  - 基于分布式文件存储的数据库，C++
  - 介于关系数据库和非关系数据库之间的产品

#### 列存储数据库

- Cassandra、HBase
- 分布式文件系统

#### 图关系数据库

- 存的是关系，比如朋友圈社交网络、广告推荐系统、社交网络、推荐系统。专注于构建关系图谱
- Neo4J、InfoGrid

#### 对比

![image-20200604135300814](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200604135300814.png)

### 分布式数据库中CAP原理CAP+BASE

- 传统的ACID：原子性、一致性、隔离性、持久性
- CAP：
  - C：Consistency强一致性
  - A：Availability可用性
  - P：Partition tolerance分区容错性

- CAP的3进2：一个分布式系统三种不可能同时满足，最多只能满足两个，**分区容忍是我们必须的**，服务器可能不在同一地点
  - CA：单点集群，满足一致性，传统数据库
  - CP：满足一致性，分区容忍的系统，Redis、MongoDB
  - AP：可用性，分区容忍，大多数网站架构的选择
  - ![image-20200604135916855](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200604135916855.png)



![image-20200604140630099](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200604140630099.png)

- BASE：Basically Available基本可用、Soft state软状态、Eventually consistent最终一致性
  - 是为了解决关系数据库强一致引起的问题而引起可用性降低提出的解决方案
  - 通过让系统放松某一时刻数据一致性的要求换取系统整体伸缩性和性能上的改观
- 分布式+集群
  - 分布式：不同的多台服务器上部署不同的服务模块
  - 集群：不同的多台服务器部署相同的服务模块

## Redis入门介绍

### 基础

- REmote DIctinary Server（远程字典服务器），Redis

- 完全开源免费的，C语言编写的，遵循BSD协议
- 高性能的kv分布式内存数据库，基于内存运行，支持持久化的NoSQL数据库
- 区别于其他的特点：
  - 支持持久化
  - 不仅仅kv，还提供list、set、zset、hash等数据结构
  - 支持数据备份，即master-slave模式的数据备份
- 能干嘛
  - 内存存储和持久化
  - 取最新的N个数据
  - 模拟类似于HttpSession这种需要设定过期时间的功能
  - 发布、订阅消息系统
  - 定时器、计数器
- 重点：
  - 数据类型、基本操作和配置
  - 持久化和复制、RDB/AOF
  - 事务的控制
  - 复制

### 杂项

- 单进程，处理客户端请求，对读写事件的响应是通过对epoll函数的包装来做的
- 默认16个databse，0-15号，默认用0号，SELECT 命令切换
- DBSIZE，可以看有几个key
- Keys *，可以看所有的key
- FLUSHDB，清空当前库
- FLUSHALL，清空所有库
- 统一密码管理，16库同样的密码
- 索引从0开始
- 端口6379，merz。。九宫格拼音。。

## Redis数据类型

五大 数据类型

### key



### String

Binary-safe strings。

redis最基本数据类型，二进制安全，即redis的string可以包含任何数据，比如jpg或者序列化的对象。

一个redis字符串value最多可以是512M

### list

按照插入顺序排序，左右两边都可插入，底层链表。

### set

String类型的无序集合，通过HashTable实现的。

### hash

类似于Java的Map。

是string类型的field和value的映射表，特别适合于存储对象。

### zset

sorted set，有序集合。

元素关联一个double分数，根据分数来为集合中的成员从小到大的排序，zset的成员是唯一的，但是分数可以重复。

### 操作

http://redisdoc.com

## Redis配置文件

### 位置

/redis-x-x-x/redis.conf

最好单独拷贝出来，修改，执行

通用：

- daemonize：是否后台
- pidfile：
- port
- tcp-backlog：tcp连接队列，队列综合=未完成三次握手队列+已完成三次握手队列，高并发环境下需要一个高backlog值来避免慢客户端连接，出厂511
- bind
- timeout
- tcp-keepalive：单位为妙，建议60，如果为0表示不回进行keepalive检测
- loglevel：日志级别，四个级别
- logfile
- databases，默认16
- syslog-enabled
- syslog-ident
- syslog-facility

安全：

- config get requirepass/dir。看一些参数
- config set requirepass "123456" // 设置密码

限制：

- Maxclients
- Maxmemory
- Maxmemory-policy
- Maxmemory-samples：样本数量，默认5

![image-20200604155443645](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200604155443645.png)

![image-20200604155455134](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200604155455134.png)

![image-20200604155524531](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200604155524531.png)

![image-20200604155712826](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200604155712826.png)

![image-20200604155732851](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200604155732851.png)

![image-20200604155622363](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200604155622363.png)

## Redis的持久化

### RDB

Redis DataBase。

在指定的时间间隔内将内存中的数据集快找写入磁盘，Snapshot快照，恢复时将快照文件直接读入内存。

- Redis会单独创建（fork）一个子进程来进行持久化
  - fork复制一个与当前进程一样的进程，新进程的所有数据都和原进程一致
  - 但是一个全新的进程，并作为原进程的子进程
- 先将数据写入到一个临时文件，等持久化过程结果，再用这个临时文件替换上次持久化好的文件
- 整个过程，主进程是不进行任何IO操作的，确保了极高的性能
- 如果需要进行大规模的数据恢复，且对于数据恢复的不完整性不是非常敏感，那么RDB方式要比AOF方式更加高效
- RDB的缺点是最后一次持久化后的数据可能丢失
- 保存的是dump.rdb文件
- 配置
  - save xx xx //dump策略
  - Dbfilename dump.rdb
  - Stop-write-on-bgsave-error：save时出错，前台停止写，默认yes
  - rdbcompression yes // 压缩LZF算法
  - Rdbchecksum：CRC64算法进行数据校验，增大10%的性能消耗 
  - dbfilename
  - dir
- 指令
  - flushall也会生成rdb
  - save指令可以指定直接生成dump.rdb，重要数据?
- 如何触发rdb快照
  - 配置文件 save xx xx
    - 冷拷贝后重新使用
  - 命令save
    - 全部阻塞
  - 命令bgsave
    - 后台异步进行快照，同时还可以响应客户端请求
  - flushall也会产生，但是为空，没有意义
- 如何恢复、
  - 将dump.rdb文件移动到redis安装目录并启动redis就可以
- 优势：
  - 适合大规模的数据恢复
  - 对数据完整性和一致性要求不高
- 劣势
  - 意外down，丢失最后一次快照后的数据
  - fork的时候，内存中的数据被克隆一份，大致2倍的膨胀性，需要考虑内存空间
- 如何停止
  - 动态停止rdb：rediscli config set save ""
- 总结![image-20200605150537293](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200605150537293.png)

### AOF

Append Only File。

- 是什么：
  - 以日志的形式记录每个写操作，将Redis执行过的所有写指令记录下来，只允许追加文件但不可以改写文件，redis启动之初回读取该文件重新构建数据，即redis重启就根据日志文件的内容将写指令从头到尾执行一遍以完成数据恢复的工作
- 文件：append only.aof
- 注意：
  - aof和rdb可以共存，如果有aof，会加载aof
  - aof文件损坏？redis-cheak-aof --fix xx.aof修复损坏的aof文件
- 配置：
  - appendonly yes/no
  - appendfilename
  - appendfsync
    - Always：同步持久化，每次发生数据变更时立即记录到磁盘，性能差但数据完整性比较好
    - EverySec：出厂默认，一部操作，每秒记录，如果一秒内宕机，有数据都是
    - No：依赖系统
    - No-appendfsync-on-rewarite：重写时是否可以运用appendfsync，默认no即可，保证数据安全性
    - auto-aof-rewrite-min-szie：重写基准，最小文件大小
    - auto-aof-rewrite-percentage：重写基准， 百分比倍数
- AOF启动、修复、恢复
  - 正常恢复：启动yes，将aof复制到对应目录，重启就加载
  - 异常恢复，备份倍写坏的aof，redis-cheak-aof -- fix修复aof，重启加载
- Rewrie：重写aof，aof文件追加会越来越大，重写机制当AOF文件大小超过阈值，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集，可以使用命令bgrewriteaof
  - 原理：aof文件过大，会fork一条新进程来将文件重写（也是先写临时文件，最后再rename），遍历新进程的内存中数据，每条记录有一条的set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库用命令的方式重写了一个新的aof文件，和快照类似
  - 触发：记录上次重写时的aOF大小，默认是上一次重写的aof的大小的2倍，且文件大于64M时触发![image-20200605154217757](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200605154217757.png)
- 优势：多种
  - 每秒同步
  - 每修改同步
  - 不同步
- 劣势：
  - 相同数据集的数据而言aof文件要远大于rdb文件，恢复速度慢于rdb
  - aof运行效率要慢于rdb，每秒同步策略效率较好，不同步效率与rdb同
- 总结![image-20200605154727502](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200605154727502.png)

### 总结

![image-20200605154851878](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200605154851878.png)

![a](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200605154941643.png)

## Redis事务

### 是什么

可以一次执行多个命令，本质是一组命令的集合。

一个事务中的所有命令都会序列化，按顺序地串行化执行而不会被其他命令插入，不许加塞

### 怎么用

MULTI、EXEC、DISCARD

![image-20200605155710690](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200605155710690.png)

- 正常执行：MULTI。。。。EXEC

- 放弃事务：MULTI。。。。DISCARD

- 全体连坐：MULTI。。。语法错误。。。EXEC，相当于编译器时异常

- 冤有头债有主：MULTI ..语法没错，但执行出错。。。EXEC，只有那一句失败，不会回滚，相当于运行时异常
- WATCH监控：
  - 比如银行可用额度和欠款
  - 为了防止加塞篡改，先watch再开启事务，保证两笔金额变动在同一个事务内。如果watch的变量在watch之后改变了，那么事务就会失败，并回滚
  - 一旦执行了exec，之前的watch都清掉

Redis的事务只是部分支持。

### 三阶段

- 开启：MULTI
- 入队：
- 执行：

### 三个特性

- 单独的隔离操作：事务中的所有命令都会序列化、按顺序执行。不会被其他客户端发来的请求命令打断
- 没有隔离级别概念：队列中的命令在提交后才执行，也就不存在“事务内的查询要看到事务里的更新，在事务外查询不能看到”这个问题
- 不保证原子性：redis同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚

## Redis的发布订阅

### 是什么

进程间的一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接收消息。

SUBSCRIBE  c1 c2 c3，还可以使用通配符订阅多个

PUBLISH c2 hello

## Redis的复制机制

Master/Slave

### 是什么

主从复制，主机数据更新后根据配置和策略，自动同步到备机的机制。

### 能干嘛

读写分离

容灾恢复

### 怎么用

1. 配从不配主
2. 从库配置：slaveof 主库ip 主库端口，每次与master断开后，都需要重新连接，除非写到配置文件
3. 修改配置文件细节操作
   1. 拷贝多个redis.conf文件
   2. 修改端口、pidfile、dump。rdb、logfile
4. 常用模式：
   1. 一主二仆：从机原地待命，即使主机挂了，主机回来照常干活
   2. 薪火相传：去中心化.   Master-slave-slave-slave，减轻主库压力。中途变更转向，会清楚之前的数据
   3. 反客为主：使当前数据库转为主库
5. Info replication。查看信息命令

### 复制原理

增量+全量。

- slave启动成功连接到master发送一个sync命令，全量
- Master接到命令启动后台的存盘进程，同时手机所有接收到的用于修改数据集的命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，完成一次全量同步
- 全量复制：slave在接口到主数据库文件数据后，将其存盘并加载到内存
- 增量复制：Master继续将新的所有收集到的修改命令一次传给slave，完成同步
- 但是只要是重新连接master，就会自动进行一次全量复制

### 哨兵模式

sentinal。反客为主的自动版，监控主库。

如果主库故障了，就根据投票数自动将某从库转换为主库。

#### 怎么用

- 6379带着80 81
- 建立sentinel.conf文件
- 配置哨兵：sentinel monitor 被监控数据库名字（自己起） 127.0.0.1 6379 1 // 1 表示要投票
- 启动哨兵：redid-sentinel sentinel.conf
- 主库宕机，投票选举新的主库
- 原来的主库启动后，成为从库
- sentinel能监控多个master，配置文件可以写多行

```
33993:X 05 Jun 2020 23:51:00.163 # Sentinel ID is 7a49ff76aa1c5b823ffe331be4f7625c3ac64463
33993:X 05 Jun 2020 23:51:00.163 # +monitor master host6379 127.0.0.1 6379 quorum 1
33993:X 05 Jun 2020 23:51:00.173 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:00.174 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:42.408 # +sdown master host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:42.408 # +odown master host6379 127.0.0.1 6379 #quorum 1/1
33993:X 05 Jun 2020 23:51:42.408 # +new-epoch 1
33993:X 05 Jun 2020 23:51:42.408 # +try-failover master host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:42.410 # +vote-for-leader 7a49ff76aa1c5b823ffe331be4f7625c3ac64463 1
33993:X 05 Jun 2020 23:51:42.410 # +elected-leader master host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:42.410 # +failover-state-select-slave master host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:42.498 # +selected-slave slave 127.0.0.1:6381 127.0.0.1 6381 @ host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:42.498 * +failover-state-send-slaveof-noone slave 127.0.0.1:6381 127.0.0.1 6381 @ host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:42.564 * +failover-state-wait-promotion slave 127.0.0.1:6381 127.0.0.1 6381 @ host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:43.152 # +promoted-slave slave 127.0.0.1:6381 127.0.0.1 6381 @ host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:43.152 # +failover-state-reconf-slaves master host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:43.214 * +slave-reconf-sent slave 127.0.0.1:6380 127.0.0.1 6380 @ host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:44.212 * +slave-reconf-inprog slave 127.0.0.1:6380 127.0.0.1 6380 @ host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:44.212 * +slave-reconf-done slave 127.0.0.1:6380 127.0.0.1 6380 @ host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:44.287 # +failover-end master host6379 127.0.0.1 6379
33993:X 05 Jun 2020 23:51:44.287 # +switch-master host6379 127.0.0.1 6379 127.0.0.1 6381
33993:X 05 Jun 2020 23:51:44.288 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ host6379 127.0.0.1 6381
33993:X 05 Jun 2020 23:51:44.288 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ host6379 127.0.0.1 6381
```



### 复制的缺点

- 复制延时：写在主库，同步到slave

## Jedis

```java
// 导包

Jedis jedis = new Jdeis(host, port); // "127.0.0.1", 6379
jedis.ping();
// pong


jedis.set("k", "v");

Set<String> sets = jedis.keys("*");


    

```

### Jedis事务

```java
Jedis jedis = new Jedis("127.0.0.1", 6379);
Transaction t = jedis.multi();
t.set(..);
t.set(..);
t.exec();


t.discard();



// 加锁
int balance;
int debt;
int amtToSubtract = 10;

jedis.watch("balance");
balance = Integer.parseInt(jedis.get("balance"));
if(balance < am) {
	jedis.unwatch();
    sout("modify");
    return false;
    
} else {
    sout("***trancation");
    Transaction t = jedis.multi();
    t.decrBy("balance", am);
    t.incrBy("debt", am);
    t.exec();
}
    
```

### 主从复制

主写从读。读写分离。

```
Jedis jedisMaster = ..
Jedis jedisSlave = ..

jedisSlave.slaveof("127.0.0.1", 6379);
jedisMaster.set...
String res = jedisSlave.get...
```

### JedisPool

单例池子，多个对象

```java
public class JedisPoolUtil {
    private static volatile JedisPool jedisPool = null;
    private JedisPoolUtil() {}
    
    public static JedisPool get() {
        if(null == jedisPool) {
            synchronized(JedisPoolUtils.class) {
                if(null = jedisPool) {
                    JedisPoolConfig  poolConfig = new JedisPoolConfig();
                    poolConfig.setMaxActive(1000); //一个pool可分配多少个jedis实例
                    poolConfig.setMaxIdle(100*1000); //控制一个pool最多有多少个状态位idle的jedis实例
                    poolConfig.setTestOnBorrow(true); // 获得一个jedis实例的时候是否检查连接可用性，ping，如果为true，则得到的jedis实例均是可用的
                    jedisPool = new JedisPool(poolConfig, "127.0.0.1", 6379);
                }
            }
        }
    }
    
    public static void release(JedisPool jedisPool, Jedis jedis) {
        if(null != jedis) {
            jedisPool.returnResourceObject(jedis);
        }
    }
}





psvm {
    JedisPool jedisPool = JedisPoolUtils.get();
    Jedis jedis = null;
    try {
        jedis.jedisPool.getResource();
        jedis.set..;
    } catch(Exception e) {
        e.printStackTrace();
    } finally {
        JedisPoolUtils.release(jedisPool, jedisl);
    }
}
```



## Redis

#### Redis简介

Redis是一个使用ANSI C编写的开源、支持网络、基于内存、可选持久性的键值对存储数据库。是速度非常快的非关系型（NoSQL）内存键值数据库，可以存储五种不同类型的值之间的映射。

键只能为字符串，值支持五种：字符串、列表、集合、散列表、有序集合。

特点：

- 开源
- 多种数据结构
- 基于键值的存储服务系统
- 高性能，功能服务
- 它可以存储键与5种不同类型的值之间的映射；可以进行持久化；可以使用复制来扩展读性能；还可以使用分片来扩展写性能。

#### Redis数据类型

![image-20200218195230602](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200218195230602.png)

##### string

值可以是字符串、数字（整数、浮点数）或者二进制。

整数范围与系统的长整型的取值范围相同（32位系统是32位，64位系统是64位 ），浮点数的精度与double相同。

| 命令                        | 说明                                                      | 时间复杂度                                                   |
| --------------------------- | --------------------------------------------------------- | ------------------------------------------------------------ |
| get key                     | 获取key对应的value                                        | O(1)                                                         |
| set key value               | 设置key value                                             | O(1)                                                         |
| del key                     | 删除key-value                                             | O(1)                                                         |
| incr                        | key自增1，如果key不存在，自增后get(key)=1                 | O(1)                                                         |
| decr                        | key自减1，如果key不存在，自检后get(key)=-1                | O(1)                                                         |
| Incrby key k                | key自增k，如果key不存在，自增后get(key)=k                 | O(1)                                                         |
| decr key k                  | key自减k，如果key不存在，自减后get(key)=-k                | O(1)                                                         |
| set key value               | 不管是否存在                                              | O(1)                                                         |
| setnx key value             | key不存在才设置                                           | O(1)                                                         |
| set key value xx            | key存在才设置？？？？                                     | O(1)                                                         |
| Get key1 key2 key3          | 批量获取key，原子操作                                     | O(N)<br />一次网络时间+n次执行命令时间，如果是n次get，那么n次网络时间+n次命令时间？？？？ |
| Set key1 value1 key2 value2 | 批量设置key-value                                         | O(1)                                                         |
| Getset key newvalue         | set key newvalue并返回旧的value                           | O(1)                                                         |
| append key value            | 将value值追加到旧的value                                  | O(1)                                                         |
| strlen key                  | 返回字符串的长度（注意中文在utf-8下一个中文占用三个字符） | O(1)                                                         |
| incrbyfloat key 3.5         | 增加key的value值3.5                                       | O(1)                                                         |
| getrange key start end      | 获取字符串指定下表所有的值                                | O(1)                                                         |
| setrange key index value    | 设置指定下标所有对应的值                                  | O(1)                                                         |
|                             |                                                           |                                                              |

##### Hash

![image-20200219103705946](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219103705946.png)

| 命令                                      | 说明                                           | 时间复杂度 |
| ----------------------------------------- | ---------------------------------------------- | ---------- |
| hget key field                            | 获取hash key对应field的value                   |            |
| hset key field value                      | 设置hash key对应的field的value                 |            |
| hexists key field                         | 判断hash key是否有field                        |            |
| hlen key                                  | 获取hash key field的数量                       |            |
| hmget key field1 field2 field3            | 批量获取hash key的一批field对应的值            |            |
| Hset key field1 value1 field2 value2 .... | 批量设置hash key的一批field value              |            |
| Hgetall key                               | 返回hash key对应所有的field和value             |            |
| Hvals key                                 | 返回hash key对应所有的field的value             |            |
| Hsetnx key field value                    | 设置hash key对应的field的value（已存在则失败） |            |
| hincrby key field intCounter              | hash key对应的field的value自增intCounter       |            |
| hincrbyfloat key field floatCounter       | 浮点数版本                                     |            |
|                                           |                                                |            |
|                                           |                                                |            |
|                                           |                                                |            |
|                                           |                                                |            |
|                                           |                                                |            |
|                                           |                                                |            |
|                                           |                                                |            |

##### List

![image-20200219104331599](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219104331599.png)

1. lpush + lpop = stack
2. lpush + rpop = queue
3. lpush+ltrim = capped colletion
4. lpush brpop = Message Queue

| 命令                                    | 说明                                                         | 例子                                        | 时间复杂度 |
| --------------------------------------- | ------------------------------------------------------------ | ------------------------------------------- | ---------- |
| rpush key value1 value2 ... valueN      | 从列表右边插入1-N个                                          | rpush list key c b a                        |            |
| lpush key value1 value2 ... vlaueN      | 从列表左边插入1-N个                                          | lpush listkey c b a                         |            |
| Linsert key before/after value newValue | 在list指定值的前/后插入newValue                              | insert list key before b java               |            |
| Lpop key                                | 从列表左侧弹出一个item                                       | lpop listkey                                |            |
| rpop key                                | 从列表右侧弹出一个item                                       | rpop listKey                                |            |
| Orem key count value                    | Count>0？从左到右，删除最多count个value相等的项；count<0?从右到左，删除最多count个value相等的项目；count=0？删除所有value相等的项 | Orem listkey 0 a;<br />Orem list key -1 c   |            |
| ltrim key start end                     | 按照索引范围修建列表                                         | Trim list key 1 4                           |            |
| lrange key start end(include)           | 获取列表指定索引范围所有item                                 | Lrang listkey 0 2;<br />lrange listkey 1 -1 |            |
| llen key                                | 获取列表长度                                                 | llen listkey                                |            |
| lset key index newValue                 | 设置列表指定索引值为newValue                                 | Set listkey 2 java                          |            |
| Blip key timeout                        | lpop阻塞版本，timeout是阻塞超时时间，timeout为0表示永远阻塞  |                                             |            |
| brpop key timeout                       | rpop的阻塞版本，timeout是阻塞超时时间，timeout为0表示永远阻塞 |                                             |            |
|                                         |                                                              |                                             |            |
|                                         |                                                              |                                             |            |
|                                         |                                                              |                                             |            |
|                                         |                                                              |                                             |            |
|                                         |                                                              |                                             |            |
|                                         |                                                              |                                             |            |

##### Set

Redis的Set是String类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

Redis中集合是通过哈希表实现的，所以添加、删除、查找的负载度都是O(1)

特点：

1. 无序
2. 不重复
3. 集合间操作？？

**集合内操作**

| 命令                  | 说明                                                  |      |
| --------------------- | ----------------------------------------------------- | ---- |
| sadd key element      | 向集合key添加element（如果element已经存在则添加失败） |      |
| srem key elemet       | 将集合key中的element移除                              |      |
| Scard key             | 计算集合大小                                          |      |
| sismember key element | 判断element是否在集合中                               |      |
| srandmember key count | 从集合中随机挑count个元素                             |      |
| spop key              | 从集合中随机弹出一个元素                              |      |
| smembers key          | 获取集合所有元素                                      |      |
| srem key element      | 将集合key中的element移除掉                            |      |
|                       |                                                       |      |

**集合间操作**

| 命令                               | 说明                              | 时间复杂度 |
| ---------------------------------- | --------------------------------- | ---------- |
| Sdiff key1 key2                    | 差集                              |            |
| sinter key1 key2                   | 交集                              |            |
| sunion key1 key2                   | 并集                              |            |
| sdiff/sinter/suion + store destkey | 将差集、交集、并集保存在destkey中 |            |
|                                    |                                   |            |
|                                    |                                   |            |

- srandmember不会改变集合
- spop会改变集合（抽奖）
- smember返回的是无序集合，并且要注意量很大的时候会阻塞
- 交集可以用在比如共同关注等地

##### Sorted Set

Redis有序集合和集合一样也是sting类型元素的集合，且不允许重复的成员。不同的是每个元素都会关联一个double类型的分数。redis通过分数来为集合中的成员进行从小到大的排序。有序集合的成员是唯一的，但分数（score）可以重复。

| 命令                                                     | 说明                                                         | 时间复杂度 |
| -------------------------------------------------------- | ------------------------------------------------------------ | ---------- |
| zadd key score element                                   | 添加score和element                                           | O(logN)    |
| zrem key element(...)                                    | 将集合key中的element移除                                     |            |
| Score key element                                        | 返回元素的分数                                               |            |
| zincrby key increScore element                           | 增加或减少元素的分数                                         |            |
| zcard key                                                | 返回元素的总数                                               |            |
| zrank(zrevrank) key member                               | 返回元素的排名                                               |            |
| zrange(zrevrange) key start end [WITHSCORES]             | 返回指定索引范围内的升序元素【分值】                         |            |
| Zrangebyscore(zrevrangebyscore) key minScore maxScore    | 返回指定分数范围内的升序元素                                 |            |
| zcount key minScore maxScore                             | 返回有序集合内在指定分数范围内的个数                         |            |
| Zremrangbyrank key start end                             | 删除置顶排名内的升序元素                                     |            |
| Zremrangebyscore key minScore maxScore                   | 删除指定分数内的升序元素                                     |            |
| zinterstore destination numkeys(表示key的个数) key [...] | 计算给定的一个或多个有序有序集的交集并讲结果存储在新的有序集合key中 |            |
| Zunionstore destination numskeys key [...]               | 计算给定的一个或多个有序集的并集，并存储在新的key中          |            |

适用于各种排行榜

Redis用一个Sorted set解决按两个字段排序的问题，也就是按照热度+时间作为排序字段，关键在于怎么拼接score的问题。这种特点的场景，解决方法是组装一个浮点数，整数部分是热度的值，小数部分是时间。注，redis里面精度应该是小数6位，所以不能把整个日期作为小数部分。（此排序方法即先按热度拍，再按时间排）

![image-20200219121026656](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219121026656.png)

#### Redis数据结构

##### 字典

dictht是一个散列表结构，使用拉链法解决哈希冲突。

##### 跳跃表

是有序集合的底层实现之一。

跳跃表是基于多指针有序链表实现的，可以看成多个有序链表。![image-20200219125636904](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219125636904.png)

在查找时，从上层指针开始查找，找到对应区间之后再到下一层去查找。![image-20200219125710697](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219125710697.png)

与红黑树等平衡树相比，跳跃表具有以下优点：

- 插入速度非常快速，因为不需要进行旋转等操作来维护平衡性；
- 更容易实现；
- 支持无锁操作。

#### Redis使用场景

##### 计数器

可以对string进行自增自减操作，从而实现计数器功能。redis这种内存型数据库的读写性能非常高，很适合存储频繁读写的计数量。

##### 缓存

将热点数据放到内存，设置内存的最大使用以及淘汰策略来保证缓存的命中率。

##### 查找表

例如DNS纪录很合适使用Redis进行存储。

查找表和缓存类似，也是利用了Redis快速的查找特性。但是查找表的内容不能失效，而缓存的内容可以失效，因此缓存不作为可靠的数据来源。

##### 消息队列

List是一个双向链表，可以通过lpush和rpop写入和读取消息。不过最好使用Kafka、RabbitMQ等消息中间件。

##### 会话缓存

可以使用Redis来统一存储多台应用服务器的会话信息。

当应用服务器不再存储用户的会话信息，也就不再具有状态，一个用户可以请求任意一个应用服务器，从而更容易实现高可用性以及伸缩性。

##### 分布式锁实现

在分布式场景下，无法使用单机环境下的锁来对多个节点上的进程进行同步。

可以使用Redis自带的setnx命令实现分布式锁，除此之外还可以使用官方体噶的RedLock分布式锁实现。

##### 其他

Set可以实现交集、并集等操作，从而实现共同好友等功能

ZSet可以实现有序行操作，从而实现排行榜功能

#### Redis与Memcached

两者都是非关系型内存键值数据库，主要有以下区别：

##### 数据类型

Memcached仅支持字符串类型，而Redis支持五种不同的数据类型，可以更灵活地解决问题。

##### 数据持久化

Redis支持两种持久化策略：RDB快照和AOF日志，而Memcached不支持持久化。

##### 分布式

Memcached不支持分布式，只能通过客户端使用一致性哈希来实现分布式存储，这种方式在存储和查询时都需要先在客户端计算一次数据所在的节点。

##### 内存管理机制

- Redis中，并不是所有数据都一直存储在内存中，可以将一些很久没用的value交换到磁盘，而Memcached的数据则会一直在内存中。
- Memcached将内存分割成特定长度的块来存储数据，以完全解决内存碎片的问题。但是这种方式会使得内存利用率不高，例如块的大小为128bytes，只存储100bytes的数据，那么剩下的28bytes就浪费了。

#### Redis键的过期时间与过期策略

Redis可以为每个键设置过期时间，当键过期时，会自动删除该键。

对于散列表这种容器，只能设置整个键过期时间（整个散列表），而不能为键里面的单个元素设置过期时间。

当key的expires超时，有三种过期策略：

##### 定时删除

含义：在设置key的过期时间的同时，为该key创建一个定时器，让定时器在key的过期时间来临时，对key进行删除；

优点：保证内存被尽快释放；

缺点：若过期key很多，删除这些key会占用很多的CPU时间，在CPU时间紧张的情况下，CPU不能把所有的时间用来做要紧的事，还需要花时间去删除这些key；定时器的创建耗时，若为每一个设置过期时间的key创建一个定时器（将会有大量的定时器产生）性能影响严重。**基本不用**

##### 惰性删除

含义：key过期的时候不删除，每次从数据库获取key的时候去检查是否过去，若过期，则删除，返回null；

优点：删除操作只发生在从数据库取出key的时候发生，而且只删除当前的key，所以对CPU的时间的占用是比较少的，而且此时的删除事已经到了非做不可的地步；

缺点：若大量的key在超出超时时间后，很久一段时间内，都没有被获取过，那么可能发生内存泄露（即无用的垃圾占用了大量的内存）

##### 定期删除

含义：每隔一段时间执行一次删除过期key操作；

优点：通过限制删除操作的时长和频率，来减少删除操作对CPU时间的占用

缺点：在内存友好方面，不如定时删除；在CPU时间友好方面，不如惰性删除

难点：合理设置删除操作的执行市场（每次删除执行多长时间）和执行频率（周期）

#### Redis数据淘汰策略

可以设置内存最大使用量，当内存使用量超出时，会实行数据淘汰策略。

Redis有6种淘汰策略：

| 策略            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| Volatile-lru    | 从已设置过期时间的数据集中挑选最近最少使用的数据淘汰         |
| Volatile-ttl    | 从已设置过期时间的数据集中挑选将要过期（最早过期）的数据淘汰 |
| Volatile-randow | 从已设置过期时间的数据集中任意选择数据淘汰                   |
| allkeys-lru     | 从所有数据集中挑选最近最少使用的数据淘汰                     |
| allkeys-random  | 从所有数据集中任意选择数据进行淘汰                           |
| noeviction      | 禁止驱逐数据，当内存不足以容纳新写入时，新写入操作会报错     |

作为内存数据库，处于对性能和内存消耗的考虑，Redis的淘汰算法实际实现上并非针对所有key，而是抽样一小部分并且从中选出被淘汰的key。

使用Redis缓存数据时，为了提高缓存命中率，需要保证缓存数据都是热点数据。可以将内存最大使用设置为热点数据占用的内存量，然后启用allkeys-lru淘汰策略，将最近最少使用的数据淘汰。

Redis4.0引入了volatile-lfu和allkeys-lu淘汰策略，LFU策略通过统计访问频率，将访问频率最少的键值对淘汰。

#### Redis持久化

redis是内存型数据库，为了保证数据在断电后不会丢失，需要将内存中的数据持久化到硬盘上。

##### 快照法

1. mysql dump
2. redis RDB

##### 日志法

1. mysql binlog
2. hbase hLog
3. redis AOF

##### RDB持久化（Redis Database，全量模式）

将某个时间点的所有数据都存放到硬盘上。

可以将快照复制到其他服务器从而创建具有相同数据的服务器副本。

如果系统发生故障，将会丢失最后一次创建快照之后的数据。

如果数据量很大，保存快照的时间会很长。

![image-20200219123429840](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219123429840.png)

RDB时Redis内存到硬盘的快照，用于持久化。

save通常会阻塞Redis，save自动配置满足任一就会被执行。

bgsave不会阻塞redis，但是会fork新进程

**触发方式**

- save（同步）![image-20200219123913914](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219123913914.png)
- bgsave（异步）![image-20200219123939220](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219123939220.png)
- 自动配置![image-20200219124007063](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219124007063.png)

**缺点**

1. 耗时![image-20200219124201791](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219124201791.png)
2. 不可控，丢失数据![image-20200219124218162](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219124218162.png)

##### AOF持久化（Append Only File，增量模式）

将写命令添加到AOF文件的末尾。

使用AOF持久化需要设置同步选项，从而确保写命令同步到磁盘文件上的实际。这是因为对文件进行写入并不会马上将内容同步到磁盘上，而是先存储到缓冲区，然后由操作系统决定什么时候同步到磁盘：

| 选项     | 同步频率                 |
| -------- | ------------------------ |
| always   | 每个写命令都同步         |
| everysec | 每秒同步一次             |
| no       | 让操作系统来决定何时同步 |



备份：![image-20200219124310131](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219124310131.png)

恢复：![image-20200219124329741](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219124329741.png)

**策略**

- always：：严重降低服务器的性能![image-20200219124404679](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219124404679.png)
- everysec：比较合适，可以保证系统奔溃时只会丢失1秒左右的数据，并且Redis美妙执行一次同步对服务器性能几乎没有任何影响![image-20200219124416674](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219124416674.png)
- no：选项并不能给服务器性能带来多大提升，而且会增加系统奔溃时数据丢失的数量。![image-20200219124435692](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219124435692.png)
- ![image-20200219124443257](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219124443257.png)

**重写**

随着服务器写请求的增多，AOF文件会越来越大。Redis提供了一种将AOF重写的特性，能够去除AOF文件中的冗余写命令。

![image-20200219124531360](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219124531360.png)

![image-20200219124544240](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219124544240.png)

**配置**

![image-20200219124557554](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219124557554.png)

**流程**

![image-20200219124628081](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219124628081.png)

**比较**

![image-20200219124733431](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219124733431.png)

##### RDB最佳策略

1. 关
2. 集中管理
3. 主从，从开

##### AOF最佳策略

1. 开，缓存和存储
2. AOF重写集中管理
3. everysec



#### Redis事务

一个事务包含了多个命令，服务器在执行事务期间，不会改区执行其他客户端的命令请求。原子执行

事务中的多个命令被一次性地发送给服务器，而不是一条一条发，这种方式被称为流水线，它可以减少客户端与服务器之间的网络通信次数从而提升性能。

Redis最简单的事物实现方式时使用MULTI和EXEC命令将事务操作包围起来

##### MULTI&EXEC（原子执行，并非互斥）

基本事务只需要MULTI和EXEC命令，这种事务可以让一个客户端在不被其他客户端打断的情况下执行多个命令。被multi和exec命令包围的所有命令会一个接一个地执行，直到所有命令都执行完毕为止。在输入命令时如中间命令有语法错误，那么该命令及其之后的命令不会被执行，之前的命令会被执行。

注：是原子执行，但其他客户端仍可能会修改正在操作的数据？为什么

![image-20200219121357969](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219121357969.png)

##### WATCH&UNWATCH（原子执行+乐观锁）

在事务开启前watch了某个key，在事务提交时会检查key值与watch的时候其值是否发生变化，如果发生变化，那么事务的命令队列不会被执行。

如果用unwatch命令，那么之前的对所有的key的监控一律取消，哪怕之前检测到watch的key值发生变化，也不会对之后的事务产生影响。

##### 分布式锁

watch&multi&exec可能会导致不断循环重试，在竞争激烈时性能很差。

**排它锁**SETNX

setnx：如果不存在，那么设置一个键值对，它是一个原子性操作。

**带有超时特性的锁**

目前的锁在持有者崩溃的时候不会自动释放，这将导致锁一直处于已被获取的状态。

为了给锁加上超时限制特性，程序将在取得锁之后，调用expire命令来为锁设置过期时间，是的redis可以自动删除超时的锁。

为了确保锁在客户端已经奔溃（有可能是在获得锁、设置超时时间之后奔溃，也有可能在设置超时之前奔溃）的情况下仍然能够自动被释放，客户端会在尝试获取锁失败后，检查锁的超时时间，并为未设置超时时间的锁设置超时时间。以你锁总会带有超时时间，并最终因为超时而自动释放，使得其他客户端可以继续尝试获取已被释放的锁。

#### 高级数据结构

BitMap

GEO

#### Redis事件

Redis服务器是一个事件驱动程序。

##### 文件事件

服务器通过套接字与客户端或者其他服务器进行通信，文件事件就是对套接字操作的抽象。

Redis基于Reactor模式开发了自己的网络事件处理器，使用I/O多路复用程序来同时监听多个套接字，并将到达的时间传送给文件事件分派器，分派器会根据套接字产生的事件类型调用相应的事件处理器。![image-20200219214453819](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200219214453819.png)

##### 时间事件

服务器有一些操作需要在给定的时间点执行，时间事件是对这类定时操作的抽象。

时间事件又分为：

- 定时事件：让一段程序在指定的时间内执行一次；
- 周期性事件：让一段程序每隔指定时间就执行一次。

Redis将所有时间事件都放在一个无序链表中，通过遍历整个链表查找出已到达的时间事件，并调用相应的事件处理器。 

##### 事件的调度与执行

服务器需要不断监听文件事件的套接字才能得到待处理的文件事件，但是不能一直监听，否则时间事件无法在规定的时间内执行，因此监听时间应该根据距离现在最近的时间事件来决定。

从事件处理的角度来看，服务器运行流程如下：

![image-20200220113028589](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200220113028589.png)



#### 复制

通过使用slaveof host port命令来让一个服务器成为另一个服务器的从服务器。一个从服务器只能有一个主服务器，并且Redis不支持主主复制。



##### 连接过程

1. slave与master建立连接，发送sync同步命令（slaveof命令）
2. 
3. 主服务器创建快照文件（bgsave），发送给从服务器，并在发送期间使用缓冲区记录执行的写命令（backlog）。快照文件发送完毕之后，开始向从服务器发送存储在缓冲区中的写命令；
4. 从服务器丢弃所有旧数据，载入主服务器发来的快照文件，之后从服务器开始接收主服务器发来的写命令；
5. 主服务器每执行一次写命令，就向从服务器发送相同的写命令。

注意：快照即指RDB方式；slave对用户来说只读，master可写（写到master才能同步到slave）。

![image-20200220154532542](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200220154532542.png)

注意：如果有多个slave节点 并发 发送SYNC命令给master，企图建立主从关系，那么在bgsave完成前收到的sync命令，返回相同的快照和后续的backlog。否则，bgsave完成后的sync将出发master的第二次bgsave。

##### 主从链

随着负载不断上升，主服务器可能无法很快地更新所有从服务器，或者重新连接和重新同步从服务器将导致系统超载。为了解决这个问题，可以创建一个中间层来分担主服务器的复制工作。中间层的服务器是最上层服务器的从服务器，又是最下层服务器的主服务器。

![image-20200220114613001](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200220114613001.png)



#### Sentinel（哨兵）

master节点挂了，redis就不能对外提供写服务了，因为剩下的slave不能成为master。因此有了哨兵模式。

Sentinel（哨兵）可以监听集群中的服务器，并在主服务器进入下线状态时，自动从从服务器中选举出新的主服务器。

Sentinel也有挂掉的可能，所以Sentinal也会启动多个形成一个Sentinal集群。Sentinal最好与Redis部署在不同机器。

使用Sentinel模式时，客户端直接接单Sentinal的ip和port即可，由Sentinal来提供具体的可提供服务的Redis实现。

Sentinel模式基本可以满足一般生产需求，具备高可用性。但当数据量大到一台服务器存放不下的情况时，主从模式或Sentinal模式就不能满足需求了，这个时候需要对存储的数据分片，将数据存储到多个Redis实例中，即分片。

附：

**Sentinel工作流程**

1sentinel 集群通过给定的配置文件发现 master，启动时会监控 master。通过向 master 发 送 info 信息获得该服务器下面的所有从服务器。
 2sentinel 集群通过命令连接向被监视的主从服务器发送 hello 信息(每秒一次)，该信息包括 sentinel 本身的 ip、端口、id 等内容，以此来向其他 sentinel 宣告自己的存在。

3sentinel 集群通过订阅连接接收其他 sentinel 发送的 hello 信息，以此来发现监视同一个主 服务器的其他 sentinel;集群之间会互相创建命令连接用于通信，因为已经有主从服务器作 为发送和接收 hello 信息的中介，sentinel 之间不会创建订阅连接。
 4s**entinel 集群使用 ping 命令来检测实例的状态**，如果在指定的时间内 (down-after-milliseconds)没有回复或则返回错误的回复，那么该实例被判为下线。

5当 failover 主备切换被触发后，failover 并不会马上进行，还需要 sentinel 中的大多数 sentinel 授权后才可以进行 failover，即进行 failover 的 sentinel 会去获得指定 quorum 个的 sentinel 的授权，成功后进入 ODOWN 状态。如在 5 个 sentinel 中配置了 2 个 quorum，等 到 2 个 sentinel 认为 master 死了就执行 failover。

6**sentinel 向选为 master 的 slave 发送 SLAVEOF NO ONE 命令，选择 slave 的条件是 sentinel 首先会根据 slaves 的优先级来进行排序，优先级越小排名越靠前。如果优先级相同， 则查看复制的下标，哪个从 master 接收的复制数据多，哪个就靠前。如果优先级和下标都 相同，就选择进程 ID 较小的。**

7sentinel 被授权后，它将会获得宕掉的 master 的一份最新配置版本号(config-epoch)，当 failover 执行结束以后，这个版本号将会被用于最新的配置，通过广播形式通知其它 sentinel， 其它的 sentinel 则更新对应 master 的配置。

1到3是自动发现机制:

以 10 秒一次的频率，向被监视的 master 发送 info 命令，根据回复获取 master 当前信息。

**以 1 秒一次的频率，向所有 redis 服务器、包含 sentinel 在内发送 PING 命令，通过回复判 断服务器是否在线。

** 以 2 秒一次的频率，通过向所有被监视的 master，slave 服务器发送当前 sentinel，master 信息的消息。

4是检测机制，5和6是 failover 机制，7是更新配置机制。

**Sentinel 职责**

Redis Sentinel 的以下几个功能。

1. 监控:Sentinel 节点会定期检测 Redis 数据节点和其余 Sentinel 节点是否可达。
2. 通知:Sentinel 节点会将故障转移通知给应用方。
3. 主节点故障转移:实现从节点晋升为主节点并维护后续正确的主从关系(Raft 主从选举)。
4. 配置提供者:在 Redis Sentinel 结构中，客户端在初始化的时候连接的是 Sentinel 节点集合，从中获取主节点信息。

#### 集群

cluster 的出现是为了解决单机 Redis 容量有限的问题，将 Redis 的数据根据一定的规则分配 到多台机器。

cluster 可以说是 Sentinal 和主从模式的结合体，**通过 cluster 可以实现主从和 master 重选 功能**，所以如果配置两个副本三个分片的话，就需要六个 Redis 实例。
 配置集群需要至少 3 个主节点(每个主节点对应一个从节点，这样一共 6 个节点)。

Redis 集群提供了以下两个好处:

**将数据自动切分(split)到多个节点的能力。 当集群中的一部分节点失效或者无法进行通讯时， 仍然可以继续处理命令请求的能力。**

不同的 master 存放不同的数据，所有的 master 数据的并集是所有的数据。 master 与之对应的 slave 数据是一样的。

#### 分片

分片是将数据划分为多个部分的方法，可以将数据存储到多台机器里面，这种方法在解决某些问题时可以获得线性级别的性能提升。

假设有4个Redis实例R0，R1，R2，R3，还有很多表示用户的键user：1，user：2，有不同的方式来选择一个指定的键存储在哪个实例中。

- 最简单的方式是范围分片，例如id从0~1000存储到实例R0中，id从1001~2000存储到实例R1中。但是这样需要维护一张映射范围表，维护操作代价很高；
- 哈希分片（哈希槽，并非一致性哈希）方式，使用CRC32哈希函数将键转换为一个数字，再对实例数量求模就能知道应该存储的实例。
- hash和范围结合：典型方式是一致性 hash。首先对 key 进行哈希计算，得到值域有限的 hash 值，再对 hash 值只做范围映射，确定该 key 对应的业务数据存放的具体实例。这种方式的优势是节点新增或退出时，涉及的数据迁移量小——变更的节点上涉及的数据只需和相邻节点发生迁移关系。

根据执行分片的位置，可以分为三种分片方式：

- 客户端分片：客户端使用一致性哈希算法决定键应当分布到哪个节点；
- 代理分片：将客户端请求发送到代理上，由代理转发请求到正确的节点上；
- 服务器分片：Redis Cluster

##### 一致性哈希与普通哈希

**普通哈希算法**

  假如有cache主机5台分别为cacheA、cacheB、cacheC、cacheD、cacheE

  当程序进行hash时，首先每个节点要根据自己的唯一参数哈希出一个值来（如根据ip进行哈希）

  主机哈希完成后形成的哈希值如下

  cacheA    0

  cacheB    1

  cacheC    2

  cacheD    3

  cacheE    4

  主机形成哈希值后，我们以缓存来做实例，比如某个用户登录后会有一个唯一的pin值，然后根据

  这个pin值进行hash求余。因为求余是用5来求余，所以数值肯定会小宇5 因此就有了落点，然后把

  缓存数据进行缓存到落点服务器中

**一致性哈希算法**

  假如有cache主机5台分别为cacheA、cacheB、cacheC、cacheD、cacheE

  当程序进行hash时，首先每个节点要根据自己的唯一参数哈希出一个值来（如根据ip进行哈希）

  主机哈希完成后形成的哈希值如下

  cacheA    key1

  cacheB    key2

  cacheC    key3

  cacheD    key4

  cacheE    key5

然后5台节点围绕称一个环形，如图

![img](https://images2017.cnblogs.com/blog/1078856/201708/1078856-20170815112509506-1168151038.png)

主机形成哈希值后，我们以缓存来做实例，比如某个用户登录后会有一个唯一的pin值，然后根据

 这个pin值进行hash。这里注意和普通hash不一样的就是这里只hash不求余，当hash出一个数值

后，查看这个这个值介于那两个值之间比如介于key4和key5之间，然后从这个落点开始顺时针查

找遇到第一个cache服务器后就把这个服务器当作此次缓存的落点，从而把这个缓存放到这个落点

上。如图:

![img](https://images2017.cnblogs.com/blog/1078856/201708/1078856-20170815112611725-2101277250.png)

两者比较：

​    经过上面的介绍我们能看出，假如普通的哈希当其中任意一台机器down掉后，我们整个的

 缓存都将安然无存，为什么这么说呢？

​    因为当某台cache down掉后我们需要重选算落点，除数已经变了，所以都要重新set缓存，

 当然不排除有一些两个pin值算的落点是一样的，但是当cache服务器增加到一定数量后，前面所

 说的可能性几乎为0，所以称之为全部缓存都要重新set。

​    而使用一致性哈希当其中一台机器down掉后，只有它上面到它这一段的环路缓存失效了，如：

当cache3 down掉后，那落掉在cache2到cache3之间的pin都将失效，那么失效的落点讲继续顺时

针查找cache服务器，那么新的落点都将落到cache4上。

##### 哈希标签

键哈希标签是一种可以让用户指定将一批键都能够被存放在同一个槽中的实现方法，用户唯 一要做的就是按照既定规则生成 key 即可，这个规则是这样的，如果我有对于同一个用户有 两种不同含义的两份数据，我只要将他们的键设置为下面即可:
 abc{userId}def 和 ghi{userId}jkl

redis 在计算槽编号的时候只会获取{}之间的字符串进行槽编号计算，这样由于上面两个不同 的键，{}里面的字符串是相同的，因此他们可以被计算出相同的槽。

##### 重定向客户端

Redis Cluster 并不会代理查询，那么如果客户端访问了一个 key 并不存在的节点，这个节点 是怎么处理的呢?比如我想获取 key 为 msg 的值，msg 计算出来的槽编号为 254，当前节 点正好不负责编号为 254 的槽，那么就会返回客户端 MOVED 槽数 所在节点地址。 如果根据 key 计算得出的槽恰好由当前节点负责，则当期节点会立即返回结果。没有代理的 Redis Cluster 可能会导致客户端两次连接集群中的节点才能找到正确的服务，推荐客户端缓 存连接，这样最坏的情况是两次往返通信。

#### 节点间通信协议

通过 Gossip 协议来进行节点之间通信。
 gossip protocol，简单地说就是集群中每个节点会由于网络分化、节点抖动等原因而具有不同的集群全局视图。节点之间通过 gossip protocol 进行节点信息共享。这是业界比较流行的去中心化的方案。
 其中 Gossip 协议由 MEET、PING、PONG 三种消息实现，这三种消息的正文都由两个 **clusterMsgDataGossip** 结构组成。

共享以下关键信息:
 1)数据分片和节点的对应关系
 2)集群中每个节点可用状态 3)集群结构发生变更时，通过一定的协议对配置信息达成一致。数据分片的迁移、故障发 生时的主备切换决策、单点 master 的发现和其发生主备关系的变更等场景均会导致集群结 构变化。

4)pub/sub 功能在 cluster 的内部实现所需要交互的信息

#### 主从选举——Raft

Redis Cluster 重用了 Sentinel 的代码逻辑，不需要单独启动一个 Sentinel 集群，Redis Cluster 本身就能自动进行 Master 选举和 Failover 切换。

集群中的每个节点都会定期地向集群中的其他节点发送 PING 消息，以此交换各个节点状态 信息，检测各个节点状态:在线状态、疑似下线状态 PFAIL、已下线状态 FAIL。

**选新主的过程基于 Raft 协议选举方式来实现的。**

![image-20200220160713076](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200220160713076.png)

```
以下是故障转移的执行步骤:
```

1)从下线主节点的所有从节点中选中一个从节点 

2)被选中的从节点执行 SLAVEOF NO NOE 命令，成为新的主节点

3)新的主节点会撤销所有对已下线主节点的槽指派，并将这些槽全部指派给自己 

4)新的主节点对集群进行广播 PONG 消息，告知其他节点已经成为新的主节点

 5)新的主节点开始接收和处理槽相关的请求

#### 一个简单的论坛系统分析

论坛功能：

- 发布文章
- 文章点赞
- 在首页可以按文章的发布时间或者文章的点赞数进行排序显示

##### 文章信息

文章包括标题、作者、赞数等信息，在关系型数据库中很容易构建一张表来存储，在Redis可以用HASH来存储每种信息以及其对应的值的映射。

Redis没有关系型数据库中的表这一概念来将同种类型的数据存放在一起，而是使用命名空间的方式来实现这一功能。键名的前面部分存储命名空间，后面部分的内容存储ID，通常使用”：“分隔。例如下面HASH的键名为article:92617，其中article为命名空间，ID为92617.

![image-20200220152525648](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200220152525648.png)

##### 点赞功能

当有用户为一篇文章点赞时，除了要对该文章的votes字段加一，还必须记录该用户已经对该文章进行了点赞，防止用户点赞次数超过1。可以建立文章的已投票用户集合来进行记录。

为了节约内存，规定一篇文章发布满一周之后，就不能再对它进行投票，而文章的已投票集合也会被删除，可以为文章的已投票集合设置一个一周的过期时间就能实现这个规定。

![image-20200220152751134](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200220152751134.png)

##### 对文章进行排序

为了按发布时间和点赞数进行排序，可以建立一个文章发布时间的有序集合和一个文章点赞数有序集合。（下图score就是点赞数；其中的值 1332065417.47根据时间和点赞数间接计算出的。

![image-20200220152856547](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200220152856547.png)

#### Redis配置文件

redis.conf 配置项说明如下:
 \1. Redis 默认不是以守护进程的方式运行，可以通过该配置项修改，使用 yes 启用守护进程

daemonize no
 \2. 当 Redis 以守护进程方式运行时，Redis 默认会把 pid 写入/var/run/redis.pid 文件，可以 通过 pidfile 指定

pidfile /var/run/redis.pid
 \3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379 作为默认端口，因为 6379 在手机按键上 MERZ 对应的号码，而 MERZ 取自意大利歌女 Alessia Merz 的名字

port 6379
 \4. 绑定的主机地址

bind 127.0.0.1
 5.当 客户端闲置多长时间后关闭连接，如果指定为 0，表示关闭该功能

timeout 300
 \6. 指定日志记录级别，Redis 总共支持四个级别:debug、verbose、notice、warning，默认 为 verbose

loglevel verbose
 \7. 日志记录方式，默认为标准输出，如果配置 Redis 为守护进程方式运行，而这里又配置为 日志记录方式为标准输出，则日志将会发送给/dev/null

logfile stdout
 \8. 设置数据库的数量，默认数据库为 0，可以使用 SELECT <dbid>命令在连接上指定数据库 id

databases 16
 \9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合

save <seconds> <changes>
 Redis 默认配置文件中提供了三个条件:
 save 900 1
 save 300 10
 save 60 10000
 分别表示 900 秒(15 分钟)内有 1 个更改，300 秒(5 分钟)内有 10 个更改以及 60 秒

内有 10000 个更改。

\10. 指定存储至本地数据库时是否压缩数据，默认为 yes，Redis 采用 LZF 压缩，如果为了节 省 CPU 时间，可以关闭该选项，但会导致数据库文件变的巨大

rdbcompression yes
 \11. 指定本地数据库文件名，默认值为 dump.rdb

dbfilename dump.rdb
 \12. 指定本地数据库存放目录

dir ./
 \13. 设置当本机为 slav 服务时，设置 master 服务的 IP 地址及端口，在 Redis 启动时，它会 自动从 master 进行数据同步 slaveof <masterip> <masterport>

\14. 当 master 服务设置了密码保护时，slav 服务连接 master 的密码

masterauth <master-password>
 \15. 设置 Redis 连接密码，如果配置了连接密码，客户端在连接 Redis 时需要通过 AUTH <password>命令提供密码，默认关闭

requirepass foobared
 \16. 设置同一时间最大客户端连接数，默认无限制，Redis 可以同时打开的客户端连接数为 Redis 进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户 端连接数到达限制时，Redis 会关闭新的连接并向客户端返回 max number of clients reached 错误信息

maxclients 128
 \17. 指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后， Redis 会先尝试清除已到期或即将到期的 Key，当此方法处理 后，仍然到达最大内存设置， 将无法再进行写入操作，但仍然可以进行读取操作。Redis 新的 vm 机制，会把 Key 存放内 存，Value 会存放在 swap 区

maxmemory <bytes>
 \18. 指定是否在每次更新操作后进行日志记录，Redis 在默认情况下是异步的把数据写入磁 盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis 本身同步数据文 件是按上面 save 条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为 no

appendonly no
 \19. 指定更新日志文件名，默认为 appendonly.aof

appendfilename appendonly.aof
 \20. 指定更新日志条件，共有 3 个可选值:

no:表示等操作系统进行数据缓存同步到磁盘(快) always:表示每次更新操作后手动调用 fsync()将数据写到磁盘(慢，安全) everysec:表示每秒同步一次(折衷，默认值)
 appendfsync everysec

\21. 指定是否启用虚拟内存机制，默认值为 no，简单的介绍一下，VM 机制将数据分页存放， 由 Redis 将访问量较少的页即冷数据 swap 到磁盘上，访问多的页面由磁盘自动换出到内存 中(在后面的文章我会仔细分析 Redis 的 VM 机制)

vm-enabled no
 \22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个 Redis 实例共享

vm-swap-file /tmp/redis.swap
 \23. 将所有大于 vm-max-memory 的数据存入虚拟内存,无论 vm-max-memory 设置多小,所 有索引数据都是内存存储的(Redis 的索引数据 就是 keys),也就是说,当 vm-max-memory 设 置为 0 的时候,其实是所有 value 都存在于磁盘。默认值为 0

vm-max-memory 0
 \24. Redis swap 文件分成了很多的 page，一个对象可以保存在多个 page 上面，但一个 page 上不能被多个对象共享，vm-page-size 是要根据存储的 数据大小来设定的，作者建议如果 存储很多小对象，page 大小最好设置为 32 或者 64bytes;如果存储很大大对象，则可以使 用更大的 page，如果不 确定，就使用默认值

vm-page-size 32
 \25. 设置 swap 文件中的 page 数量，由于页表(一种表示页面空闲或使用的 bitmap)是在放在内存中的，，在磁盘上每 8 个 pages 将消耗 1byte 的内存。 vm-pages 134217728

\26. 设置访问 swap 文件的线程数,最好不要超过机器的核数,如果设置为 0,那么所有对 swap 文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为 4

vm-max-threads 4
 \27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启

glueoutputbuf yes
 \28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法

hash-max-zipmap-entries 64

hash-max-zipmap-value 512
 \29. 指定是否激活重置哈希，默认为开启(后面在介绍 Redis 的哈希算法时具体介绍)

activerehashing yes
 \30. 指定包含其它的配置文件，可以在同一主机上多个 Redis 实例之间使用同一份配置文件， 而同时各个实例又拥有自己的特定配置文件

include /path/to/local.conf