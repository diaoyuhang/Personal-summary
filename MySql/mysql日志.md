# bin log日志

binlog用于记录数据库执行的写入性操作，以二进制的形式保存，bin log是mysql的逻辑日志，并且是由server层进行记录，与存储引擎无关；

- 逻辑日志：简单认为就是记录的sql语句
- 物理日志：mysql数据最终是保存在数据页中，物理日志记录的就是数据页的变更,记录的是对物理磁盘上数据的修改；

binlog是通过追加的方式进行写入，可以通过max_binlog_size参数设置每个binlog的文件大小，当达到给定值后，就会生成新的文件保存日志，是不会覆盖之前的日志；

**主要应用场景主从复制和数据恢复**

## binlog的刷盘时机

对innoDB而言，只有在事务提交的时才会记录binlog，此时记录还在内存中，mysql是通过`sync_binlog`参数来控制binlog的刷盘时机，取值范围0~n

1. 0：不主动要求刷写，由系统来判断何时写入磁盘
2. 1：每次commit的时候都要将binlog写入磁盘，这是默认设置；
3. n：每n个事务，才会将binlog写入磁盘

## binlog的日志格式

statment、row和mixed

- statment:基于sql的复制，不需要记录每一个行的变化，减少日志量
- row：基于行的复制，缺点会产生大量日志
- `MIXED `： 基于` STATMENT `和 `ROW `两种模式的混合复制( `mixed-based replication, MBR `)，一般的复制使用 `STATEMENT `模式保存 `binlog `，对于 `STATEMENT `模式无法复制的操作使用 `ROW `模式保存 `binlog`

# redo log日志

事务里的四大特性持久性，就是靠这个日志来完成的，只要事务提交成功，那么对数据库的修改就被永久保存
**如果将事务涉及到的数据页全部刷写到磁盘会有很严重的性能问题**

1. 内存是以页为单位的，一个事务可能只修改了一个数据页中的几个字节，就需要将整个页刷到磁盘，浪费资源；
2. 一个事务也可能会涉及多个数据页，这些页物理上并不是连续的，随机io写入的性能太差

因此设计了redo log，只记录事务对数据页做了哪些修改，而且是顺序io

## redo log基本概念

由两个部分组成：一个是内存中的日志缓冲区redo log buffer,另一个是磁盘上的日志文件redo log file；每执行一个DML语句，先写入日志缓冲区，后续某个时间点再写入到日志文件，这种先写日志，在写磁盘技术叫做WAL(write-Ahead Loggin)技术。

- 用户空间下数据一般是无法直接写入磁盘，中间必须经过内核空间，由内核空间缓冲区（OS buffer）刷写到磁盘，实际上redolog buffer先写入OS buffer，然后再通过系统调用fsync()写入redo log file

![](https://image-static.segmentfault.com/755/731/755731335-aec527828a1323b6_fix732)

**支持三种刷写机制**

可以通过 `
innodb_flush_log_at_trx_commit ` 参数配置

- 0：事务提交不会立即将redo log buffer写入os buffer,而是每秒提交写入os buffer并调用fsync()刷写到磁盘，当系统崩溃，会丢失1秒钟的数据。
- 1：事务每次提交都会将redo log buffer中的数据写入os buffer中并调用fsync()写入磁盘，系统崩溃也不会对视数据，io性能较差
- 2：每次提交都仅仅写入os buffer,然后每秒fsync()写入磁盘

## **双1设置**

> ###  就是将redo log日志和binlog日志设置成每次事务提交就刷写到磁盘，innodb_flush_log_at_trx_commit=1&&sync_binlog=1

## 记录形式

`redo log `实际上记录数据页的变更，而这种变更记录是没必要全部保存，因此 `redo log`
实现上采用了大小固定，循环写入的方式，当写到结尾时，会回到开头循环写日志

![](https://image-static.segmentfault.com/390/444/3904443652-cc3225d69e1d0476_fix732)

> ##  在innodb中，既有` redo log `需要刷盘，还有 `数据页 `也需要刷盘， `redo log `存在的意义主要就是降低对 `数据页 `刷盘的要求 。



**在上图中， `write pos `表示 `redo log `当前记录的 `LSN` (逻辑序列号)位置， `check point `表示** 数据页更改记录 刷盘后对应 `redo log `所处的 `LSN `(逻辑序列号)位置。 `write pos `到 `check point `之间的部分是 `redo log `空着的部分，用于记录新的记录；` check point `到 `write pos `之间是 `redo log `待落盘的数据页更改记录。

**启动 `innodb `的时候，不管上次是正常关闭还是异常关闭，总是会进行恢复操作。因为 `redo log `记录的是数据页的物理变化，因此恢复的时候速度比逻辑日志(如 `binlog `)要快很多。 重启 `innodb `时，首先会检查磁盘中数据页的 `LSN `，如果数据页的 `LSN `小于日志中的 `LSN `，则会从 `checkpoint `开始恢复。 还有一种情况，在宕机前正处于`checkpoint `的刷盘过程，且数据页的刷盘进度超过了日志页的刷盘进度，此时会出现数据页中记录的 `LSN `大于日志中的 `LSN`，这时超出日志进度的部分将不会重做，因为这本身就表示已经做过的事情，无需再重做。**

## 如何保证数据不丢失的

1. mysql的server层的执行器调用Innodb存储引擎的进行数据更新；
2. 存储引擎更新Buffer Pool中的缓存页；
3. 同时存储引擎记录一条redo log到redo log buffer中，并将redo log的状态标记为prepare;
4. 接着通知执行器，可以提交事务，执行器接收到通知后，会写binlog日志，然后提交事务；
5. 存储引擎收到提交事务的通知后，会将redo log日志状态标记为commit状态；
6. 最后根据`innodb_flush_log_at_trx_commit`参数配置，决定是否将redo log记录刷写到磁盘中；binlog则是根据`sync_binlog`配置，决定是否将bin log记录刷写到磁盘；

启动 `innodb `的时候，不管上次是正常关闭还是异常关闭，总是会进行恢复操作。

如果redo log处于prepare状态后，buffer pool中缓存页（脏页）还没有刷新到磁盘中，写完bin log后就出现了宕机或断电，此时提交事务是失败的，那么mysql重启后，进行数据重做，在redo log日志中没有该事务的redo log记录的commit状态，那么就不会进行数据重做，那么磁盘上数据还是原来的的数据，也就是事务没有提交。

**要保证数据不丢失，必须配innodb_flush_log_at_trx_commit置为1**

> 比如：如果配置成0，每秒将redo log写入os buffer并刷写到磁盘，当redo log已经被标记为commit状态，由于此时的redo log处于redo log buffer中，并没有持久化到磁盘中，恰好buffer pool中更新的数据也没有刷新到磁盘，这时断电，redo log丢失，buffer pool中的数据也丢失了，然后mysql重启，由于少了一条redo log日志，就是少了一条对应的重做日志，这样断电前提交的那一次事务数据就丢失了。

# undo log日志

数据库中事务的原子性通过undo log实现的。undo log只要记录了数据的逻辑变化，比如insert语句，对应一条delet的记录日志，这样在发生错误时，就能回滚到事务之前的数据状态。