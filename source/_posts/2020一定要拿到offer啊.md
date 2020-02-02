---
title: 2020一定要拿到offer啊
date: 2020-02-02 11:06:02
tags: 
    - Java 
    - Redis
    - Mysql
    - Vue
    - Springboot
categories: 求职
---
# 学习计划

> 将之前博客系统重启
+ redis、mysql部署在130虚拟机

+ 前端部署在129机器上(nginx代理)

+ 后端部署在131虚拟机

> leetcode刷题

## 网站重新部署和优化工作

### redis安装部署备忘

redis安装在130机器

```bash
# 运行服务for windows
docker run -it --name redis -v F:/codeHub/docker/redis/redis.conf:/usr/local/etc/redis/redis.conf -v F:/codeHub/docker/redis/data:/data -d -p 6379:6379 redis:latest /bin/bash

# 运行服务 
docker run -it --name redis -v /home/huanhuan/myweb/redisdata/redis.conf:/usr/local/etc/redis/redis.conf -v /home/huanhuan/myweb/redisdata/data:/data -d -p 6379:6379 redis:latest /bin/bash
# 进入容器
docker exec -it redis bash
# 加载配置
redis-server /usr/local/etc/redis/redis.conf
# 测试连接
redis-cli -a wanghuan
```


## 数据类型

> `new Integer(123)` 与 `Integer.valueOf(123)` 的区别在于

[参考](https://stackoverflow.com/questions/9030817/differences-between-new-integer123-integer-valueof123-and-just-123)

> 红黑树与平衡二叉树的区别