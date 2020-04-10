# Docker使用

## 一、安装Docker

> ## 前提：centos7

### 更新yum源

```sh
yum -y update
```

### 移除原有的Docker

```sh
 yum remove docker
```

### 安装Docker

```sh
yum -y install docker
```

### 启动Docker

```
systemctl start docker
```

### 设置成开机启动

```sh
systemctl enable docker
```

### 停止Docker

```sh
systemctl stop docker
```

## 二、Docker常用命令&操作

### 1、镜像操作

| 操作     | 命令                   | 说明                                                     |
| :------- | :--------------------- | -------------------------------------------------------- |
| 检索镜像 | docker search 镜像名称 | 我们经常去docker hub上检索镜像的详细信息，如镜像的TAG    |
| 拉取镜像 | docker pull 镜像名:tag | :tag是可选的，tag表示标签，多为软件的版本，默认 是latest |
| 本地镜像 | docker images          | 查看所有本地镜像                                         |
| 删除镜像 | docker rmi -f image-id | 删除指定的本地镜像                                       |

![](D:\0_LeargingSummary\Docker\images\列出本地镜像.png)

### 2、容器操作

| 命令                                    | 说明                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| docker ps (-a)                          | 查看运行中的容器                                             |
| docker stop 运行的的容器id              | 停止运行中的容器                                             |
| docker run ‐d ‐p 8888:8080 tomcat       | 启动做一个端口映射，-d后台运行，-p将主机的端口映射到容器的一个端口  主机端口:容器内部的端口 |
| docker logs container‐name/container‐id | 查看容器的日志                                               |

![](D:\0_LeargingSummary\Docker\images\停止运行的容器.png)

> 更多命令参看
> https://docs.docker.com/engine/reference/commandline/docker/
> 可以参考每一个镜像的文档  

## 三、事例：mysql

```sh
#拉取msyql,默认罪行latest
docker pull mysql
#-e输入环境变量参数，指定密码，-p做端口映射
docker run ‐p 3306:3306 ‐‐name mysql02 ‐e MYSQL_ROOT_PASSWORD=123456 ‐d
mysql
```

## 四、事例：rabbitmq

- 拉取镜像

```sh
docker pull rabbitmq:3.8-rc-management
```

- 启动镜像

```sh
docker run -d --name="MyRabbitMQ" -p 5672:5672 -p 15672:15672 rabbitmq:TAG
```

