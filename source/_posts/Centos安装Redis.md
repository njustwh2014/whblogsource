---
title: Centos安装Redis
date: 2019-11-13 16:07:23
categories: Centos
# img: http://pzpoejx7j.bkt.clouddn.com/install-centos-7-logo.png
---

> 在CentOS和Red Hat系统中，首先添加EPEL仓库，然后更新yum源：

```bash
sudo yum install epel-release
sudo yum update
```
<!--more-->
> 然后安装redis
```bash
sudo yum -y install redis
```

> 为了可以使Redis能被远程连接，需要修改配置文件，路径为/etc/redis.conf
```bash
vi /etc/redis.conf
```
需要修改的地方：

首先，注释这一行：
```conf
#bind 127.0.0.1
```
另外，推荐给Redis设置密码，取消注释这一行：
```conf
#requirepass foobared
```
foobared即当前密码，可以自行修改为

```
requirepass 密码
```

> 常用命令
```bash
systemctl start redis.service #启动redis服务器

systemctl stop redis.service #停止redis服务器

systemctl restart redis.service #重新启动redis服务器

systemctl status redis.service #获取redis服务器的运行状态

systemctl enable redis.service #开机启动redis服务器

systemctl disable redis.service #开机禁用redis服务器
```