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
# 运行服务
docker run -it --name redis -v /root/wanghuan/docker/redis/cfg/redis.conf:/usr/local/etc/redis/redis.conf -v /root/wanghuan/docker/redis/data:/data -d -p 6379:6379 redis:latest /bin/bash

# 进入容器
docker exec -it redis bash
# 加载配置
redis-server /usr/local/etc/redis/redis.conf
# 测试连接
redis-cli -a wanghuan
```

> tips: redis desktop manager for free: https://github.com/qishibo/AnotherRedisDesktopManager/

### mysql安装部署备忘
mysql安装在130机器

```bash
# pull image
docker pull mysql:latest
# run
docker run --name mysql-mstc -v /root/wanghuan/docker/mysql/data:/data -e MYSQL_ROOT_PASSWORD=123456 -d -i -p 3306:3306 --restart=always  mysql:latest

# 导出sql文件
mysqldump -u root -p seumstc > F:/seumstc.sql
# docker容器内导入sql文件
# 进入docker容器
docker exec -it mysql-mstc bash
# 连接mysql
mysql -u root -p
# 新建数据库
create database seumstc;
use seumstc;
# 导入sql文件
source /data/seumstc.sql;
```
## 数据类型

> `new Integer(123)` 与 `Integer.valueOf(123)` 的区别在于

[参考](https://stackoverflow.com/questions/9030817/differences-between-new-integer123-integer-valueof123-and-just-123)

> 红黑树与平衡二叉树的区别

> [子字符串匹配kmp算法](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html) 

[leetcode 28](https://leetcode.com/problems/implement-strstr/)

[参考知乎实现](https://www.zhihu.com/question/21923021)
```java
/**
* @Description: kmp算法查找子字符串在字符串中匹配的位置 leetcode 28https://leetcode.com/problems/implement-strstr/
* @Param: [haystack, needle]
* @return: int
* @thorws:
* @Author: Mr.Wang
* @Date: 2020/2/5
*/
public int strStr(String haystack, String needle) {
    if(needle==null||needle.length()==0){
        return 0;
    }
    if(haystack==null||haystack.length()==0){
        return -1;
    }
    int[] maxLengthTable=maxLocalMatchLength(needle);
    int count=0;
    for(int i=0;i<haystack.length();i++){
        while(count>0&&needle.charAt(count)!=haystack.charAt(i)){
            count=maxLengthTable[count-1];
        }
        if(needle.charAt(count)==haystack.charAt(i)){
            count++;
        }
        if(count==needle.length()){
            return i-count+1;
        }
    }
    return -1;
}

private int[] maxLocalMatchLength(String pattern){
    if(pattern==null||pattern.length()==0){
        return null;
    }
    int[] maxLengthTable=new int[pattern.length()];
    int maxLength=0;
    for(int i=1;i<maxLengthTable.length;i++){
        while(maxLength>0&&pattern.charAt(maxLength)!=pattern.charAt(i)){
            maxLength=maxLengthTable[maxLength-1];
        }
        if(pattern.charAt(maxLength)==pattern.charAt(i)){
            maxLength++;
        }
        maxLengthTable[i]=maxLength;
    }
    return maxLengthTable;
}
```