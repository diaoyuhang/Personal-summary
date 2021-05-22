```shell
命令帮助文档man

       1   Executable programs or shell commands
       2   System calls (functions provided by the kernel)
       3   Library calls (functions within program libraries)
       4   Special files (usually found in /dev)
       5   File formats and conventions eg /etc/passwd
       6   Games
       7   Miscellaneous (including macro packages and conventions), e.g. man(7), groff(7)
       8   System administration commands (usually only for root)
       9   Kernel routines [Non standard]

```

# 文件系统io

application应用程序想要读取io，需要通过调用系统内核

- kernel操作系统内核，就是一个程序

- VFS虚拟文件系统（目录树，每个节点映射到不同存储节点）

  - inode，每个文件打开的时候都有一个唯一的inode号标识

  - pagecache页缓存，默认4k，读取文件先由内核读取，读到内存中开辟一个页缓存pagecache；

    如果两个程序打开同一个文件，是不会加载两次文件，共享一个pagecache；数据被加载到内存，程序和内存交互；

  - dirty脏，如果pagecache被程序修改，就会被标记为dirty；

  - flush,将脏页刷写到磁盘，可以由内核自动刷写（这里是flush的全部的脏页），也可以由程序调用内核刷写；

  - FD文件描述符，不同的程序对同一文件读写都有一个文件描述符，FD中有一个指针seek，每个程序通过这个seek找到自己读写的pagecache

```shell
[root@node01 ~]# df  查看不同分区的目录挂载，在虚拟文件系统中：/ 根目录所有文件下是来自所有分区的
Filesystem     1K-blocks     Used Available Use% Mounted on
/dev/sda3      202092480 10776508 181050220   6% /
tmpfs            1954400        0   1954400   0% /dev/shm
/dev/sda1         198337    27795    160302  15% /boot
```

- 文件类型(常见)

  ```json
  -:普通文件
  d:目录
  b:块设备 可来回随意的读 （硬盘）
  c：字符设备 不能读取到过去和未来的的数据（键盘）
  s: socket
  p: pipeline
  l:链接 硬链接、软连接 （命令：stat 文件名 可以查看文件iNode信息）
  eventPoll:
  ```

  
  
  ```shell
  lsof -op 进程号  （查看某一进程打开了那些文件,-o作用查看fd指针偏移量）
  
  标准输入输出FD文件描述，每个程序都会有这三个文件描述符
  0:输入
  1:输出
  2：错误输出
  
  输入输出重定向操作符，重定向左边为文件描述符，右边可接文件(1< abc.txt),如果右边接文件描述符（1<& 2）
  < ：输入
  > ：输出
  
  管道 |
  管道左右两边都会启动一个子进程，管道衔接两个进程的输入输出；
  
  这两个都会输出当前进程的id号；
  echo $$ 
  echo $BASHPID
  注意：echo $$ | cat 输出的是父进程的id号; echo $BASHPID | cat 输出的子进程的id号；
  因为$$优先级高于管道，$BASHPID优先级低于管道；
  ```
  
  

# 内存

物理内存中包含了kernel内核程序、各种app应用程序等等；

各种程序在物理内存上不一定是连续的，所有每个程序都会有一个虚拟内存，通过mmu映射到物理内存的每个pagecache上；

程序的内存申请不是一下子全部申请，而是随用随申请，当发现执行下一步没有数据，就会发生异常缺页，用户态切换到内核态，内核就会在物理内存上再分配出一个pagecache映射到虚拟内存；

程序发生写操作的时候是写到pagecache中，不是写到磁盘中，pagecache会被标记为dirty脏页，如果不手动调用系统刷新，将脏页写到磁盘，只有等到pagecache的体积达到系统阈值，由内核调用刷新写到磁盘，所有这样会有丢失大量数据的风险；

- java中的为什么BufferedOutputStream会比不带缓冲区的字节流快？

  因为java每write一次就是一次系统调用，需要用户态内核态的切换，BufferedOutputStream减少了系统调用

## mmap

- 简而言之，就是将内核空间的一段内存区域映射到用户空间，用户对这段内存区域的修改可以直接反应到内核空间；当然这段内存区域可以映射到多个用户空间（多个进程），实现进程间的内存共享通信；
-  但是，依然受到内核pagecache的约束，需要调用系统调用将pagecache脏页写入磁盘，不然还是会丢数据；

java中RandomAccessFile可以实现文件的随机读写操作，通过RandomAccessFile可以获得FileChannel文件通道，通过FileChannel可以获得MappedByteBuffer

通过MappedByteBuffer可以不使用系统调用直接将数据写入pagecache，就没有了用户态内核态的切换，少了一次cpu拷贝

# 网络IO

- netstat -natp 罗列出所有tcp连接

  -n 禁止域名解析功能，有助于提高查询速度

  -a 列出所有监听和未监听端口

  -t 列出所有tcp协议端口

  -p 列出tcp连接持有者的进程ID

- tcpdump -nn -i 网卡 port 端口号   实时展示数据包的交互过程

  -nn 不进行端口名称的转换

  -i 指定监听网络接口

## Tcp

- 面向连接的，经历三次握手，在内核中开辟资源

**注：java中在accept之前，连接就已经创建完成了，只是该连接没有被分配**

- 什么是socket?

**socket是指四元组：ClientIP:port+ServerIP:port ,四元组是唯一的，代表了一个tcp连接**

**当java服务端accept之后，这个连接才会被分配到一个进程，进程中用一个文件描述符指向该连接；**

- tcp参数 服务启动配置BACK_LOG,服务端上客户端Socket属性KEEPALIVE,客户端TcpNoDelay

  BACK_LOG:服务最多有保留N个未分配的连接

  KEEPALIVE:如果客户端长时间未通信，确保client是否活着，会定时发送数据包

  TcpNoDelay:客户端会将数据攒到一起，一次性发送

- Tcp拥塞控制

  MTU(最大传输单元,1500字节) ；MSS（TCP用来限制application层最大的发送字节数），MSS = 1500- 20(IP Header) -20 (TCP Header) = 1460 byte

  客户端，服务端双发都会有一个窗口大小，当一方的窗口已满，接收方回复发送方，包含信息窗口大小为0，发送方就会阻塞自己，停止发送，直到接收方有空间，如果不阻塞一直发送，会造成后面的数据被对其；

## IO模型变化

- strace -ff -o out cmd 追踪系统调用

  -ff 追踪cmd程序中所有的进程产生的系统调用

  -o 	输出到以out.pid的文件中,pid进程号

### BIO

同步阻塞模型，服务启动必会发生的三次系统调用

```java
ServerSocket server = new ServerSocket(9090,20);
```

socket 返回一个文件描述符

bind 将文件描述符绑定到指定端口

listen 监听这个文件描述符

```java
server.accept();//这一步会阻塞，直到有连接进来
```

等待读取同样是会发生阻塞；

如果将读取操作交给一个新的线程进行操作，创建一个线程有需要clone系统调用，频繁创建线程影响cpu性能；

而且当读取的线程都阻塞住，那么线程资源无法被释放，cpu资源浪费在线程间的切换；

### NIO

```c
Nio系统调用

socket(AF_INET6, SOCK_STREAM, IPPROTO_IP) = 6
bind(6, {sa_family=AF_INET6, sin6_port=htons(9091),    
listen(6, 50)                           = 0
    
accept(6, 0x7f66bc0fee90, [28])         = -1 //这里在接受有没有新连接，不会阻塞
read(9, 0x7f66bc104b20, 4096)           = -1 //这里读取有没有数据进来，不会阻塞
```

**在使用多路复用器之前，为读取数据，服务器需要遍历所有连接，调用系统调用来判断是否有数据到达；**

**有了多路复用器，服务器只需要调用一次多路复用器，由内核返回哪些连接有数据，然后由程序再对这些有数据的连接的进行读取！**

**注：这里只要是程序自己进行读写，那么模型就是同步的**

```
只关注IO读写事件，不关注IO读写完之后怎么处理
同步：app自己进行R/W
异步：kernel完成R/W，内核将数据读取到程序进程的区域，程序不需要调用recv
```

### 多路复用器

**select**

```
synchronous I/O multiplexing
int select(int nfds, fd_set *readfds, fd_set *writefds,fd_set *exceptfds, struct timeval *timeout);

缺点：一次只能传1024个文件描述符
```

**POLL**

```
没有1024的限制
```

**注：其实以上都是要遍历所有的io，询问状态，只是由内核遍历**

**中断**

软中断：程序需要调用内核系统调用，这时就会产生软中断

硬中断：时钟中断

有中断就可以产生回调

**EPOLL**

epoll_create、epoll_ctl、epoll_wait

1. 调用epoll_create开辟空间，并返回文件描述符（比如fd4）代表这个空间；
2. 调用epoll_ctl向fd4这个空间中添加、删除其他文件描述符，这里是红黑树结构；
3. 调用epoll_wait查看一个链表空间是否有可操作的文件描述符；

**注：当网卡接受到数据或是请求时，产生中断，之后还有一个回调函数将红黑树中的对应有状态文件描述符拷贝到链表中，这是程序调用epoll_wait就可以直接获取到哪些fd可以操作**

```
注：异步的IO模型，其真正的读操作是发生在内核中，并不是app去读取数据
```

