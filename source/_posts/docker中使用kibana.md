---
title: docker中使用kibana
date: 2019-11-22 14:38:21
categories: docker
---

## 拉取镜像
```bash
docker pull docker.elastic.co/kibana/kibana:6.5.2
docker images
```
<!--more-->
## 启动镜像
```bash
docker run -it -d -e ELASTICSEARCH_URL=http://127.0.0.1:9200 --name kibana --network=container:elasticsearch docker.elastic.co/kibana/kibana:6.5.2
```
> --network 指定容器共享elasticsearch容器的网络栈 (使用了--network 就不能使用-p 来暴露端口)