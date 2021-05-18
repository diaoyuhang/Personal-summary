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

- 

java中RandomAccessFile可以实现文件的随机读写操作，通过RandomAccessFile可以获得FileChannel文件通道，通过FileChannel可以获得MappedByteBuffer

通过MappedByteBuffer可以不使用系统调用直接将数据写入pagecache，就没有了用户态内核态的切换
