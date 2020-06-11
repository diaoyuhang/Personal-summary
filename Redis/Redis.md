# 前置知识点

# epoll

在linux中，一个客户端的连接是需要经过kernel。

- 在早期的时候，一个客户端连接进来，就会生成一个文件描述符，然后系统就会起一个线程调用系统的read方法读取这个文件描述符，这里是同步阻塞；
- 后来，起一个线程轮询访问每个文件描述符，看哪个文件描述符有数据需要读取，轮询发生在用户态，这里是同步非阻塞；
- 再后来，调用kernal的select方法，将轮询文件描述符交给kernal处理，如果有描述符有数据，则将这个描述符传给线程，在调用read方法读取数据
- 

# Redis知识总结

> ### redis是二进制安全的，是按照字节数组的形式存储的
>
> ### redis-cli --raw启动客户端，获取值会按照编码格式进行显示，而不是显示ASCII码

## 命令学习方式

![](D:\0_LeargingSummary\Redis\images\redis-cli_help命令.png)

# String命令操作

![](D:\0_LeargingSummary\Redis\images\String类型的命令操作.png)



## set

![](D:\0_LeargingSummary\Redis\images\set.png)

## mset

![](D:\0_LeargingSummary\Redis\images\mset多值插入.png)

## append追加

![](D:\0_LeargingSummary\Redis\images\append.png)

## setrange指定下标设值

![](D:\0_LeargingSummary\Redis\images\setrange.png)

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

> ![](D:\0_LeargingSummary\Redis\images\对二进制的操作.png)

![](D:\0_LeargingSummary\Redis\images\setbit.png)

## bitpos

![](D:\0_LeargingSummary\Redis\images\bitpos.png)

## bitcount统计二进制1出现的次数

![](D:\0_LeargingSummary\Redis\images\bitcount.png)

## bitop二进制计算

![](D:\0_LeargingSummary\Redis\images\bitop.png)

# List操作命令

![](D:\0_LeargingSummary\Redis\images\list操作命令.png)

## LPush\LRange\Lpop

![](D:\0_LeargingSummary\Redis\images\LPush.png)

## LIndex\Lset

![](D:\0_LeargingSummary\Redis\images\LIndex-LSet.png)

## LRem移除指定个数的元素

![](D:\0_LeargingSummary\Redis\images\LRem.png)

## LInsert

![](D:\0_LeargingSummary\Redis\images\LInsert.png)

## BLPOP阻塞获取

![](D:\0_LeargingSummary\Redis\images\BLpop.png)

## LTrim

![](D:\0_LeargingSummary\Redis\images\LTrim.png)

# Hash

![](D:\0_LeargingSummary\Redis\images\hash.png)

## hset\hmset\hget\hmget\hkeys\hvals\hgetall

![](D:\0_LeargingSummary\Redis\images\hash一系列操作.png)

## HIncrByFloat

![](D:\0_LeargingSummary\Redis\images\HIncrByFloat.png)

# Set

![](D:\0_LeargingSummary\Redis\images\set操作命令.png)

## sadd\sinter\sinterstore\smembers\sunion\sdiff

![](D:\0_LeargingSummary\Redis\images\set一系列操作.png)

## SRandMember\Spop

![](D:\0_LeargingSummary\Redis\images\SRandMember-spop.png)

# sorted_set

![](D:\0_LeargingSummary\Redis\images\sort_set操作命令.png)

## ZAdd\ZRange\ZRevRange\

![](D:\0_LeargingSummary\Redis\images\sorted_set一系列操作.png)

## ZScore\ZRank\ZRange\ZIncrBy\

![](D:\0_LeargingSummary\Redis\images\ZScore.png)

# 管道

```shell
echo -e 启动对反斜杠的转义功能，nc建立一个redis服务的socket连接，可以一次发送多条命令
[root@node1 diao]# echo -e "set k2 yuhang \n set k3 99 \n incr k3 | nc localhost 6379
```

# PUB\SUB

![](D:\0_LeargingSummary\Redis\images\pub-sub.png)

# 事务

![](D:\0_LeargingSummary\Redis\images\事务.png)

# 布隆过滤器

```shell
bf.add 添加元素到布隆过滤器
  bf.exists 判断元素是否在布隆过滤器
  bf.madd 添加多个元素到布隆过滤器，bf.add只能添加一个
  bf.mexists 判断多个元素是否在布隆过滤器
```

> ### 利用Redis的BitMap实现布隆过滤器的底层映射(以二进制的方式存储)。
>
> 当布隆过滤器返回不存在时，那就一定不存在；返回存在时，极小概率可能是不存在的
>
> 如果穿透了，不存在，采取措施：在client，增加redis中的key，value标记。
>
> 数据库增加了元素，对bloom的添加

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
appendonly no

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
aof-use-rdb-preamble yes
```



## Linux中的补充点

- 管道会出发创建子进程

```shell
echo $$|more
echo $BASHPID|more
```

![](D:\0_LeargingSummary\Redis\images\管道出发子进程.png)

正常情况下，进程之间的数据是隔离的。

但是，父进程可以让子进程看到数据！export的环境变量，子进程的修改不会影响到父进程，同样父进程对变量的修改也是不会影响的子进程。

- linux系统调用fork(),速度快，空间小

创建子进程并不会发生数据的复制，只是将父进程中虚拟地址指针复制了一份，如果父进程发生修改操作，fork中使用的是copy-on-write机制写时复制，原内存地址地址空间的数据是不会变化的，复制出一份数据进行修改，然后将原指针指向新地址，这样就不会影响到子进程的操作

# 击穿/穿透/雪崩

击穿：当redis缓存中的key失效了，同时又有并发进来了，发现redis中没有该key，就会到数据库中查找；

解决法方案：redis是单线程实例，当第一个连接发现没有key时，就setnx(不存在时设置成功)设置锁，获得锁的连接才能到数据库进行查询，从数据库中查询成功后，重新设置redis的key，其它并发的连接获取不到锁就短暂的睡眠，醒来后再去重新获取key。



穿透：查询的是系统中根本不存在的数据。布隆过滤器

雪崩：大量的key同时失效，造成大量的访问到达数据库。随机过期时间

# 分布式锁：redis

- setnx
- 过期时间
- 多线程（守护线程）延长过期时间