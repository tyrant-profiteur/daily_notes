## 1.1 Redis简介

​	完全免费，遵循 BSD 协议，是一个高性能(NOSQL)的 key-value 数据库。

​	传统数据库对超大规模和高并发的 SNS 类型的 web 2.0 纯动态网站已力不从心。

​	<font color=ff0000>NoSQL 数据库的产生就是为了解决大规模数据集合多重数据类型带来的挑战。</font>

## 1.2 NoSQL介绍

非关系性：数据与数据之间没有关联关系

关系数据库：表与表之间的有关系

主要分为四种：

- 键值(key-value)存储的数据库

  使用到一个 hash 表，表中一个特定的键和一个指针指向特定的数据。如 TokyoCabinet/Tyrant，<font color=ff0000>Redis</font>，Voldemort，Oracle BDB

- 列存储的数据库

  用来应对分布式存储的海量数据。键仍然存在，但是只想多个列，这些列由列家族安排。如 Cassandra，<font color=ff0000>HBase</font>，Riak

- 文档型数据库

  半结构化的数据以特定的格式存储，比如 JSON。可以看作为键值数据库的升级版，允许之间嵌套键值。文档型数据库比键值数据库的查询效率更高，如 CouchDB，<font color=ff0000>MongoDB</font>，国内的 SequoiaDB(已开源)。

- 图形数据库

  使用灵活的图形模式，能够扩展到多个服务器上。如 <font color=ff0000>Neo4j</font>，InfoGrid，Infinite Graph

没有标准的查询语句(SQL)，进行数据查询需要定制数据模型。许多 NoSQL 数据库都有 REST 式的数据接口或者查询 API。

NoSQL 适合环境：

- 数据模型简单
- 灵活度更高的操作系统
- 数据库性能要求比较高
- 不需要高度的数据一致性
- 对给定的 key，比较容易映射复杂值的环境

## 1.3 特点

Redis 的特点：

- 性能高：读的速度是110000次/s,写的速度是81000次/s 

- 支持数据的持久化，可以将内存重的饿数据保存在磁盘中，重启的时候可以再次加载使用。
- 丰富的数据类型：不仅支持 key-value 类型数据，还支持 list，set，zset，hash 等
- 支持数据备份，集群等<font color=ff00ff>高可用</font>功能
- 原子性：操作都是原子的，通过 MULTI 和 EXEC 指令包起来
- 支持 publish/subscribe，通知，key 过期等特性

## 1.4 总结

- 单个 key 存入 512M 大小
- 支持多种类型的数据结构
- 单线程，原子性
- 可持久化，使用 RDB 和 AOF 机制
- 支持集群，支持 16 个库
- 可做消息队列

企业级开发：做数据库，缓存（热点数据，经常被查询，但不经常被修改或删除的数据）和消息中间件等大部分功能。

优点：

- 丰富的数据结构
- 高速读写，使用自己的分离器，代码量短，没有 lock（MySQL）

缺点：

- 持久化
  - 定时快照（snapshot）：每隔一段时间将整个数据库写道磁盘上，每次均写全部数据，代价高
  - 语言追加（aof）：只追踪变化的数据，但是追加的 log 可能豁达，同时所有的操作均重新执行一遍，回复速度慢
- 耗内存，占用内存过高

## 2.Redis 安装

1. Redis 由 C 语言编写，需要 gcc 运行环境

   Ubuntu：*sudo apt-get  install  build-essential* 

2. 下载 Redis tar包，解压到 /usr/local/ 下

3. redis 目录下执行：make MALLOC=libc

4. 执行 make PREFIX=/usr/local/redis install

   PREFIX 必须大写

5. bin 目录下执行 ./redis-server 启动服务器

   ![1585572877812](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis启动成功.png)

6. ./redis-cli 启动客户端

## 3. redis 自动化配置

## 4.数据类型

### 4.1 String

#### 4.1.1 简单操作

- 添加/修改

  ~~~
  set key value
  ~~~

- 获取

  ~~~
  get key
  ~~~

- 删除

  ~~~
  del key
  ~~~

- 添加/修改多个数据

  ~~~
  mset key1 value1 key2 value2
  ~~~

- 获取多个数据

  ~~~
  mget key1 key2
  ~~~

- 获取数据字符个数（字符串长度）

  ~~~
  strlen key
  ~~~

- 追加信息到原始数据后部

  ~~~
  append key value
  ~~~

####  4.1.2 String 类型数据的扩展操作

- 设置数值数据增加指定范围的值

  ~~~
  incr key
  incrby key increment
  incrbyfloat key increment
  ~~~

- 设置数值数据减少指定范围的值

  ~~~
  decr key
  decrby key increment
  ~~~

- 设置指定的生命周期

  ~~~
  setex key seconds value
  psetex key milliseconds value
  ~~~

  如果重新设置 set key,时效就会被清除掉

#### 4.1.3 String 类型数据的注意事项

- 数据操作不成功和数据正常操作之间的差异
- 数据未获取：nil
- 最大存储量：512MB
- 数值计算最大范围：Long.MAX_INDEX(long 的最大值)

key 的命名规范：`user: id:1223:fans` 或者`user: id:1223:{}`   `表名：主键名：主键值：字段名`

### 4.2 hash

- 新需求

  对一系列存储的数据进行编组，方便管理

- 存储结构

  一个存储空间保存多个键值对数据

#### 4.2.1 简单操作

- 添加/修改

  ~~~
  hset key value
  ~~~

- 获取

  ~~~
  hget key field
  hgetall key
  ~~~

- 删除

  ~~~
  hdel key field1 [field2]
  ~~~

- 添加/修改多个数据

  ~~~
  hmset key1 field1 value1 field2 value2
  ~~~

- 获取多个数据

  ~~~
  hmget key1 field1 field2
  ~~~

- 获取数据字符个数（字符串长度）

  ~~~
  hlen key
  ~~~

- 是否存在

  ~~~
  hexists key field
  ~~~

- 获取所有字段名或者字段值

  ~~~
  hkeys key
  hvals key
  ~~~

- 设置指定字段的数值数据增加指定范围的值

  ~~~
  hincrby key field increment
  hincybyfloat key field increment
  ~~~

#### 4.2.2 注意事项

- value 只能存字符串，不存在嵌套现象；未获取到，返回 nil

- 每个 hash 可以存储 2^32 - 1 个键值对

- hash 类型十分贴切对象的数据存储形式，并且可以灵活添加删除对象属性。但是初衷不是如此，不可滥用，更不可将 hash 作为对象列表使用
- hgetall 操作可以获取全部属性，但是如果 field 过多，遍历效率就低，可能成为访问瓶颈

![1589726461851](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-hash的实际用途.png)

~~~
hsetnx key field value
~~~

#### 4.2.3 hash 使用场景

- 实现购物车数据存储
- 抢购

> String 用 json 存储对象适用于读或者数据一次性都修改的场景；hash 存储对象适用于修改

### 4.3 list

- 对数据进入存储的顺序进行区分

#### 4.3.1 基本操作

- 添加/修改数据

  ~~~
  lpush key value1
  rpush key value1
  ~~~

- 获取数据

  ~~~
  lrange key start stop
  lindex key index
  llen key
  ~~~

- 获取并移除数据

  ~~~
  lpop key
  rpop key
  ~~~

  ![1589805313509](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-list.png)

- 规定时间获取并移除数据

  ~~~
  blpop key1 [key2] timeout
  brpop key1 [key2] timeout
  ~~~

#### 4.3.2 扩展操作

- 移除指定数据

  ~~~
  lrem key count value
  ~~~

  ![1589806077514](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-list-lrem.png)

#### 4.3.3 注意事项

- list 保存的数据都是 String 类型，最多 2^32-1 个
- list 具有索引，操作数据通常以队列或者栈的形式进行操作
- 获取全部数据，结束索引设为 -1

- list 可以对数据进行分页操作，通常第一页来自list，第二页及更多通过数据库形式加载 

#### 4.3.4 应用场景

- 依赖 list 数据具有顺序的特征对信息进行管理
- 使用队列模型解决多路信息汇总合并问题
- 使用栈模型解决最新消息的问题

### 4.4 set 类型

- 存储大量的数据，在查询方面提供更高的效率

#### 4.4.1 基本操作

![1589807027451](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\set 数据格式.png)

- 添加数据

  ~~~
  sadd key member1 [member2]
  ~~~

- 获取全部数据

  ~~~
  smembers key
  ~~~

- 删除数据

  ~~~
  srem key member1 [member2]
  ~~~

- 获取集合数据总量

  ~~~
  scard key
  ~~~

- 判断是否包含指定数据

  ~~~
  sismember key member
  ~~~

- 随机获取集合中指定数量的数据

  ~~~
  srandmember key [count]
  ~~~

- 随机获取集合中某个数据并移除集合

  ~~~
  spop key [count]
  ~~~

#### 4.4.2 扩展操作

- 求集合的交、并、差

  ~~~
  sinter key1 [key2]
  sunion key1 [key2]
  sdiff key1 [key2]
  ~~~

- 求集合的交、并、差并存储到指定集合

  ~~~
  sinterstore destination key1 [key2]
  sunionstore destination key1 [key2]
  sadiffstore destination key1 [key2]
  ~~~

- 将指定数据从原集合中移动到目标集合中

  ~~~
  smove source destination member
  ~~~

#### 4.4.3 注意事项

- 不允许数据重复
- set 和 hash 存储结构相同，但是无法启用hash 中存储值的空间

#### 4.4.4 应用场景

![1589808682898](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-set-获取并集.png)

![1589808822349](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-set-应用场景1.png)

![1589808890398](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-set-应用场景1-解决.png)

![1589808916093](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-set-应用场景2.png)

![1589809031776](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-set-应用场景2-解决.png)

### 4.5 zset 类型

- 数据排序有利于数据的有效展示，需要一种可以进行排序的方式

![1589809219643](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\sorted_set 数据格式.png)

#### 4.5.1 基本操作

- 添加数据

  ~~~
  zadd key score1 member1 [score2 member2]
  ~~~

- 获取全部数据

  ~~~
  zrange key start stop [WITHSCORES]
  zrevrange key start stop [WITHSCORES]
  ~~~

- 删除数据

  ~~~
  zrem key member [member]
  ~~~

- 按条件获取数据

  ~~~
  zrangebyscore key min max [WITHSCORES] [LIMIT]
  zrevrangebyscore key min max [WITHSCORES] [LIMIT]
  ~~~

- 按条件删除数据

  ~~~
  zremrangebyrank key start stop
  zremrangebyscore key min max
  ~~~

- 获取集合数据总数

  ~~~
  zcard key
  zcount key min max
  ~~~

- 集合交、并操作

  ~~~
  zinterstore destination numkeys key [key...]
  zunionstore destination numkeys key [key...]
  ~~~

#### 4.5.2 扩展操作

- 获取数据对应的索引

  ~~~
  zrank key member
  zrevrank key member
  ~~~

- score 值获取修改

  ~~~
  zscore key member
  zincrby key increment member
  ~~~

#### 4.5.3 注意事项

- score 保存的数据存储空间 64 位
- score 保存的值也可以是双精度的 double 值，但是可能会损失精度，要慎重
- sorted_set 底层还是 set 结构，数据不能重复

## 5. 通用指令

### 5.1 key 的基本操作

- 删除指定的key

  ~~~
  del key
  ~~~

- 指定的key是否存在

  ~~~
  exists key
  ~~~

- 获取 key 对应的 value 类型

  ~~~
  type key
  ~~~

- 为 key 设置有效期

  ~~~
  expire key seconds
  pexpire key milliseconds
  expireat key timestamp
  pexpireat key milliseconds-timestamp
  ~~~

- 获取 key 的有效期

  ~~~
  ttl key-- -2 key不存在;-1 key存在没有时间
  pttl key
  ~~~

- 切换 key 从时效性转为永久性

  ~~~
  persist key
  ~~~

- 查询key

  ~~~
  keys pattern
  ~~~

  规则：* 匹配任意数量的任意字符

  ​			？匹配一个任意符号

  ​			[] 匹配一个指定符号

- key 改名

  ~~~
  rename key newkey
  renamenx key newkey
  ~~~

- 对 key 排序

  ~~~
  sort
  ~~~

- key 的其他操作

  ~~~
  help @generic
  ~~~

### 5.2  db 的基本操作

#### 5.2.1 基本操作

- 切换数据库

  ~~~
  select index
  ~~~

- 其他操作

  ~~~
  quit
  ping
  echo message
  ~~~

- 数据移动

  ~~~
  move key db
  ~~~

  必须保证移动到的 db 里没有 key 的数据

- 数据清除

  ~~~
  dbsize
  flushdb
  flushall
  ~~~

## 6. 持久化

- RDB：数据快照
- AOF：过程日志

### 6.1 RDB

- 命令

  ~~~
  save
  ~~~

- 作用：手动执行一次保存

- save 相关配置

  - dbfilename dump.rdb

    设置本地数据库文件名

    通常为 dump-端口号.rdb

  - dir

    设置存储路径

    通常设置成存储空间大的目录，目录名 data

  - rdbcompression yes

    设置存储至本地时是否压缩数据，默认 yes，采用 LZF 压缩

    通常默认为开启，如果为 no，可以节省 CPU 运行时间，但是文件会巨大

  - rdbchecksum yes

    是否进行 RDB 文件格式校验，校验过程在读写的时候都会进行

    通常默认为开启，如果为 no，可以节省 10% 的时间损耗，但是存储一定的数据损坏风险
  
- 单线程，线上环境不建议使用，会阻塞

- ~~~
  bgsave
  ~~~

  会在后台执行，调用 fork 函数，生成子进程

- 其他配置

  - 其他四个

  - stop-writes-on-bgsave-error yes

    后台存储过程中出错，是否停止保存操作

    默认开启

- 自动保存--配置到 conf 中

  ~~~
  save second changes
  ~~~

  - 满足限定时间内 key 变化数量达到指定数量即进行持久化
  - 参数
    - second 时间
    - changes key的变化量

### 6.2 AOF

- 配置

  ~~~
  appendonly yes|no
  ~~~

  是否开启 AOF 持久化功能，默认不开启

  ~~~
  appendsync always|everysec|no
  ~~~

  AOF 写数据策略

  ~~~
  appendfilename filename
  ~~~

  修改文件名

- AOF 重写

  - 降低磁盘占用率
  - 提高持久化效率，降低持久化写时间，提高性能
  - 降低数据恢复用时，提高数据恢复效率

  手动重写

  ~~~
  bgrewriteaof
  ~~~

  自动重写

  ~~~
  auto-aof-rewrite-min-size size
  auto-aof-rewrite-percentage
  ~~~

![1590151406205](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-aof-重写流程.png)

![1590071512171](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-rdbVaof.png)

![1590072157761](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-RDB&AOF.png)

![1590072703250](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-应用场景.png)

持久化-零时的数据建议持久化，比如5，6，7，13这种

## 7. 事务

~~~
multi
~~~

​	开启事务

~~~
exec
~~~

​	执行事务并退出当前

~~~discard
discard
~~~

​	取消事务

![1590151209748](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-事务流程.png)

### ![1590151588453](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-事务注意事项.png)7.2 锁

~~~
watch key1 key2
~~~

​	在执行exec之前如果key发生变化，终止事务执行

~~~
unwatch
~~~

​	取消监视

![1590152045077](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-分布式锁.png)

### 死锁

~~~
expire lock-key second
pexpire lock-key milliseconds
~~~

![1590496199049](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-分布式锁改良.png)

### 数据删除策略

 

![1590496465425](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-时效数据.png)

![1590496595269](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-定时删除.png)

![1590496687612](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-惰性删除.png)

![1590496924072](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-定期删除.png)

![1590497065602](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-定期删除特点.png)

![1590497086001](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\删除策略对比.png)

### 逐出算法

![1590497239340](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\逐出算法配置.png)

相关配置

![1590497440498](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-相关配置.png)

### redis-基本配置

![1590497743736](C:\Users\tyrant\AppData\Roaming\Typora\typora-user-images\1590497743736.png)

![1590497784673](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-配置2.png)

![1590497844515](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\redis-配置3.png)

![1590497895005](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\reids-配置4.png)

### Bitmaps

![1590498333831](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\Bitmaps-基本操作.png)

![1590498572226](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\bitmaps-扩展操作.png)

### HyperLogLog

 

![1590498832562](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\HyperLogLog-应用场景png)

![1590498904073](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\HyperLogLog-基本操作.png)

![1590499063415](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\HyperLogLog-注意事项.png)

### GEO

![1590499213718](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\GEO-基本操作.png)

![1590499319223](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\GEO-基本操作2.png)

## 主从复制

![1590499891865](E:\Java\new_Java_Study\daily_notes\pictures\学习笔记\redis\单机redis问题.png)