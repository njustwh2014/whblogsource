---
title: docker中使用grafana
date: 2019-11-28 13:54:07
categories: docker
---

## 查看grafana镜像

```bash
docker search grafana
```
<!--more-->
## 拉取grafana镜像
```bash
docker pull grafana/grafana:latest
```

## 运行镜像
```bash
docker run -d -p 3000:3000 --name=grafana -v D:/grafana:/var/lib/grafana grafana/grafana
docker logs -f grafana
```

> 默认账户密码:admin/admin


> 第一次登陆后我修改为:admin/trend#1..


## grafana容器访问elasticsearch容器

> 查看elasticsearch容器ip
```bash
docker inspect --format={{.NetworkSettings.IPAddress}} elasticsearch
```

result:
```bash
172.17.0.2
```

> [关于docker的网络类型](https://www.jianshu.com/p/0ded8d810860)

> 安装后一些目录作用
```bash
#主配置文件
/etc/grafana/grafana.ini
#数据文件
/var/lib/grafana
#home目录
/usr/share/grafana
#日志目录
/var/log/grafana
#插件目录
/var/lib/grafana/plugins
#自定义一些精细化配置的文件夹
/etc/grafana/provisioning
```

[配置参考](https://www.li-rui.top/2018/12/12/monitor/grafana%E4%BD%BF%E7%94%A8/)