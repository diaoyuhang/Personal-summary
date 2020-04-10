# YApi搭建

## 一、安装node.js

> 安装nodejs

- 获取资源（部署nodejs尽可能选择偶数版本，因为偶数版本官方有较长的维护时间，故这次选择8.x。）
  curl -sL https://rpm.nodesource.com/setup_8.x | bash -
- 安装
  yum install -y nodejs
- 查看版本
  node -v
- 查看npm版本
  npm -v

## 二、安装mongodb

- 更新yum源

  yum -y update

- 添加mongodb源文件，在/etc/yum.repos.d 创建一个 mongodb-org.repo 文件
  touch /etc/yum.repos.d/mongodb-org.repo

- 编辑mongodb-org.repo文件
  vim /etc/yum.repos.d/mongodb-org.repo

- 添加文件内容

```shell
[mongodb-org]
name=MongoDB Repository
baseurl=http://mirrors.aliyun.com/mongodb/yum/redhat/7Server/mongodb-org/3.2/x86_64/
gpgcheck=0
enabled=1
```

- 安装mongodb
  yum install -y mongodb-org

- 启动mongodb
  service mongod start

- 设置开机启动
  chkconfig mongod on

- 配置远程访问,修改mongod.conf配置文件,vim /etc/mongod.conf

  注释 bindIp: 127.0.0.1==>\#bindIp: 127.0.0.1

- 重启mongod
  service mongod restart

## 三、安装YApi

- 准备环境搭建完成后，开始搭建YApi,安装命令
  npm install -g yapi-cli --registry https://registry.npm.taobao.org
  启动服务：yapi server
- 执行 yapi server 启动可视化部署程序，浏览器打开提示窗口上的地址，非本地服务器，将0.0.0.0替换指定的域名或IP，进入部署页面。
- ![](D:\0_LeargingSummary\YApi\images\yapi部署.png)
- 根据部署日志截图上的提示信息，启动服务
  启动服务：node vendors/server/app.js
  浏览器打开部署日志上的访问地址http://127.0.0.1:3000就可以访问搭建的YApi工具了（非本地服务器，将127.0.0.1替换指定的域名或IP），此时YApi环境搭建完成

![](D:\0_LeargingSummary\YApi\images\部署结果.png)