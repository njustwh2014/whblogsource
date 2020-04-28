---
title: zookeeper????
date: 2020-04-04 15:23:06
tags:
    - Zookeeper
    - Docker Compose
    - Docker
categories: Zookeeper
---

## Docker Compose????

???????`zookeeper-docker`????`docker-compose.yml`???

```yml
version: '3.1'

services:
  zoo1:
    image: zookeeper
    restart: always
    hostname: zoo1
    networks:
      - zoo-net
    ports:
      - 2181:2181
    volumes:
      - zoo1-data:/data
      - zoo1-log:/datalog
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
      ZOO_4LW_COMMANDS_WHITELIST: "*"

  zoo2:
    image: zookeeper
    restart: always
    hostname: zoo2
    networks:
      - zoo-net
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=0.0.0.0:2888:3888;2181 server.3=zoo3:2888:3888;2181
      ZOO_4LW_COMMANDS_WHITELIST: "*"

  zoo3:
    image: zookeeper
    restart: always
    hostname: zoo3
    networks:
      - zoo-net
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=0.0.0.0:2888:3888;2181
      ZOO_4LW_COMMANDS_WHITELIST: "*"

volumes:
  zoo1-data:
    external: false # ?????true????????Compose????? docker-compose up?????????????????????????
  zoo1-log:
    external: false  

networks:
  zoo-net:
```

???????????????

```bash
docker run -it --rm \
    --link zoo1:zk1 \
    --link zoo2:zk2 \
    --link zoo3:zk3 \
    --net zk_cluster_zoo-net \
    zookeeper zkCli.sh -server zk1:2181,zk2:2181,zk3:2181
```

????zk_cluster_zoo-net????{project_name}_{network_name}?project_name????????network_name???docker-compose.yml???????docker-compose??????????????????docker network ls????

```bash
[root@services zookeeper-docker]# docker network ls
NETWORK ID          NAME                       DRIVER              SCOPE
7b230cc98274        bridge                     bridge              local
983d7757ab51        docker-compose_default     bridge              local
323230c6409a        elasticsearch_default      bridge              local
68513b9f627d        ftp-service_default        bridge              local
d42cd7951781        harbor_harbor              bridge              local
9180b58093e4        host                       host                local
c624c844f1d8        nexus_default              bridge              local
a88ccf0e0508        none                       null                local
d27663727c82        zookeeper-docker_zoo-net   bridge              local 
```

????`nc`????

```bash
[root@services zookeeper-docker]# echo stat | nc 127.0.0.1 2181
Zookeeper version: 3.6.0--b4c89dc7f6083829e18fae6e446907ae0b1f22d7, built on 02/25/2020 14:38 GMT
Clients:
 /192.168.160.1:35724[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 1/9.75/33
Received: 5
Sent: 4
Connections: 1
Outstanding: 0
Zxid: 0x100000002
Mode: follower
Node count: 5
[root@services zookeeper-docker]# echo stat | nc 127.0.0.1 2182
Zookeeper version: 3.6.0--b4c89dc7f6083829e18fae6e446907ae0b1f22d7, built on 02/25/2020 14:38 GMT
Clients:
 /192.168.160.1:59584[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0.0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x100000002
Mode: follower
Node count: 5
[root@services zookeeper-docker]# echo stat | nc 127.0.0.1 2183
Zookeeper version: 3.6.0--b4c89dc7f6083829e18fae6e446907ae0b1f22d7, built on 02/25/2020 14:38 GMT
Clients:
 /192.168.160.1:42524[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0.0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x100000002
Mode: leader
Node count: 5
Proposal sizes last/min/max: 48/48/48
```
## Zookeeper????

## Zookeeper????

## Zookeeper????