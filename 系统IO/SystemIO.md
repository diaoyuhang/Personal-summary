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



