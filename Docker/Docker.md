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

![](images\列出本地镜像.png)

### 2、容器操作

| 命令                                    | 说明                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| docker ps (-a)                          | 查看运行中的容器                                             |
| docker stop 运行的的容器id              | 停止运行中的容器                                             |
| docker run ‐d ‐p 8888:8080 tomcat       | 启动做一个端口映射，-d后台运行，-p将主机的端口映射到容器的一个端口  主机端口:容器内部的端口 |
| docker logs container‐name/container‐id | 查看容器的日志                                               |

![](images\停止运行的容器.png)

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

## 五、构建YApi

**1、启动 MongoDB**

```undefined
docker run -d --name mongo-yapi mongo
```

**2、获取 Yapi 镜像，版本信息可在 [阿里云镜像仓库](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.aliyun.com%2Fdetail.html%3Fspm%3D5176.1972343.2.26.I97LV8%26repoId%3D139034) 查看**

```undefined
docker pull registry.cn-hangzhou.aliyuncs.com/anoy/yapi
```

**3、初始化 Yapi 数据库索引及管理员账号**

```markdown
docker run -it --rm \
  --link mongo-yapi:mongo \
  --entrypoint npm \
  --workdir /api/vendors \
  registry.cn-hangzhou.aliyuncs.com/anoy/yapi \
  run install-server
```

> 自定义配置文件挂载到目录 `/api/config.json`，官方自定义配置文件 -> [传送门](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FYMFE%2Fyapi%2Fblob%2Fmaster%2Fconfig_example.json)

**4、启动 Yapi 服务**

```markdown
docker run -d \
  --name yapi \
  --link mongo-yapi:mongo \
  --workdir /api/vendors \
  -p 3000:3000 \
  registry.cn-hangzhou.aliyuncs.com/anoy/yapi \
  server/app.js
```

### ▶ 使用 Yapi

访问 [http://localhost:3000](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A3000)   登录账号 **[admin@admin.com](https://links.jianshu.com/go?to=mailto%3Aadmin%40admin.com)**，密码 **ymfe.org**

![img](https:////upload-images.jianshu.io/upload_images/3424642-fb6bf5bf59a1f9da.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

![img](https:////upload-images.jianshu.io/upload_images/3424642-fcc0ad487cbb7c83.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

至此，帅气的 Yapi 就可以轻松使用啦！更多文档信息，请参考

- [Yapi 官方文档](https://links.jianshu.com/go?to=https%3A%2F%2Fyapi.ymfe.org%2Fdocuments%2Findex.html)
- [Yapi 版本更新记录](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FYMFE%2Fyapi%2Fblob%2Fmaster%2FCHANGELOG.md)

### ▶ 其他相关操作

**关闭 Yapi**

```undefined
docker stop yapi
```

**启动 Yapi**

```undefined
docker start yapi
```

**升级 Yapi**

```bash
# 1、停止并删除旧版容器
docker rm -f yapi

# 2、获取最新镜像
docker pull registry.cn-hangzhou.aliyuncs.com/anoy/yapi

# 3、启动新容器
docker run -d \
  --name yapi \
  --link mongo-yapi:mongo \
  --workdir /api/vendors \
  -p 3000:3000 \
  registry.cn-hangzhou.aliyuncs.com/anoy/yapi \
  server/app.js
```

