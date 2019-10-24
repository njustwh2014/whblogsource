---
title: 我的网站开发经历与Java学习心路历程
date: 2019-10-22 10:36:59
img: http://pzpoejx7j.bkt.clouddn.com/image/javajava.png
top: true
cover: true
coverImg: http://pzpoejx7j.bkt.clouddn.com/image/javajava.png
toc: false
mathjax: false
summary: 俱乐部网站以及个人博客网站介绍，以及从零开始接触Java的学习心路历程。
categories: Java
tags:
  - Java
  - Spring
  - JavaScript
  - Vue
  - Redis
  - Mysql
---

## 网站介绍
### 功能介绍
### 什么是前后端分离的开发模式
### 开发这样一个网站我们需要的技术栈
## Java学习历程
![Java后端学习路线](我的网站开发经历与Java学习心路历程/javaweblearningroute.png)

学习资源：
> https://github.com/Snailclimb/JavaGuide
>
> https://github.com/CL0610/Java-concurrency

### Java基础
![Java基础思维导图](我的网站开发经历与Java学习心路历程/JavaBase.jpg)

**书本推荐：**《Java核心技术》《Java编程思想》 建议先看核心技术

Java作为一门强类型语言，我们学习Java需要首先明确**面向对象**的概念，可以理解以下几个概念：**封装、继承、多态**等，并学会灵活运用。此外还可以学习一些Java语言特性，如泛型、反射等。这些特性广泛运用于一些框架中，如Spring的Aop和Ioc均使用了Java的反射特性，然而我们知道，原生Java反射效率不高，具体原因我们后面专门写博客分析。

其次，就是要学习一些Java的标准库，如集合、IO、String等等.

学习中可以采用这样一个模式：
+ 理解概念
+ 找相关demo学习
+ 动手复现
+ 总结：效率、扩展性等方面

### 数据结构、基础算法与设计模式

**推荐书籍：**《剑指Offer》《修炼Java开发技术》《Effective Java》

数据结构的学习是学任何一门编程语言的基础，在学完基础语法以后就需要深入学习数据结构。只要掌握好数据结构才能结合算法写出效率与鲁棒性俱佳的代码。

对于这部分的学习，没有更多方法就是多刷题，多总结。我后续会在博客网站写相关内容。

### JVM以及操作系统
**推荐书籍：** 《深入理解Java虚拟机》

**一定要看**，重点关注JMM(Java内存模型)，GC，还需要了解一些Java虚拟机指令，方便调试。

### 数据库
这部分我没有书籍推荐，我个人觉得是要自己安装并使用才是最重要的。

对于数据库你需要理解
> 什么是关系型数据库(Mysql, Orcal sql etc.)及其优缺点

> 什么是非关系型数据库(Redis, MongoDB etc.)及其优缺点

> Sql与NoSql的应用场景

> 数据库运维基础

> 对于关系型数据库，怎么提升其性能

> 非关系数据库的备份与同步问题

> Java操作数据库的API(如对于Mysql的Mybatis，对于Redis的Jedis)
### 框架学习
Java的一大优势就是拥有很多优秀的开源与非开源的框架。

![](http://pzpoejx7j.bkt.clouddn.com/JustCopyIt.jpg)

但是首推自然是Spring全家桶。


### 计算机网络与Web基础
### 一些工具
 + git
 + docker
### 前端基础
### 中间件
### 一些测试框架
### 学无止境，寻找Java替代品
