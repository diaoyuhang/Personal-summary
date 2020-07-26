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
| docker rm <-f> 容器id                   | 删除容器                                                     |

![](D:\0_LeargingSummary\Docker\images\停止运行的容器.png)

> 更多命令参看
> https://docs.docker.com/engine/reference/commandline/docker/
> 可以参考每一个镜像的文档  

## 三、宿主机与容器通信

`docker run -d -p 8000:8080 tomcat`

-p就是将本机的端口8000用于监听tomcat容器8080端口

![](images\QQ截图20200726134444.png)

## 四、容器内部结构

**比如tomcat**

![](images\tomcat容器内部结构.png)

## 五、在容器中执行命令

`格式：docker exec [-it] 容器id 命令`

exec 在对应容器中执行命令

-it 采用交互方式执行命令

**实例：docker exec -it 0738ed2fe68b  /bin/bash**

## 六、容器生命周期

![](D:\0_LeargingSummary\Docker\images\容器生命周期.png)

## 七、Dockerfile镜像描述文件

Dockerfile是一个包含用于组合镜像的命令的文本文档

Docker通过读取Dockerfile中的指令按步自动生成镜像

`docker build -t 机构/镜像名<:tags> Dockerfile目录`

**实例**

创建一个Dockerfile文件

```markdown
FROM tomcat:latest 制作基准镜像
MAINTAINER diaoyuhang 标明维护者
WORKDIR /usr/local/tomcat/webapps 相当于在容器中cd /usr/local/tomcat/webapps
ADD docker-web ./docker-web 将docker-web目录的文件拷贝到容器中docker-web目录下
```

## 八、Dockerfile基础命令

**基础命令**

- FROM 基于基准镜像

- LABEL & MAINTAINER - 说明信息

- WORKDIR - 设置工作目录

  WORKDIR /usr/local/newdir #自动创建
  尽量使用绝对路径

- ADD复制文件

  ADD hello / #复制到根路径
  ADD test.tar.gz / #添加根目录并解压
  ADD 除了复制,还具备添加远程文件功能

- ENV 设置环境变量

  ENV JAVA_HOME /usr/local/openjdk8
  RUN ${JAVA_HOME}/bin/java -jar test.jar
  尽量使用环境常量,可提高程序维护性

- EXPOSE 暴露容器端口

  将容器内部端口暴露给物理机
  EXPOSE 8080
  docker run -p 8000:8080 tomcat

**执行命令**

- RUN & CMD & ENTRYPOINT

  RUN: 在build构建镜像的时候执行

  CMD: 在容器创建时执行命令

  ENTRYPOINT: 在容器创建时执行命令

> ### 注：RUN-构建时运行
>
> RUN yum install -y vim  #Shell 命令格式
> RUN ["yum","install","-y","vim"] #Exec命令格式
>
> 使用Shell执行时，当前shell是父进程，生成一个子shell进程
> 在子shell中执行脚本。脚本执行完毕，退出子shell，回到当前shell。
>
> 使用Exec方式，会用Exec进程替换当前进程，并且保持PID不变
> 执行完毕，直接退出，并不会退回之前的进程环境
>
> ### ENTRYPOINT命令
>
> ENTRYPOINT(入口点)用于在容器启动时执行命令
> Dockerfile中只有最后一个ENTRYPOINT会被执行
> ENTRYPOINT ["ps"] #推荐使用Exec格式
>
> ### CMD默认命令
>
> CMD用于设置默认执行的命令
> 如Dockerfile中出现多个CMD,则只有最后一个被执行
> 如容器启动时附加指令,则CMD被忽略
> CMD ["ps" , "-ef"] #推荐使用Exec格式

## 九、镜像分层

在执行Dockerfile文件的时候每执行一步都会生成一个临时容器，这个临时容器只能用来被构建，不能执行。

临时容器在创建过程中是会被重用的

![](D:\0_LeargingSummary\Docker\images\QQ截图20200726144625.png)

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

