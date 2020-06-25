# 前置知识点

# epoll

在linux中，一个客户端的连接是需要经过kernel。

- 在早期的时候，一个客户端连接进来，就会生成一个文件描述符，然后系统就会起一个线程调用系统的read方法读取这个文件描述符，这里是同步阻塞；
- 后来，起一个线程轮询访问每个文件描述符，看哪个文件描述符有数据需要读取，轮询发生在用户态，这里是同步非阻塞；
- 再后来，调用kernal的select方法，将轮询文件描述符交给kernal处理，如果有描述符有数据，则将这个描述符传给线程，在调用read方法读取数据
- epoll，线程调用epoll_create创建一个epoll对象epfd，再通过调用epoll_ctl将需要监视的Socket添加到epfd中，最后调用epoll_wait阻塞等待数据。利用mmap(内存映射)在内核和用户空间形成一个共享空间，空间中包含一个红黑树和链表，红黑树负责存放文件描述符，链表存放有数据的文件描述符（这两部都是内核操作），这样就避免了内核和用户空间之间不断地复制。链表中有数据后，wait就会返回，用户线程就可以拿去文件描述符进行读取。

# Redis知识总结

> ### redis是二进制安全的，是按照字节数组的形式存储的
>
> ### redis-cli --raw启动客户端，获取值会按照编码格式进行显示，而不是显示ASCII码
>
> ### redis中是分为16库的，每个库都是相互隔离的，如果选中2号库，就无法访问其他库中的内容
>
> ### 每一个key中都是包含了类型用type key查看操作类型，其中包含了一个属性encoding，记录了value的类型

## 命令学习方式

![](D:\0_LeargingSummary\Redis\images\redis-cli_help命令.png)

# String命令操作

![](D:\0_LeargingSummary\Redis\images\String类型的命令操作.png)



## set

![](D:\0_LeargingSummary\Redis\images\set.png)

## mset批量设置

![](D:\0_LeargingSummary\Redis\images\mset多值插入.png)

## append追加

![](D:\0_LeargingSummary\Redis\images\append.png)

## setrange指定下标设值

![](D:\0_LeargingSummary\Redis\images\setrange.png)

```shell
127.0.0.1:6379> SETRANGE k1 3 2
(integer) 7
127.0.0.1:6379> get k1
"2da2adf"
```

## strlen获取值的长度

![](D:\0_LeargingSummary\Redis\images\strlen.png)

## getrange获取指定下标范围内值

有正向索引还有反向索引

![](D:\0_LeargingSummary\Redis\images\getrange.png)

## Type获取key的类型

> key包含type

![](D:\0_LeargingSummary\Redis\images\type.png)

## Object获取key对应值的类型

> key中包含encoding

![](D:\0_LeargingSummary\Redis\images\object检查redis对象.png)

## String数值的操作

![](D:\0_LeargingSummary\Redis\images\String数值的操作.png)

## getset获取设置

![](D:\0_LeargingSummary\Redis\images\getset.png)

## setbit

**bitmap位图示意**

![](D:\0_LeargingSummary\Redis\images\对二进制的操作.png)

![](D:\0_LeargingSummary\Redis\images\setbit.png)

## bitpos

![](D:\0_LeargingSummary\Redis\images\bitpos.png)

## bitcount统计二进制1出现的次数

![](D:\0_LeargingSummary\Redis\images\bitcount.png)

## bitop二进制计算

![](D:\0_LeargingSummary\Redis\images\bitop.png)

> ### 注：使用bitmap操作，典型应用：统计用户登录天数，比如统计1年，就使用365位二进制，第几天登录了就标为1

# List操作命令

![](D:\0_LeargingSummary\Redis\images\list操作命令.png)

## LPush\LRange\Lpop

![](D:\0_LeargingSummary\Redis\images\LPush.png)

```shell
127.0.0.1:6379> LPUSH k3 1 2 3 4 5 6
127.0.0.1:6379> LRANGE k3 0 -1  ##栈结构，先push进去的后出来
1) "6"
2) "5"
3) "4"
4) "3"
5) "2"
6) "1"
127.0.0.1:6379> LPOP k3
"6"
```

## LIndex\Lset

![](D:\0_LeargingSummary\Redis\images\LIndex-LSet.png)

```shell
127.0.0.1:6379> LINDEX k3 1
"4"
127.0.0.1:6379> LSET k3 4 xxoo
OK
127.0.0.1:6379> LRANGE k3 0 -1
1) "5"
2) "4"
3) "3"
4) "2"
5) "xxoo"
```

## LRem移除指定个数的元素

![](D:\0_LeargingSummary\Redis\images\LRem.png)

## LInsert

![](D:\0_LeargingSummary\Redis\images\LInsert.png)

## BLPOP阻塞获取

![](D:\0_LeargingSummary\Redis\images\BLpop.png)

## LTrim

![](D:\0_LeargingSummary\Redis\images\LTrim.png)

# Hash

> ### 类似于hashmap

![](D:\0_LeargingSummary\Redis\images\hash.png)

## hset\hmset\hget\hmget\hkeys\hvals\hgetall

![](D:\0_LeargingSummary\Redis\images\hash一系列操作.png)

```shell
127.0.0.1:6379> HSET person name diao
(integer) 1
127.0.0.1:6379> HGET person name
"diao"
127.0.0.1:6379> HMSET person name diao age 23
OK
127.0.0.1:6379> HMGET person name age #获得多个字段
1) "diao"
2) "23"
127.0.0.1:6379> HGETALL person #直接获得所有
1) "name"
2) "diao"
3) "age"
4) "23"
```

## HIncrByFloat

![](D:\0_LeargingSummary\Redis\images\HIncrByFloat.png)

```shell
127.0.0.1:6379> HINCRBYFLOAT person age 2.3 #增长其中一个字段
"25.3"
```

# Set

> 无序不重复集合

![](D:\0_LeargingSummary\Redis\images\set操作命令.png)

## sadd\sinter\sinterstore\smembers\sunion\sdiff

![](D:\0_LeargingSummary\Redis\images\set一系列操作.png)

```shell
127.0.0.1:6379> SADD k1  4 3 2 1 5 4 3 2 1 #添加集合元素
(integer) 5
127.0.0.1:6379> SMEMBERS k1 #获得set集合中所有的成员
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"

```

## SRandMember\Spop

![](D:\0_LeargingSummary\Redis\images\SRandMember-spop.png)

```shell
127.0.0.1:6379> SRANDMEMBER k1 6 #正数：取出一个去重的结果集（不能超过已有集）
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
127.0.0.1:6379> SRANDMEMBER k1 -6 #负数：取出一个带重复的结果集，一定满足你要的数量  如果：0，不返回
1) "3"
2) "1"
3) "4"
4) "2"
5) "2"
6) "4"
```

# sorted_set

![](D:\0_LeargingSummary\Redis\images\sort_set操作命令.png)

## ZAdd\ZRange\ZRevRange\

![](D:\0_LeargingSummary\Redis\images\sorted_set一系列操作.png)

```shell
127.0.0.1:6379> ZADD k1 1 apple 5 orange 2 banana
(integer) 3
127.0.0.1:6379> ZRANGE k1 0 -1
1) "apple"
2) "banana"
3) "orange"
127.0.0.1:6379> ZRANGE k1 0 -1 withscores
1) "apple"
2) "1"
3) "banana"
4) "2"
5) "orange"
6) "5"
127.0.0.1:6379> ZREVRANGE k1 0 -1 withscores
1) "orange"
2) "5"
3) "banana"
4) "2"
5) "apple"
6) "1"
```

## ZScore\ZRank\ZRange\ZIncrBy\

![](D:\0_LeargingSummary\Redis\images\ZScore.png)

# 管道

```shell
echo -e 启动对反斜杠的转义功能，nc建立一个redis服务的socket连接，可以一次发送多条命令
[root@node1 diao]# echo -e "set k2 yuhang \n set k3 99 \n incr k3 | nc localhost 6379
```

```shell
[root@node1 ~]# nc localhost 6379 #建立一个与redis的socket连接
set k1 1
+OK
[root@node1 ~]# echo -e "set k1 2 \n get k1 \n incr k1"|nc localhost 6379 #利用管道符将命令发送到redis中
+OK
$1
2
:3
```

# PUB\SUB

![](D:\0_LeargingSummary\Redis\images\pub-sub.png)

```shell
127.0.0.1:6380> SUBSCRIBE ooxx  #无法收到订阅之前的消息
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "ooxx"
3) (integer) 1
1) "message"
2) "ooxx"
3) "hello"
127.0.0.1:6380> PUBLISH ooxx hello #相对应的管道发布消息
(integer) 1
```

# 事务

![](D:\0_LeargingSummary\Redis\images\事务.png)

```shell
127.0.0.1:6380> MULTI #开始一个事务
OK
127.0.0.1:6380> set k3 3 #将这个命令加入到对应的队列中，并未执行
QUEUED
127.0.0.1:6380> EXEC #执行队列中的命令  watch一个key时，如果这个key发生改变，则这个事务执行失败
1) OK
```

# 布隆过滤器

https://github.com/RedisBloom/RedisBloom

下载完成后，解压，make

启动redis服务的时候加载布隆过滤器

```
方式1、redis-server --loadmodule /path/to/redisbloom.so
方式2、修改配置文件，添加 loadmodule /opt/modules/RedisBloom-master/redisbloom.so
```

```shell
bf.add 添加元素到布隆过滤器
  bf.exists 判断元素是否在布隆过滤器
  bf.madd 添加多个元素到布隆过滤器，bf.add只能添加一个
  bf.mexists 判断多个元素是否在布隆过滤器
  
127.0.0.1:6379> help BF.ADD
BF.ADD key ...options...
summary: Help not available
since: not known
group: generic

```

> ### 利用Redis的BitMap实现布隆过滤器的底层映射(以二进制的方式存储)。
>
> 当布隆过滤器返回不存在时，那就一定不存在；返回存在时，极小概率可能是不存在的
>
> 如果穿透了，不存在，采取措施：在client，增加redis中的key，value标记。
>
> 数据库增加了元素，对bloom的添加

# 作为缓存和数据库的区别

- 缓存的数据不是那么的重要
- 不是全量数据
- 缓存的数据应该随着访问变化
- 保存热数据

> 问题：redis中的数据怎么才能随着业务的变化，只保留热数据，由于内存有限，随意存储有瓶颈

# key有效期

```shell
127.0.0.1:6379> help EXPIRE #设置有效时长

  EXPIRE key seconds
  summary: Set a key's time to live in seconds
  since: 1.0.0
  group: generic
  
127.0.0.1:6379> help EXPIREAT #设置某个时间点过期

  EXPIREAT key timestamp
  summary: Set the expiration for a key as a UNIX timestamp
  since: 1.2.0
  group: generic
```

- key的有效时长不会随着访问延长
- 如果该key发生写操作，会剔除过期时间
- 如果设置是时间点过期，则key不能延长时间

> ### 注：Redis如何淘汰过期的keys
>
> 有两种方式：被动和主动的方式
>
> 当一些客户端尝试访问的时候，key会被发现并主动的过期。
>
> 但是光这样是不行的，有些过期的key是永远访问不到的，所以定时随机测试设置key的过期时间，所有这些过期keys将会删除。
>
> 具体做法：
>
> 1. 测试随机的20个keys进行相关过期测试；
> 2. 删除所有已过期的keys;
> 3. 如果有多于25%的keys过期，重复步骤1

# Redis回收策略

Maxmemory配置指令

`maxmemory`配置指令用于配置Redis存储数据时指定限制的内存大小

```shell
volatile-lru -> Evict using approximated LRU among the keys with an expire set.
allkeys-lru -> Evict any key using approximated LRU.
volatile-lfu -> Evict using approximated LFU among the keys with an expire set.
allkeys-lfu -> Evict any key using approximated LFU.
# lfu最少使用
# lru最久未使用
```

# CAP

一致性（c）:在分布式系统中所有的数据备份，在同一时刻是否同样的值。

可用性（A）：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。

分区容忍性（P）：系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。

CAP原则的精髓就是要么AP，要么CP，要么AC，但是不存在CAP。

# RDB/AOF

- RDB将数据存储到磁盘中，redis创建一个子进程对数据进行存储，为了保证存储数据的时点性准确，这里用到下面的linux知识点。

save命令触发阻塞的数据存储；

bgsave命令是非阻塞的其实就是fork创建子进程；

```shell
################################ SNAPSHOTTING  ################################
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
save 900 1
save 300 10 #当达到300秒，操作10次，发生save
save 60 10000 #当时间达到60秒，操作有10000，发生save

# The filename where to dump the DB
dbfilename dump.rdb

# Note that you must specify a directory here, not a file name.
dir /var/lib/redis/6379
```

弊端：不支持拉链，只有一个dump.rdb；

 丢失数据相对多一些，时间点与时间点之间的数据。

优点：恢复的速度相对快

- AOF：将Redis的写操作记录到文件中；

  优点：丢失数据少；

  RDB和AOF可以同时开启，但是如果开启AOF，就只会用AOF恢复；

  4.0之后AOF文件中包含RDB全量，后面增加记录新的的写操作；

弊端：体量无线变大，恢复慢

```shell
######################### APPEND ONLY MODE ##########################
appendonly no #开启aof

# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof" 

# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.
# appendfsync always
appendfsync everysec
# appendfsync no

# When rewriting the AOF file, Redis is able to use an RDB preamble in the
# AOF file for faster rewrites and recoveries. When this option is turned
# on the rewritten AOF file is composed of two different stanzas:
#
#   [RDB file][AOF tail]
#
# When loading Redis recognizes that the AOF file starts with the "REDIS"
# string and loads the prefixed RDB file, and continues loading the AOF
# tail.
aof-use-rdb-preamble yes ##是否开启RDB和AOF混合
```

BGREWRITEAOF重写aof文件，合并删除文件中重复的操作命令；

## Linux的补充知识点

### 管道会触发创建子进程

`$$`获得当前进程号，但是优先级高于管道符`|`,会在管道之前触发

```shell
echo $$|more
echo $BASHPID|more
```

![](D:\0_LeargingSummary\Redis\images\管道出发子进程.png)

正常情况下，进程之间的数据是隔离的。

但是，父进程可以让子进程看到数据！export的环境变量，子进程的修改不会影响到父进程，同样父进程对变量的修改也是不会影响的子进程。

### fork系统调用

- linux系统调用fork(),速度快，空间小

创建子进程并不会发生数据的复制，将父进程中虚拟地址指针复制了一份；

fork中使用的是copy-on-write机制写时复制，如果父进程发生修改操作，原内存地址地址空间的数据是不会变化的，复制出一份数据进行修改，然后将原指针指向新地址，这样就不会影响到子进程的操作；

Redis中进行bgsave的时候就是调用fork创建子进程，子进程进行数据保存RDB；

# AKF拆分原则

单节点问题：单点故障、容量有限、压力

从坐标轴三个方向x,y,z进行诠释；

- X：全量镜像，主从复制，可以解决单点故障的问题，client修改增加访问主节点，读取访问从结点，稍微减轻压力问题；
- Y：根据业务功能进行拆分，不同的业务访问不同的主节点，从而可以减轻访问的压力；
- Z：进行逻辑上的拆分，在满足条件的情况下，将数据存到不同的结点上，从而解决容量有限的问题；

![](D:\0_LeargingSummary\Redis\images\AKF拆分.png)

> ## Redis默认采用的异步复制，其特点是低延迟和高性能
>
> ## 当采用哨兵监控保证高可用的时候，哨兵数量的最好保证奇数个，其一就是能成功做出决策，其二就是降低故障风险；

# 主从

REPLICAOF host port 追随主节点；

当一个从结点追随一个主节点，主节点是可以得知有哪些从结点；因此当哨兵监控主节点的时候，是可以得知有哪些从结点；

> ## 哨兵的之间的通过发布订阅发现其他哨兵节点；

# 哨兵

用来监控主节点，保证redis的高可用

启动方式：redis-sentienl 或者 redis-server --sentinel

# 数据划分

1. hash+取模，弊端：取模的数是固定的，会影响分布式下的扩展
2. random随机分配，弊端：客户端1存进去数据，无法得知存到哪台服务
3. 一致性hash，虚拟出一个环形，服务器结点经过hash算法位于环上的一点，然后操作的key同样也经过hash算法，找到距离最近的服务结点。优点：加节点可以分担其它结点的压力，不会造成全局洗牌；缺点：新增结点造成部分数据无法命中，造成击穿，访问压到了mysql；解决方案：访问最近的两个结点。

# 击穿/穿透/雪崩

击穿：当redis缓存中的key失效了，同时又有并发进来了，发现redis中没有该key，就会到数据库中查找；

解决法方案：redis是单线程实例，当第一个连接发现没有key时，就setnx(不存在时设置成功)设置锁，获得锁的连接才能到数据库进行查询，从数据库中查询成功后，重新设置redis的key，其它并发的连接获取不到锁就短暂的睡眠，醒来后再去重新获取key。



穿透：查询的是系统中根本不存在的数据。布隆过滤器

雪崩：大量的key同时失效，造成大量的访问到达数据库。随机过期时间

# 分布式锁：redis

- setnx
- 过期时间
- 多线程（守护线程）延长过期时间