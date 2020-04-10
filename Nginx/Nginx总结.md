# Nginx总结

## Nginx配置解析

### 定义Nginx运行的用户和用户组，进程数

进程建议设置成cpu核心数

![](D:\0_LeargingSummary\Nginx\images\用户组进程设置.png)

### 全局错误日志

全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]

![](D:\0_LeargingSummary\Nginx\images\全局日志配置.png)

### Http

include mime.types; #文件扩展名与文件类型映射表

default_type application/octet-stream; #默认文件类型

charset utf-8; #默认编码

client_header_buffer_size 32k; #上传文件大小限制

sendfile on; #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。

sendfile()还能够用来在两个文件夹之间移动数据

`tcp_nopush` 在linux/Unix系统中优化tcp数据传输，仅在sendfile开启时有效

`autoindex on; `#开启目录列表访问，合适下载服务器，默认关闭。

`keepalive_timeout 120; `#长连接超时时间，单位是秒



`gzip on;` 开启gzip压缩输出

`gzip_min_length 1k;`  设置允许压缩的页面最小字节数，页面字节数从header头得content-length中进行获取。默认值是0，不管页面多大都压缩。建议设置成大于2k的字节数，小于2k可能会越压越大。  

`gzip_buffers 4 16k;` 设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。 例如 4 4k 代表以4k为单位，按照原始数据大小以4k为单位的4倍申请内存。 4 8k 代表以8k为单位，按照原始数据大小以8k为单位的4倍申请内存。    如果没有设置，默认值是申请跟原始数据相同大小的内存空间去存储gzip压缩结果。

`gzip_http_version 1.0; `压缩版本（默认1.1，前端如果是squid2.5请使用1.0）

`gzip_comp_level 2;` 压缩级别，1-10，数字越大压缩的越好，也越占用CPU时间  

`gzip_types text/plain application/x-javascript text/css application/xml;`
#压缩类型，默认就已经包含text/html，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。

 默认值: gzip_types text/html (默认不对js/css文件进行压缩) 

压缩类型，匹配MIME类型进行压缩 

设置哪压缩种文本文件可参考 conf/mime.types

 `gzip_disable "MSIE [1-6]\.";   `E6及以下禁止压缩

`gzip_vary on;  `给CDN和代理服务器使用，针对相同url，可以根据头信息返回压缩和非压缩副本  

![](D:\0_LeargingSummary\Nginx\images\http设置.png)

### Server

` listen       80;`  监听端口
`server_name www.mashibing.com mashibing.com; ` 域名可以有多个，用空格隔开

`charset koi8-r;` 编码集

 ```access_log  logs/host.access.log  main;
日志相关
access_log  "pipe:rollback logs/host.access_log interval=1d baknum=7 maxsize=2G"  main;
 ```

`index index.html index.htm index.jsp;` 默认页
`root /data/www/ha97;` 主目录

![](D:\0_LeargingSummary\Nginx\images\server配置.png)

虚拟主机是一种特殊的软硬件技术，它可以将网络上的每一台计算机分成多个虚拟主机，每个虚拟主机可以独立对外提供www服务，这样就可以实现一台主机对外提供多个web服务，每个虚拟主机之间是独立的，互不影响的

通过nginx可以实现虚拟主机的配置，nginx支持三种类型的虚拟主机配置

- 基于ip的虚拟主机， （一块主机绑定多个ip地址）
- 基于域名的虚拟主机（servername）
- 基于端口的虚拟主机（listen如果不写ip端口模式）

![](D:\0_LeargingSummary\Nginx\images\虚拟主机配置.png)



##### location

映射/虚拟目录

```
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /documents/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```

location [ = | ~ | ~* | ^~ ] uri { ... }

`location URI {}` 对当前路径及子路径下的所有对象都生效；

`location = URI {}` 注意URL最好为具体路径。  精确匹配指定的路径，不包括子路径，因此，只对当前资源生效；

`location ~ URI {}   location ~* URI {} `  模式匹配URI，此处的URI可使用正则表达式，~区分字符大小写，~*不区分字符大小写；

`location ^~ URI {}` 禁用正则表达式

**优先级**：= > ^~ > ~|~* >  /|/dir/

###### location配置规则

location 的执行逻辑跟 location 的编辑顺序无关。
矫正：这句话不全对，“普通 location ”的匹配规则是“最大前缀”，因此“普通 location ”的确与 location 编辑顺序无关；

但是“正则 location ”的匹配规则是“顺序匹配，且只要匹配到第一个就停止后面的匹配”；

“普通location ”与“正则 location ”之间的匹配顺序是？先匹配普通 location ，再“考虑”匹配正则 location 。

注意这里的“考虑”是“可能”的意思，也就是说匹配完“普通 location ”后，有的时候需要继续匹配“正则 location ”，有的时候则不需要继续匹配“正则 location ”。两种情况下，不需要继续匹配正则 location ：

- （ 1 ）当普通 location 前面指定了“ ^~ ”，特别告诉 Nginx 本条普通 location 一旦匹配上，则不需要继续正则匹配；
- （ 2 ）当普通location 恰好严格匹配上，不是最大前缀匹配，则不再继续匹配正则

##### location中alias与root的区别

- root   实际访问文件路径会拼接URL中的路径,

- alias  实际访问文件路径不会拼接URL中的路径

  经过多次试验root匹配的location后面只能是'/'符号，可以有多个alias别名路径（必须是绝对路径）

![](D:\0_LeargingSummary\Nginx\images\location中alias与root配置.png)