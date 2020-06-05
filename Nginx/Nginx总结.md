# Nginx

## Nginx和Apache的优缺点

**nginx**

- 轻量级，同样是web服务器，但是比apache所占资源少
- 抗并发，nginx异步非阻塞处理请求，而apache同步阻塞

**Apache**

- 稳定
- rewrite强大

## Nginx配置文件解析

**user  nobody**

定义nginx运行的用户和用户组

**worker_processes  1**

定义工作的进程数

**全局错误日志**

error_log  logs/error.log;

**worker_connections**

单个进程最大的连接数量，总连接数量是work_processes*worker_connections,但是收到系统的最大文件句柄个数的限制，cat /proc/sys/fs/file-max查看系统最大句柄个数，cat /proc/sys/fs/file-nr查看当前系统已经使用的句柄个数

**http**

- include       mime.types #文件扩展名与文件类型映射表
- default_type  application/octet-stream#默认文件类型
- sendfile        on 开启高效文件传输，零拷贝
- tcp_nopush 优化tcp数据传输，仅在sendfile开启时有效
- keepalive_timeout 长连接有效时间
- server 虚拟主机
  - ` listen       80;`  监听端口
  - `server_name www.mashibing.com mashibing.com; ` 域名可以有多个，用空格隔开
  - `charset koi8-r;` 编码集
  - `index index.html index.htm index.jsp;` 默认页
  - `root /data/www/ha97;` 主目录

> 虚拟主机是一种特殊的软硬件技术，它可以将网络上的每一台计算机分成多个虚拟主机，每个虚拟主机可以独立对外提供www服务，这样就可以实现一台主机对外提供多个web服务，每个虚拟主机之间是独立的，互不影响的。通过nginx可以实现虚拟主机的配置，nginx支持三种类型的虚拟主机配置
>
> - 基于ip的虚拟主机， （一块主机绑定多个ip地址）
> - 基于域名的虚拟主机（servername）
> - 基于端口的虚拟主机（listen如果不写ip端口模式）

- location映射/虚拟目录

  - ```
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
    
    > ###### location配置规则
    >
    > location 的执行逻辑跟 location 的编辑顺序无关。
    > 矫正：这句话不全对，“普通 location ”的匹配规则是“最大前缀”，因此“普通 location ”的确与 location 编辑顺序无关；
    >
    > 但是“正则 location ”的匹配规则是“顺序匹配，且只要匹配到第一个就停止后面的匹配”；
    >
    > “普通location ”与“正则 location ”之间的匹配顺序是？先匹配普通 location ，再“考虑”匹配正则 location 。
    >
    > 注意这里的“考虑”是“可能”的意思，也就是说匹配完“普通 location ”后，有的时候需要继续匹配“正则 location ”，有的时候则不需要继续匹配“正则 location ”。两种情况下，不需要继续匹配正则 location ：
    >
    > - （ 1 ）当普通 location 前面指定了“ ^~ ”，特别告诉 Nginx 本条普通 location 一旦匹配上，则不需要继续正则匹配；
    > - （ 2 ）当普通location 恰好严格匹配上，不是最大前缀匹配，则不再继续匹配正则

  - 用户认证访问

    模块ngx_http_auth_basic_module 允许使用“HTTP基本认证”协议验证用户名和密码来限制对资源的访问。

    ```shell
    location ~(.*)\.avi$ {
        auth_basic  "closed site";
        auth_basic_user_file conf/users;
    }
    ```

  - 反向代理

    ```shell
        location /baidu {
                proxy_pass http://www.baidu.com;
        }
    ```

## 负载均衡

**upstream**

```shell
upstream tomcats {
    server 192.168.253.135:8080;
    server 192.168.253.136:8080;
}
```
```shell
location /tocmat {
	proxy_pass http://tomcats/; ##这里最后一个斜杠必须要加，否则失效
}
```
**weight(权重)**

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。

```
upstream httpds {
    server 127.0.0.1:8050       weight=10 down;
    server 127.0.0.1:8060       weight=1;
     server 127.0.0.1:8060      weight=1 backup;
}
```

- down：表示当前的server暂时不参与负载 
- weight：默认为1.weight越大，负载的权重就越大。 
- backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。

**max_conns**

可以根据服务的好坏来设置最大连接数，防止挂掉，比如1000，我们可以设置800

```
upstream httpds {
    server 127.0.0.1:8050    weight=5  max_conns=800;
    server 127.0.0.1:8060    weight=1;
}
```

**max_fails、 fail_timeout**

max_fails:失败多少次 认为主机已挂掉则，踢出，公司资源少的话一般设置2~3次，多的话设置1次

max_fails=3 fail_timeout=30s代表在30秒内请求某一应用失败3次，认为该应用宕机，后等待30秒，这期间内不会再把新请求发送到宕机应用，而是直接发到正常的那一台，时间到后再有请求进来继续尝试连接宕机应用且仅尝试1次，如果还是失败，则继续等待30秒...以此循环，直到恢复。

```
upstream httpds {
    server 127.0.0.1:8050    weight=1  max_fails=1  fail_timeout=20;
    server 127.0.0.1:8060    weight=1;
}
```

**负载均衡算法**

**轮询+weight**  **ip_hash** **url_hash** **least_conn** **least_time** 

## 虚拟目录

```shell
    location /www {
            alias /opt/modules/www ;
            autoindex on; ## 开启自动索引
    }
```
## 动静分离

```shell
    location /tomcat/ {
            proxy_pass http://tomcats/;
    }
## 将静态资源请求连接
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|html|htm|css|js|ico|svg)$ {
            alias /opt/modules/static;
    }
```
## SSL

SSL 能够帮助系统在客户端和服务器之间建立一条安全通信通道。SSL 安全协议是由 Netscape Communication 公司在 1994 年设计开发，SSL 依赖于加密算法、极难窃听、有较高的安全性，因此 SSL 协议已经成为网络上最常用的安全保密通信协议，该安全协议主要用来提供对用户和服务器的认证；对传送的数据进行加密和隐藏；确保数据在传送中不被改变，即数据的完整性，现已成为该领域中全球化的标准。

### 非对称加密

非对称加密算法需要两个密钥：公开密钥（publickey）和私有密钥（privatekey），公开密钥与私有密钥是一对，如果用公开密钥对数据进行加密，只有用对应的私有密钥才能解密；如果用私有密钥对数据进行加密，那么只有用对应的公开密钥才能解密。因为加密和解密使用的是两个不同的密钥，所以这种算法叫作非对称加密算法。

> #### 但是仅仅这样依然是不安全的，如果数据在传输过程中被拦截，中间商就可以使用公钥进行解密修改内容，然后用自己的私钥进行加密，传输给客户端，客服端使用中间商的公钥进行解密，得到的数据就是不安全的数据。

### CA(Certificate Authority)

CA 是负责签发证书、认证证书、管理已颁发证书的机关。

**证书种类**

![](D:\0_LeargingSummary\Nginx\images\83cf43c8886ff637c1eb77a319bb0fc5a327e3e5.jpeg)

**多网站公用同一证书**

### OpenSSL

- key 私钥  =  明文 自己生成的
- csr 公钥    = 由私钥生成 
- crt 证书    = 公钥 + 签名

**生成私钥**

制台输入 genrsa，会默认生成一个 2048 位的私钥

```
openssl genrsa -des3 -out server.key 1024
```

**有私钥生成公钥**

```
openssl req -new -key server.key -out server.csr
```

**查看证书内容**

```
req -text -in server.csr
```

- Country Name (2 letter code) [XX]:CN           #请求签署人的信息
- State or Province Name (full name) []: #请求签署人的省份名字
- Locality Name (eg, city) [Default City]:# 请求签署人的城市名字
- Organization Name (eg, company) [Default Company Ltd]:#请求签署人的公司名字
- Organizational Unit Name (eg, section) []:#请求签署人的部门名字
- Common Name (eg, your name or your server's hostname) []:#这里一般填写请求人的的服务器域名， 

**签名**

```
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

**nginx配置HTTPS**

```
server {
        listen       443 ssl;
        server_name  192.168.253.135;

        ssl_certificate      /opt/modules/tenginx/server.crt;
        ssl_certificate_key  /opt/modules/tenginx/server.key;
}
```

