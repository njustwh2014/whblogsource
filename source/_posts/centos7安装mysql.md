---
title: centos7安装mysql
date: 2019-11-11 15:36:25
categories: Centos
# img: http://pzpoejx7j.bkt.clouddn.com/install-centos-7-logo.png
---

> 系统环境
```bash
Centos 7.6 64位
```

## 添加mysql yum源

在centOS上直接使用yum install mysql安装，最后安装上的会是MariaDB，所以要先添加mysql yum源。
<!--more-->
```bash
rpm -Uvh https://repo.mysql.com//mysql80-community-release-el7-2.noarch.rpm
```

## 安装

> 查看yum源中mysql版本
```bash
yum repolist all | grep mysql
```

> 禁用最新版本mysql8.0
```bash
yum-config-manager --disable mysql80-community
```

> 启用我们需要的mysql5.7
```bash
yum-config-manager --enable mysql57-community
```

> 检查刚刚配置是否生效
```bash
yum repolist enabled | grep mysql
```

> 开始安装
```bash
yum install mysql-community-server
```

## 启动服务

> 启动mysql
```bash
service mysqld start
```

> 查看mysql启动状态
```bash
service mysqld status
```

> 设置开机自启动(optional)
```bash
service mysqld enable
service mysqld restart
```

> 初次登陆需要临时密码
```bash
grep 'temporary password' /var/log/mysqld.log
```

> 登录
```bash
mysql -u root -p
```

## 修改密码

> 修改密码要求

login mysql first.

```sql
set global validate_password_policy=0; 
set global validate_password_length=1;
```

> 修改密码
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```

## 授权其他机器登陆
```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
FLUSH  PRIVILEGES;
```
## mysql导入导出sql文件

> 导出整个数据库
```bash
mysqldump -u 用户名 -p 数据库名 > 导出的文件名
mysqldump -u dbuser -p dbname > dbname.sql
```

> 导出一个表
```bash
mysqldump -u 用户名 -p 数据库名 表名> 导出的文件名
mysqldump -u dbuser -p dbname users> dbname_users.sql
```
> 导出一个数据库结构
```bash
mysqldump -u dbuser -p -d --add-drop-table dbname >d:/dbname_db.sql
-d 没有数据 --add-drop-table 在每个create语句之前增加一个drop table
```

> 导入数据库
常用source 命令
进入mysql数据库控制台，如
```bash
mysql -u root -p
mysql>use 数据库
```
然后使用source命令，后面参数为脚本文件(如这里用到的.sql)
```bash
mysql>source d:/dbname.sql
```
