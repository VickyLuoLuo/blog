---
title: 高并发利器Redis
header-img: /img/header_img/post-bg-desk.jpg
catalog: true
top:
tags:
  - redis
categories:
  - work
abbrlink: a10a
subtitle:
---



### 一. 什么是Redis

​		官方简介：Redis是一个基于BSD**开源**的项目，是一个把**结构化**的数据放在**内存**中的一个存储系统，可以把它**作为数据库，缓存和消息中间件**来使用。同时支持**strings，lists，hashes，sets，sorted sets，**bitmaps，hyperloglogs、geospatial indexes和streams等**数据类型**。还内建了**复制、lua脚本、LRU、事务**、不同级别的**持久化**功能，通过redis sentinel实现**高可用**，通过redis cluster实现了**自动分片**，以及**发布/订阅**，**自动故障转移**等等特性。



### 二. Redis能解决什么问题

> **假如我们有个查询列表的API，用户抱怨说每次请求都要2秒左右才能返回结果，如何改善用户体验呢？**

​		**方案一、基于HTTP缓存**：

​		为API的响应头加上缓存控制 **cache-control**:max-age=600 ，即在浏览器缓存这个响应10分钟，简单粗暴。但是这个方法有两个缺点：第一个是在缓存生效的10分钟内，API消费者可能会得到旧的数据；第二个是如果客户端浏览器不使用缓存，方法直接无效。

​		**方案二、基于本机内存的缓存**

​		该API请求耗时操作主要在于使用SQL查询结果的过程中消耗了将近2秒的时间，于是，我们又想到了一个方案，把SQL查询的结果直接缓存在当前API服务器的内存中，比如使用JDK自带的**HashMap**和ConcurrentHashMap，或者**Guava Cache**、Spring Cache等本地缓存框架，设置缓存有效时间为10分钟，并且在修改和更新操作时同步修改缓存中的数据。这样后续10分钟内的请求直接读缓存，可以做到及时响应，不再花费2秒去执行SQL了。

​		结果其他API的小伙伴发现这是个好办法，于是很快我们就发现API服务器的内存要爆了...而且对于分布式架构，相同的服务部署在多台机器上的时候，各个服务之间的缓存是无法共享的。

​		**方案三、使用Redis做缓存**

​		要解决API服务器内存被缓存塞满及各服务间缓存无法共享的问题，最直接的办法就是把这些缓存部署在一台单独的服务器上，即使同一个相同的服务部署在再多机器上，也是使用的同一份缓存。于是我们需要为分布式缓存引入额外的服务，比如 Redis 或Memcached，而且要保证服务的高可用。

#### Redis主要应用场景

​		缓存（数据查询、短连接、新闻内容、商品内容等）、分布式会话（Session）、任务队列（秒杀、抢购、12306等）、应用排行榜、访问统计、数据过期处理（可以精确到毫秒）。

### 三. Redis 和 Memcached 的区别和共同点

- **共同点** ：

1. 都是基于内存的缓存。
2. 都有过期策略。
3. 两者的性能都非常高。

- **区别** ：

1. **Redis 支持更丰富的数据类型（支持更复杂的应用场景）**。Redis 不仅仅支持简单的 k/v 类型的数据，同时还提供 list，set，zset，hash 等数据结构的存储。Memcached 只支持最简单的 k/v 数据类型。
2. **Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用，而 Memecache 把数据全部存在内存之中。**
3. **Redis 有灾难恢复机制。** 因为可以把缓存中的数据持久化到磁盘上。
4. **Redis 在服务器内存使用完之后，可以将不用的数据放到磁盘上。但是，Memcached 在服务器内存使用完之后，就会直接报异常。**
5. **Memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据；但是 Redis 目前是原生支持 cluster 模式的。**
6. **Memcached 是多线程，非阻塞 IO 复用的网络模型；Redis 使用单线程的多路 IO 复用模型。** （Redis 6.0 引入了多线程 IO ）
7. **Redis 支持发布订阅模型、Lua脚本、事务等功能，而Memcached不支持。并且，Redis支持更多的编程语言。**



### 四. 安装和配置

#### 1. 安装：略

#### 2. 主要配置：

​	Linux下配置文件为redis.conf，Windows下配置文件为redis.windows.conf

- 开启持久化，配置文件装中添加如下内容：

    ```
    appendonly yes
    ```

- port：端口配置项，查看和设置Redis监听端口，默认端口为6379。
- bind：主机地址配置项，查看和绑定的主机地址，默认地址的值为127.0.0.1。这个选项，在单网卡的机器上，一般不需要修改。

- timeout：连接空闲多长要关闭连接，表示客户端闲置一段时间后，要关闭连接。如果指定为0，表示时长不限制。这个选项的默认值为0，表示默认不限制连接的空闲时长

- dbfilename：指定保存缓存数据库的本地文件名，默认值为dump.rdb。

- dir：指定保存缓存数据的本地文件所存放的目录，默认值为安装目录。

- rdbcompression：指定存储至本地数据库时是否压缩数据，默认为yes,Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变得巨大。

- save：指定在多长时间内，有多少次Key-Value更新操作，就将数据同步到本地数据库文件，可以设置多个条件。save配置项的格式为save<seconds><changes>:seconds表示时间段的长度，changes表示变化的次数。如设置为900秒（15分钟）内有1个更改，则同步到文件：

    ```
    127.0.0.1:6379> config set save "900 1"
    OK
    127.0.0.1:6379> config get save
    1) "save"
    2) "900 1"
    ```

- requirepass：设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认这个选项是关闭的。

- slaveof：在主从复制的模式下，设置当前节点为slave（从）节点时，设置master（主）节点的IP地址及端口，在Redis启动时，它会自动从master（主）节点进行数据同步。如果已经是slave（从）服务器，则会丢掉旧数据集，从新的master主服务器同步缓存数据。格式为：

    ```
    slaveof<masterip><masterport>
    ```

- masterauth：在主从复制的模式下，当master（主）服务器节点设置了密码保护时，slave（从）服务器连接master（主）服务器的密码。格式为:

    ```
    masterauth<master-password>
    ```

- databases：设置缓存数据库的数量，默认数据库数量为16个。这16个数据库的id为0-15，默认使用的数据库是第0个。可以使用SELECT <dbid>命令在连接时通过数据库id来指定要使用的数据库。

#### 3. Redis连接客户端命令：

```
#redis-cli -h host -p port -a password
```

本地连接：

```
root@0912b31c4171# redis-cli
```



### 五. Redis的主要特点

#### 1. 速度异常快 

​		采用多路 IO 复用模型，不需要等待磁盘的IO，在内存之间进行的数据存储和查询，速度非常快。官方数据显示每秒可执行大约 110000 次的设置(SET)操作，每秒大约可执行 81000 次的读取/获取(GET)操作。可以运行命令 `redis-benchmark -n 100000 -q` 来检测本地同时执行 10 万个请求时的性能：

```
root@0912b31c4171:/data# redis-benchmark -n 100000 -q
PING_INLINE: 31172.07 requests per second
PING_BULK: 31615.55 requests per second
SET: 30432.14 requests per second
GET: 31289.11 requests per second
INCR: 30441.40 requests per second
LPUSH: 29550.83 requests per second
RPUSH: 30184.12 requests per second
```

#### 2. 丰富的数据结构 

​		除了string之外，还有list、hash、set、sortedset、bitmaps、hyperloglogs、geospatial indexes和stream。其中**string(字符串)**、**list(列表)**、**hash(字典)**、**set(集合)** 和 **zset(有序集合)**这 5 种是 Redis 最基础、最重要的部分。

#### 3. 单线程

​		避免了频繁的上下文切换。Redis 内部使用文件事件处理器 `file event handler`，这个文件事件处理器是单线程的，所以 Redis 才叫做单线程的模型。它采用 IO 多路复用机制同时监听多个 socket，根据 socket 上的事件来选择对应的事件处理器进行处理。

文件事件处理器的结构包含 4 个部分：

- 多个 socket

- IO 多路复用程序

- 文件事件分派器

- 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

    ​		多个 socket 可能会并发产生不同的操作，每个操作对应不同的文件事件，但是 IO 多路复用程序会监听多个 socket，会将 socket 产生的事件放入队列中排队，事件分派器每次从队列中取出一个事件，把该事件交给对应的事件处理器进行处理。

    ***传统的并发模型，每个 I/O 流都有一个新的线程管理***

     ***I/O 多路复用，只有单个线程，通过跟踪每个 I/O 流的状态，来管理多个 I/O 流。***

#### 4. 可持久化 

​		支持RDB与AOF两种方式，将内存中的数据写入外部的物理存储设备。

​		**RDB**(Redis DataBase)：是最简单的 Redis 持久性模式。当满足特定条件时，它将生成数据集的时间点**快照**，例如，如果先前的快照是在2分钟前创建的，并且现在已经至少有 *100* 次新写入，则将创建一个新的快照。此条件可以由用户配置 Redis 实例来控制，也可以在运行时修改而无需重新启动服务器。快照作为包含整个数据集的单个 `.rdb` 文件生成。

​		**AOF** (Append Only File - 仅追加文件)：它的工作方式非常简单：每次执行 **修改内存** 中数据集的写操作时，都会 **记录** 该操作。假设 AOF 日志记录了自 Redis 实例创建以来 **所有的修改性指令序列**，那么就可以通过对一个空的 Redis 实例 **顺序执行所有的指令**，也就是 **「重放」**，来恢复 Redis 当前实例的内存数据结构的状态。

​		**RDB优势**

- RDB文件紧凑，全量备份，非常适合用于进行备份和灾难恢复。

- 生成RDB文件的时候，redis主进程会fork()一个子进程来处理所有保存工作，主进程不需要进行任何磁盘IO操作。

- RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。

​		**RDB劣势**

- RDB快照是一次全量备份，存储的是内存数据的二进制序列化形式，存储上非常紧凑。当进行快照持久化时，会开启一个子进程专门负责快照持久化，子进程会拥有父进程的内存数据，父进程修改内存子进程不会反应出来，所以在快照持久化期间修改的数据不会被保存，可能丢失数据。

​		**AOF优势**

- AOF可以更好的保护数据不丢失，一般AOF会每隔1秒，通过一个后台线程执行一次fsync操作，最多丢失1秒钟的数据。（2）AOF日志文件没有任何磁盘寻址的开销，写入性能非常高，文件不容易破损。

- AOF日志文件即使过大的时候，出现后台重写操作，也不会影响客户端的读写。

- AOF日志文件的命令通过非常可读的方式进行记录，这个特性非常适合做灾难性的误删除的紧急恢复。比如某人不小心用flushall命令清空了所有数据，只要这个时候后台rewrite还没有发生，那么就可以立即拷贝AOF文件，将最后一条flushall命令给删了，然后再将该AOF文件放回去，就可以通过恢复机制，自动恢复所有数据

​		**AOF劣势**

- 对于同一份数据来说，AOF日志文件通常比RDB数据快照文件更大

- AOF开启后，支持的写QPS会比RDB支持的写QPS低，因为AOF一般会配置成每秒fsync一次日志文件，当然，每秒一次fsync，性能也还是很高的

- 以前AOF发生过bug，就是通过AOF记录的日志，进行数据恢复的时候，没有恢复一模一样的数据出来。

#### 5. 支持发布、订阅、管道。

​		**发布/ 订阅系统** 是 Web 系统中比较常用的一个功能。简单点说就是 **发布者发布消息，订阅者接受消息**，这有点类似于我们的报纸/ 杂志社之类的。我们可以使用redis的一个 `list` 列表结构结合 `lpush` 和 `rpop` 来实现消息队列的功能，但是似乎很难实现实现 **消息多播** 的功能：

![image](https://tvax2.sinaimg.cn/large/006YzKDNly1gopdit62wmj30yg0ccgmp.jpg)

​		**Redis**为了消除`Publisher` 与 `Consumer` 的强关联，支持消息多播，引入了另一种概念：**频道** *(channel)*：

![image](https://tvax1.sinaimg.cn/large/006YzKDNly1gopdjtvehij30yg0cbmy0.jpg)

当 `Publisher` 往 `channel` 中发布消息时，关注了指定 `channel` 的 `Consumer` 就能够同时受到消息。但这里的 **问题** 是，消费者订阅一个频道是必须 **明确指定频道名称** 的，这意味着，如果我们想要 **订阅多个** 频道，那么就必须 **显式地关注多个** 名称。

为了简化订阅的繁琐操作，**Redis** 提供了 **模式订阅** 的功能 **Pattern Subscribe**，这样就可以 **一次性关注多个频道** 了，即使生产者新增了同模式的频道，消费者也可以立即受到消息：

![image](https://tvax2.sinaimg.cn/large/006YzKDNly1gopdkll7gtj30yg0cb0ul.jpg)

例如上图中，**所有** 位于图片下方的 **`Consumer` 都能够受到消息**。

`Publisher` 往 `wmyskxz.chat` 这个 `channel` 中发送了一条消息，不仅仅关注了这个频道的 `Consumer 1` 和 `Consumer 2` 能够受到消息，图片中的两个 `channel` 都和模式 `wmyskxz.*` 匹配，所以 **Redis** 此时会同样发送消息给订阅了 `wmyskxz.*` 这个模式的 `Consumer 3` 和关注了在这个模式下的另一个频道 `wmyskxz.log` 下的 `Consumer 4` 和 `Consumer 5`。

另一方面，如果接收消息的频道是 `wmyskxz.chat`，那么 `Consumer 3` 也会受到消息。  

##### *快速体验*

在 **Redis** 中，**PubSub** 模块的使用非常简单，常用的命令也就下面这么几条：

```bash
# 订阅频道：
SUBSCRIBE channel [channel ....]   # 订阅给定的一个或多个频道的信息
PSUBSCRIBE pattern [pattern ....]  # 订阅一个或多个符合给定模式的频道
# 发布频道：
PUBLISH channel message  # 将消息发送到指定的频道
# 退订频道：
UNSUBSCRIBE [channel [channel ....]]   # 退订指定的频道
PUNSUBSCRIBE [pattern [pattern ....]]  #退订所有给定模式的频道
```

我们可以在本地快速地来体验一下 **PubSub**：

![](https://tvax2.sinaimg.cn/large/006YzKDNly1gopdlahl8sg30qq0cenpg.gif)

具体步骤如下：

1. 开启本地 Redis 服务，新建两个控制台窗口；
2. 在其中一个窗口输入 `SUBSCRIBE wmyskxz.chat` 关注 `wmyskxz.chat` 频道，让这个窗口成为 **消费者**。
3. 在另一个窗口输入 `PUBLISH wmyskxz.chat 'message'` 往这个频道发送消息，这个时候就会看到 **另一个窗口实时地出现** 了发送的测试消息。

#### 6. 支持分布式锁 

​		在分布式系统中，如果不同的节点需要访同到一个资源，往往需要通过互斥机制来防止彼此干扰，并且保证数据的一致性。在这种情况下，需要使用到分布式锁。分布式锁和Java的锁用于实现不同线程之间的同步访问，原理上是类似的。

#### 7.支持原子操作和事务

​		Redis事务是一组命令的集合。一个事务中的命令要么都执行，要么都不执行。如果命令在运行期间出现错误，不会自动回滚。

#### 8.支持主-从（Master-Slave）复制与高可用（Redis Sentinel）集群

​		3.0版本以上功能



### 六.常见数据结构及使用操作

*一般情况下是这样设计 key 的： `表名:列名:主键名:主键值`*

#### 1. string：

> 值可以是任何种类的字符串（包括二进制数据），例如你可以在一个键下保存一张 `.jpeg` 图片，需要注意不要超过 512 MB 。

- **设置和获取键值对**：

```
> SET key value
OK
> GET key
"value"
```

当 key 存在时，`SET` 命令会覆盖掉你上一次设置的值：

```
> SET key newValue
OK
> GET key
"newValue"
```

另外还可以使用 `EXISTS` 和 `DEL` 关键字来查询是否存在和删除键值对：

```
> EXISTS key
(integer) 1
> DEL key
(integer) 1
> GET key
(nil)
```

- **批量设置键值对**

```
> SET key1 value1
OK
> SET key2 value2
OK
> MGET key1 key2 key3    # 返回一个列表
1) "value1"
2) "value2"
3) (nil)
> MSET key1 value1 key2 value2
> MGET key1 key2
1) "value1"
2) "value2"
```

- **过期和 SET 命令扩展**

可以对 key 设置过期时间，到时间会被自动删除，这个功能常用来控制缓存的失效时间。*(过期可以是任意数据结构)*

```
> SET key value1
> GET key
"value1"
> EXPIRE name 5    # 5s 后过期
...                # 等待 5s
> GET key
(nil)
```

等价于 `SET` + `EXPIRE` 的 `SETEX` 命令：

```
> SETEX key 5 value1
...                # 等待 5s 后获取
> GET key
(nil)

> SETNX key value1  # 如果 key 不存在则 SET 成功
(integer) 1
> SETNX key value1  # 如果 key 存在则 SET 失败
(integer) 0
> GET key
"value"             # 没有改变 
```

- **计数**

如果 value 是一个整数，还可以对它使用 `INCR` 命令进行 **原子性** 的自增操作，这意味着及时多个客户端对同一个 key 进行操作，也决不会导致竞争的情况：

```
> SET counter 100
> INCR counter
(integer) 101
> INCRBY counter 50
(integer) 151
```

#### 2. list

> Redis 的列表相当于 Java 语言中的 **LinkedList**，注意它是链表而不是数组。这意味着 list 的插入和删除操作非常快，时间复杂度为 O(1)，但是索引定位很慢，时间复杂度为 O(n)。

**链表的基本操作**

- `LPUSH` 和 `RPUSH` 分别可以向 list 的左边（头部）和右边（尾部）添加一个新元素；
- `LRANGE` 命令可以从 list 中取出一定范围的元素；
- `LINDEX` 命令可以从 list 中取出指定下表的元素，相当于 Java 链表操作中的 `get(int index)` 操作；

示范：

```console
> rpush mylist A
(integer) 1
> rpush mylist B
(integer) 2
> lpush mylist first
(integer) 3
> lrange mylist 0 -1    # -1 表示倒数第一个元素, 这里表示从第一个元素到最后一个元素，即所有
1) "first"
2) "A"
3) "B"
```

- list 实现队列

队列是先进先出的数据结构，常用于消息排队和异步逻辑处理，它会确保元素的访问顺序：

```console
> RPUSH books python java golang
(integer) 3
> LPOP books
"python"
> LPOP books
"java"
> LPOP books
"golang"
> LPOP books
(nil)
```

- list 实现栈

栈是先进后出的数据结构，跟队列正好相反：

```console
> RPUSH books python java golang
> RPOP books
"golang"
> RPOP books
"java"
> RPOP books
"python"
> RPOP books
(nil)
```

- 使用场景举例：简单的消息队列、分页功能（lrange ）

#### 3. hash

> Redis 中的字典相当于 Java 中的 **HashMap**，内部实现也差不多类似，都是通过 **"数组 + 链表"** 的链地址法来解决部分 **哈希冲突**，同时这样的结构也吸收了两种不同数据结构的优点。

- 基本操作：

```
> HSET books java "think in java"    # 命令行的字符串如果包含空格则需要使用引号包裹
(integer) 1
> HSET books python "python cookbook"
(integer) 1
> HGETALL books    # key 和 value 间隔出现
1) "java"
2) "think in java"
3) "python"
4) "python cookbook"
> HGET books java
"think in java"
> HSET books java "head first java"  
(integer) 0        # 因为是更新操作，所以返回 0
> HMSET books java "effetive  java" python "learning python"    # 批量操作
OK
```

- 使用场景举例：单点登录。存储用户信息，以 CookieId 作为 Key，设置 30 分钟为缓存过期时间，能很好的模拟出类似 Session 的效果。

#### 4. set

> Redis 的集合相当于 Java 语言中的 **HashSet**，它内部的键值对是无序、唯一的。它的内部实现相当于一个特殊的字典，字典中所有的 value 都是一个值 NULL。

- 基本操作：

```
> SADD books java
(integer) 1
> SADD books java    # 重复
(integer) 0
> SADD books python golang
(integer) 2
> SMEMBERS books    # 注意顺序，set 是无序的 
1) "java"
2) "python"
3) "golang"
> SISMEMBER books java    # 查询某个 value 是否存在，相当于 contains
(integer) 1
> SCARD books    # 获取长度
(integer) 3
> SPOP books     # 弹出一个
"java"
```

- 使用场景举例：全局去重、计算共同喜好等

#### 5. zset

> 这可能使 Redis 最具特色的一个数据结构了，它类似于 Java 中 **SortedSet** 和 **HashMap** 的结合体，一方面它是一个 set，保证了内部 value 的唯一性，另一方面它可以为每个 value 赋予一个 score 值，用来代表排序的权重。它的内部实现用的是一种叫做 **「跳跃表」** 的数据结构。

- 基本操作

```
> ZADD books 9.0 "think in java"
> ZADD books 8.9 "java concurrency"
> ZADD books 8.6 "java cookbook"

> ZRANGE books 0 -1     # 按 score 排序列出，参数区间为排名范围
1) "java cookbook"
2) "java concurrency"
3) "think in java"

> ZREVRANGE books 0 -1  # 按 score 逆序列出，参数区间为排名范围
1) "think in java"
2) "java concurrency"
3) "java cookbook"

> ZCARD books           # 相当于 count()
(integer) 3

> ZSCORE books "java concurrency"   # 获取指定 value 的 score
"8.9000000000000004"                # 内部 score 使用 double 类型进行存储，所以存在小数点精度问题

> ZRANK books "java concurrency"    # 排名
(integer) 1

> ZRANGEBYSCORE books 0 8.91        # 根据分值区间遍历 zset
1) "java cookbook"
2) "java concurrency"

> ZRANGEBYSCORE books -inf 8.91 withscores  # 根据分值区间 (-∞, 8.91] 遍历 zset，同时返回分值。inf 代表 infinite。
1) "java cookbook"
2) "8.5999999999999996"
3) "java concurrency"
4) "8.9000000000000004"

> ZREM books "java concurrency"             # 删除 value
(integer) 1
> ZRANGE books 0 -1
1) "java cookbook"
2) "think in java"
```

- 使用场景举例：排行榜应用取 TOP N 、范围查找

### 七. Redis常见问题及解决方法

#### 1. 缓存与数据库双写不一致

##### 什么是双写不一致？

​		一般情况下我们都是这样使用缓存的：先读缓存，缓存没有的话，就读数据库，然后取出数据后放入缓存，同时返回响应。如果仅仅查询的话，缓存的数据和数据库的数据是没问题的。但是当要执行数据的更新操作的时候，数据库和缓存的数据就会出现不一致的情况。

从理论上说，只要我们设置了**键的过期时间**，就能保证缓存和数据库的数据**最终一致**。

##### 有哪些解决办法？

- 先更新数据库，再删缓存。

    如果在高并发的场景下，出现数据库与缓存数据不一致的**概率特别低**，也不是没有：

    > 1、 缓存刚好失效
    > 2、线程A查询数据库，得一个旧值
    > 3、线程B将新值写入数据库
    > 4、线程B删除缓存
    > 5、线程A将查到的旧值写入缓存

- 先删除缓存，再更新数据库。

    并发场景下分析一下，还是有问题：

    > 线程A删除了缓存
    > 线程B查询，发现缓存已不存在
    > 线程B去数据库查询得到旧值
    > 线程B将旧值写入缓存
    > 线程A将新值写入数据库

- 将删除缓存、修改数据库、读取缓存等的操作积压到队列里边，实现串行化。

这些方案从根本上来说，只能降低不一致发生的概率，无法完全避免。因此，**有强一致性要求的数据，不能放缓存。**

#### 2. 缓存穿透

##### 什么是缓存穿透？

​		缓存穿透说简单点就是大量请求的 key 根本不存在于缓存中，导致请求直接到了数据库上，根本没有经过缓存这一层。举个例子：我们有一张数据库表，ID都是从1开始的，但是可能有黑客想把我们的数据库搞垮，每次请求的ID都是负数，导致缓存失去意义，请求都会去找数据库。

##### 有哪些解决办法？

1. 缓存无效 key

    ​		如果缓存和数据库都查不到某个 key 的数据就写一个到 Redis 中去并设置过期时间，具体命令如下：`SET key value EX 10086`。这种方式可以解决请求的 key 变化不频繁的情况，如果黑客恶意攻击，每次构建不同的请求 key，会导致 Redis 中缓存大量无效的 key 。很明显，这种方案并不能从根本上解决此问题。如果非要用这种方式来解决穿透问题的话，尽量将无效的 key 的过期时间设置短一点比如 1 分钟。如果用 Java 代码展示的话，差不多是下面这样的：

```java
public Object getObjectInclNullById(Integer id) {
    // 从缓存中获取数据
    Object cacheValue = cache.get(id);
    // 缓存为空
    if (cacheValue == null) {
        // 从数据库中获取
        Object storageValue = storage.get(key);
        // 缓存空对象
        cache.set(key, storageValue);
        // 如果存储数据为空，需要设置一个过期时间(300秒)
        if (storageValue == null) {
            // 必须设置过期时间，否则有被攻击的风险
            cache.expire(key, 60 * 5);
        }
        return storageValue;
    }
    return cacheValue;
}
```

2. 布隆过滤器

    ​		布隆过滤器是一个非常神奇的数据结构，通过它我们可以非常方便地判断一个给定数据是否存在于海量数据中，我们需要的就是判断 key 是否合法。

    ​		具体做法：内部维护一系列合法有效的 Key都存放在布隆过滤器中，当用户请求过来，先判断用户发来的请求的值是否存在于布隆过滤器中。不存在的话，直接返回请求参数错误信息给客户端，存在的话才会走下面的流程。

    ​		但是，需要注意的是布隆过滤器可能会存在误判的情况。总结来说就是： **布隆过滤器说某个元素存在，小概率会误判。布隆过滤器说某个元素不在，那么这个元素一定不在。**

    ****

    ***布隆过滤器原理*：**  

    **当一个元素加入布隆过滤器中的时候，会进行如下操作：**

    1. 使用布隆过滤器中的哈希函数对元素值进行计算，得到哈希值（有几个哈希函数得到几个哈希值）。

    2. 根据得到的哈希值，在位数组中把对应下标的值置为 1。

    **当我们需要判断一个元素是否存在于布隆过滤器的时候，会进行如下操作：**

    1. 对给定元素再次进行相同的哈希计算；

    2. 得到值之后判断位数组中的每个元素是否都为 1，如果值都为 1，那么说明这个值在布隆过滤器中，如果存在一个值不为 1，说明该元素不在布隆过滤器中。

    然后，一定会出现这样一种情况：**不同的字符串可能哈希出来的位置相同。** （可以适当增加位数组大小或者调整我们的哈希函数来降低概率）

#### 3. 缓存雪崩

##### 什么是缓存雪崩？

​		缓存雪崩描述的就是这样一个简单的场景：**缓存在同一时间大面积的失效，后面的请求都直接落到了数据库上，造成数据库短时间内承受大量请求。** 这就好比雪崩一样，数据库的压力可想而知，可能直接就挂了。

​		举个例子：系统的缓存模块出了问题比如宕机导致不可用，造成系统的所有访问，都要走数据库。

​		还有一种缓存雪崩的场景是：**有一些被大量访问数据（热点缓存）在某一时刻大面积失效，导致对应的请求直接落到了数据库上。**

​		举个例子 ：秒杀开始12个小时之前，我们统一存放了一批商品到 Redis 中，设置的缓存过期时间也是12个小时，那么秒杀开始的时候，这些秒杀的商品的访问直接就失效了。导致的情况就是，相应的请求直接就落到了数据库上，就像雪崩一样可怕。

##### 有哪些解决办法？

Redis服务不可用：

1. 采用Redis集群，避免单机出现问题整个缓存服务都没办法使用。
2. 限流，避免同时处理大量的请求。

热点缓存失效：

1. 设置不同的失效时间，比如随机设置缓存的失效时间。
2. 缓存永不失效。

