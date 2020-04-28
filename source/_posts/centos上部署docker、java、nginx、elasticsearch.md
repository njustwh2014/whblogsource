---
title: centos上部署docker、java、nginx、elasticsearch
date: 2019-12-05 10:00:46
categories: Centos
---

## 安装centos6
```bash
#查看版本信息
cat /etc/redhat-release
```
> [版本信息参考链接](https://blog.csdn.net/shuaigexiaobo/article/details/78030008)

**开启相应端口**
<!--more-->
## centos中安装docker

```bash
#查看你当前的内核版本
uname -r

#更新yum源
yum update

#安装 Docker
yum -y install docker

#启动 Docker 后台服务
service docker start

#测试运行 hello-world,由于本地没有hello-world这个镜像，所以会下载一个hello-world的镜像，并在容器内运行z
docker run hello-world
```

> [docker常用命令参考链接](https://segmentfault.com/a/1190000012063374)

## docker中安装elasticsearch

```bash
# 拉取镜像
docker pull elasticsearch:6.5.0
# 查看镜像
docker images
# 启动镜像
docker run -d  -p 9300:9300 -p 9200:9200 -p 5601:5601 --name elasticsearch -v /home/huanhuan/dockerdata/elasticsearch:/data elasticsearch:6.5.0
# 查看日志
docker logs -f elasticsearch
```
> 错误max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```bash
# 在宿主机内修改
echo "vm.max_map_count=262144" > /etc/sysctl.conf
sysctl -p
```

> [centos7常见elasticsearch错误解决方法](https://www.jianshu.com/p/04f4d7b4a1d3)
## docker中安装kibana

本来打算用nginx代理访问，结果可以直接访问，应该是docker的kibana镜像内置了代理访问。

```bash
# 拉取镜像
docker pull docker.elastic.co/kibana/kibana:6.5.2
# 查看镜像
docker images
# 启动镜像
docker run -it -d -e ELASTICSEARCH_URL=http://127.0.0.1:9200 --name kibana --network=container:elasticsearch docker.elastic.co/kibana/kibana:6.5.2
```

## docker中安装grafana
```bash
#拉取镜像
docker pull grafana/grafana:latest
#启动镜像
docker run -d -p 3000:3000 --name=grafana  grafana/grafana
#查看日志
docker logs -f grafana
#查看容器ip
docker inspect --format={{.NetworkSettings.IPAddress}} elasticsearch
```
> 默认账户密码:admin/admin

> 第一次登陆后我修改为:admin/trend#1..

## docker中安装nginx
```bash
# 拉取镜像
docker pull nginx
# 启动镜像
docker run -d -p 9528:9528 -p 9529:9529 -p 9530:9530 --name nginx --privileged=true -v /home/huanhuan/dockerdata/nginx/element-admin:/home/huanhuan/dockerdata/nginx/element-admin -v /home/huanhuan/dockerdata/nginx/images:/home/huanhuan/dockerdata/nginx/images -v /home/huanhuan/dockerdata/nginx/conf/nginx.conf:/etc/nginx/nginx.conf  nginx
```
> --privileged=true 解决docker读取宿主机文件的权限


