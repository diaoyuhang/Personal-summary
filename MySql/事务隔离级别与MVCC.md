# 什么是事务

事务是DBMS数据库管理系统执行过程中的一个逻辑单元，由有限的数据库操作构成

## 事务四大特性（ACID）

- 原子性（Atomicity）：指数据库的一系列操作之后，要么全部成功，要么全部失败；
- 一致性（Consistency）：指事务执行前后的数据状态是合法的；
- 隔离性（Isolation）：多个事务彼此之间是完全隔离的、互不干扰；
- 持久性（Durability）：只要事务提交成功，那么对数据库修改就会被永久保存；

## 事务状态

- 活动状态：当前事务对应的数据库操作正在执行中
- 部分提交状态：事务中最后一个操作结束，但未提交
- 失败状态：活动或部分提交状态时，出现错误
- 中止状态：当处于失败状态，且回滚执行完毕
- 提交状态：事务提交成功

## 事务的隔离级别

事务并发执行时，如果不做任何控制，可能出现脏写、脏读、不可重复读、幻读

- 脏写：一个事务修改了其他事务未提交的数据；

  事务A和事务B同时修改一条记录，事务A先将值改为A未提交，然后事务B将值改为B提交了，这是事务回滚，则事务B的值将不复存在

- 脏读：一个事务读到了其他事务未提交的数据

  事务A将值修改为A未提交，事务B读到了A，然后事务A回滚数据，事务B其实就是读到一个不存在的数据

- 不可重复读：一个事务中对同一条记录的多次查询，第二次查询前该记录被其他事务修改，导致两次查询结果不一样

- 幻读：一个事务执行过程中，读取到其他事务新插入的数据，导致两次读取结果不一致

  事务A第一次条件查询得到10条记录，然后事务B插入一条记录，事务A再次查询得到11条记录

> 不可重复读和幻读的区别就是，不可重复读针对的是读到了其他事务修改的数据，幻读读到其他事务新插入的记录

## 四种隔离级别

各个隔离级别下可能出现的读一致性问题如下：

| 隔离级别                     | 脏读   | 不可重复读 | 幻读                   |
| ---------------------------- | ------ | ---------- | ---------------------- |
| 未提交读（READ UNCOMMITTED） | 可能   | 可能       | 可能                   |
| 已提交读（READ COMMITTED）   | 不可能 | 可能       | 可能                   |
| 可重复读（REPEATABLE READ）  | 不可能 | 不可能     | 可能（对InnoDB不可能） |
| 串行化（SERIALIZABLE）       | 不可能 | 不可能     | 不可能                 |

`InnoDB`支持四个隔离级别（和`SQL`标准定义的基本一致）。隔离级别越高，事务的并发度就越低

# 什么是MVCC

MVCC多版本并发控制，通过维护数据历史版本，从而解决并发访问情况下的读一致性问题

## 版本链

在innoDB中，每行记录实际上都包含了两个隐藏字段事务id(trx_id)和回滚指针(roll_pointer)

1. trx_id：事务id，每次修改了一条记录时，都会把该事务的事务id赋值给trx_id
2. roll_pointer:回滚指针，每次修改记录时，都会把undo日志记录地址赋值给该字段

```sql
CREATE TABLE hero (
    number INT,
    name VARCHAR(100),
    country varchar(100),
    PRIMARY KEY (number)
) Engine=InnoDB CHARSET=utf8;
```

事务A插入了一条记录，当事务A的id为80，此时记录如下所示

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e41d5dcb3d704a60a3a5ded50593ee86~tplv-k3u1fbpfcp-zoom-1.image)

假设之后两个事务`id`分别为`100`、`200`的事务对这条记录进行`UPDATE`操作，操作流程如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e0ad4a317904426cac3903a89a293917~tplv-k3u1fbpfcp-zoom-1.image)

每次的变动都会先把undo日志记录下来，并用roll_pointer指向undo日志地址。对于该条记录的修改日志串联起来就形成了一个版本链，头结点就是当前记录的最新值。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ccde9c179fae491484bfffb3d8e4f521~tplv-k3u1fbpfcp-zoom-1.image)

## Read View视图

MVCC多版本并发控制也叫做快照读，

## 在Repeatable Read隔离级别下如何实现的

在REPEATABLE READ可重复读的隔离级别下，当事务开启时会创建一个视图（Read View）,这是视图也叫做快照，整个事务期间都会一直使用这个视图。

当执行begin开启事务之后，就会拍下如下图一样的快照：

![](https://img2020.cnblogs.com/blog/1496926/202012/1496926-20201202223831209-276590871.png)

假设事务A、B在同一时刻开启，那么两个事务会分别得到如下视图：

![](https://img2020.cnblogs.com/blog/1496926/202012/1496926-20201202223850839-941502644.png)

只要事务不提交，上述ReadView就一直有效。

利用上图来说：

在事务A中，它的事务id为61，此时活跃未提交的事务id为｛61，61｝，所有未提交的事务中id最小的就是61，下一个开启的事务id为63

在事务B中，它的事务id为62，此时活跃未提交的事务id为｛61，61｝，所有未提交的事务中id最小的就是61，下一个开启的事务id为63

1. 如果事务A想要读取name=tom,发现这条记录的事务id为60，而60小于当前活跃的最小事务id=61，说明事务ID=60的事务在事务A开启前就已经提交了。所以事务A能直接读取到数据。

2. 如果事务B通过update去修改这行数据，将name修改为jerry，就会记录undo log日志，形成版本链，如下图所示：

   ![](https://img2020.cnblogs.com/blog/1496926/202012/1496926-20201202223901339-575511488.png)

3. 如果这时事务A再读取这行数据，首先拿到数据name=jerry，但是发现记录的事务id=62，比当前活跃的最小id=61大，下一个事务id=63要小。说明，这条记录其实是和自己同时开启的事务修改的结果，这时就会沿着版本链往前找，直到找到一个事务id等于或者小于自己事务id的记录为止，所以事务A再一次读到的结果name=tom。

这就是所谓的快照机制。

在上述情况下，在可重复读的隔离级别下，确实保证事务A每次读取的结果都是一样的，在B修改了之后，依然可以读到name=tom。但是这个name=tom真就只是一个快照了，本质上算是一个不存在的数据。

## 在Read Commited隔离级别下是如果实现的

在Read Commited隔离级别下，每次select都会创建一个新的视图Read View

1. 还是使用上述例子，事务A、B并发开启，得到下图Read View。然后事务B将数据name=tom改成name=jerry未提交。

2. 事务A查询，首先找到name=jerry,但是发现trx_id=62,在trx_ids活跃的事务列表中，说明name=jerry是被同期的其他事务更改的，最后读到name=tom

   ![](https://img2020.cnblogs.com/blog/1496926/202012/1496926-20201202223913684-220126922.png)

3. 接着事务B提交了事务，事务A又一次查询，得到一个新的视图，如下图所示：

   当A得到数据name=jerry的trx_id=62，发现62不在trx_ids列表中。说明这个是一个已经提交的数据，那么事务就可以读出name=jerry。

   ![](https://img2020.cnblogs.com/blog/1496926/202012/1496926-20201202223920044-1239089718.png)

**如果数据库的隔离级别是READ UNCOMMITED读未提交，那么直接读取版本链中最新的记录就可以了**

