---
title: docker中使用grafana
date: 2019-11-28 13:54:07
tags:
    - docker
    - grafana
categories: docker
---

## 查看grafana镜像

```bash
docker search grafana
```

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
