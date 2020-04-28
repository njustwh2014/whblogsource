---
title: 2020一定要拿到offer啊
date: 2020-02-02 11:06:02
tags: [Java,Redis,Mysql,Vue,Springboot]
categories: 求职
---
# 学习计划

> 将之前博客系统重启
+ redis、mysql部署在130虚拟机

+ 前端部署在129机器上(nginx代理)

+ 后端部署在131虚拟机

> leetcode刷题
<!--more-->

## 网站重新部署和优化工作

> [docker-compose.yml详解](https://www.jianshu.com/p/2217cfed29d7)

> [docker网络模式](https://juejin.im/post/5c3363bf6fb9a049e2322cdb)

> [servlet+idea环境配置参考](https://www.cnblogs.com/javabg/p/7976977.html)

> [centos上安装mysql](https://www.cnblogs.com/starof/p/4680083.html) 

> [centos上部署springboot项目](https://www.cnblogs.com/zuidongfeng/p/8859229.html)

> [centos上配置fdfs](https://blog.csdn.net/chen3888015/article/details/70172505/)

> [微笑大神： 使用fdfs](http://www.ityouknow.com/springboot/2018/01/16/spring-boot-fastdfs.html)

> [HTTP请求不能访问文件的原因](https://blog.csdn.net/qq_34301871/article/details/80060235)

> [出现缺少fastcommon.h修改办法]() https://blog.csdn.net/babydavic/article/details/21240021)

> [nginx安装](https://blog.csdn.net/qq_36922927/article/details/79554806)

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

> list与set的区别

list与set均继承自Collections接口。

**list：** 有序，可重复，可动态扩展，查找效率高，但插入删除效率低，支持for循环和迭代器。

**set：** 无序，不重复，位置固定，由hashcode决定，加入set的object必须实现equals方法，插入删除效率高，检索效率低下，插入删除不会改变其他元素位置。set只能使用迭代器。


> HashSet如何保证元素不重复

HashSet使用add()方法时调用HashMap的add()方法。判断元素是否存在时除了使用hashcode还需要调用equals()方法。HashMap的key是唯一，HashSet加入的Object作为HashMap的key。

以下是HashSet的部分源码：
```java
private static final Object PRESENT=new Object();
private transient HashMap<E,Object> map;
public HashSet(){
    map=new HashMap<>();
}
public boolean add(E e){
    return map.put(e,PRESENT)==null;
}
```

> HashMap线程不安全


## JVM

### 1. JVM上堆和栈的比较

[参考文章](https://iamjohnnyzhuang.github.io/java/2016/07/12/Java%E5%A0%86%E5%92%8C%E6%A0%88%E7%9C%8B%E8%BF%99%E7%AF%87%E5%B0%B1%E5%A4%9F.html)

### 2. JVM运行时数据区域

> 栈区：栈区需要内存连续

 + 程序计数器：记录正在执行的虚拟机字节码指令地址(如果是本地方法则为空)

 + Java虚拟机栈：每一个Java方法在执行的同时会创建一个栈帧用于存放**局部变量表、操作数栈、常量池引用**等信息，方法的调用直至执行完成对应着栈帧入栈和出栈过程。

    - 可以通过`-Xss`这个参数来描述Java虚拟机栈的内存大小，JDK1.4默认256k，而JDK1.5+默认为1M
    ```
    java -Xss2M HackTheJava
    ```

    - 该区域可能抛出如下异常：

        + 线程请求的栈深度超过最大值，抛出`StackOverflowError`

        + 栈进行动态扩展时无法申请到足够内存时，抛出`OutOfMemoryError`

+ 本地方法栈：本地方法栈与 Java 虚拟机栈类似，它们之间的区别只不过是本地方法栈为本地方法服务。

本地方法一般是用其它语言（C、C++ 或汇编语言等）编写的，并且被编译为基于本机硬件和操作系统的程序，对待这些方法需要特别处理。

> 堆区：堆区不需要连续的内存，可动态增加内存，增加内存失败会抛出`OutOfMemoryError`

所有的对象都在堆区分配内存，是垃圾收集的主要区域(GC堆)

现在的垃圾收集器都采用分代收集算法，其主要思想采用不同类型的对象采用不同的垃圾收集算法，可以将堆分成两块：

    + 新生代

    + 老年代

可以通过 `-Xms` 和 `-Xmx` 这两个虚拟机参数来指定一个程序的堆内存大小，第一个参数设置初始值，第二个参数设置最大值。

```
java -Xms1M -Xmx2M HackTheJava
```

> 方法区

用于存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

和**堆一样不需要连续的内存**，并且可以动态扩展，动态扩展失败一样会抛出 `OutOfMemoryError` 异常。

对这块区域进行垃圾回收的主要目标是对常量池的回收和对类的卸载，但是一般比较难实现。

HotSpot虚拟机把它当成永久代来进行垃圾回收。但很难确定永久代的大小，因为它受到很多因素影响，并且每次 Full GC 之后永久代的大小都会改变，所以经常会抛出`OutOfMemoryError`异常。为了更容易管理方法区，从 JDK 1.8 开始，移除永久代，并把方法区移至元空间(metaspace)，它位于本地内存中，而不是虚拟机内存中。

方法区是一个JVM规范，永久代与元空间都是其一种实现方式。在 JDK 1.8 之后，原来永久代的数据被分到了堆和元空间中。元空间存储类的元信息，静态变量和常量池等放入堆中。

  + 运行时常量池

    **运行时常量池是方法区的一部分**。

    Class 文件中的常量池（编译器生成的字面量和符号引用）会在类加载后被放入这个区域。

    除了在编译期生成的常量，还允许动态生成，例如 String 类的 intern()。

> 直接内存

在 JDK 1.4 中新引入了 NIO 类，它可以使用 Native 函数库直接分配堆外内存，然后通过 Java 堆里的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在堆内存和堆外内存来回拷贝数据。

## 计算机网络

### 1. 从输入URL到页面加载的详细过程

[参考文章](https://segmentfault.com/a/1190000006879700)



## Redis
### 1. Redis是单线程吗？为什么Redis如此之快？

redis单线程指的是网络请求模块使用了一个线程，即一个线程处理所有网络请求，其他模块仍用了多个线程。

因为CPU不是Redis的瓶颈。Redis的瓶颈最有可能是机器内存或者网络带宽，既然单线程容易实现，而且CPU不会成为瓶颈，那就顺理成章地采用单线程的方案了。关于redis的性能，官方网站也有，普通笔记本轻松处理每秒几十万的请求。

> redis为什么如此之快？

+ 完全基于内存，绝大部分请求是纯粹的内存操作，非常快速。数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)；

+ 数据结构简单，对数据操作也简单，Redis中的数据结构是专门进行设计的；

+ 采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗；

+ 使用多路I/O复用模型，非阻塞IO；[参考](https://draveness.me/redis-io-multiplexing)

+ 使用底层模型不同，它们之间底层实现方式以及与客户端之间通信的应用协议不一样，Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求；

> 多路I/O复用模型，非阻塞IO

下面举一个例子，模拟一个tcp服务器处理30个客户socket。
假设你是一个监考老师，让30个学生解答一道竞赛考题，然后负责验收学生答卷，你有下面几个选择：

第一种选择：按顺序逐个验收，先验收A，然后是B，之后是C、D。。。这中间如果有一个学生卡住，全班都会被耽误。
这种模式就好比，你用循环挨个处理socket，根本不具有并发能力。

第二种选择：你创建30个分身，每个分身检查一个学生的答案是否正确。 这种类似于为每一个用户创建一个进程或者线程处理连接。

第三种选择，你站在讲台上等，谁解答完谁举手。这时C、D举手，表示他们解答问题完毕，你下去依次检查C、D的答案，然后继续回到讲台上等。此时E、A又举手，然后去处理E和A。。。

这种就是IO复用模型，Linux下的select、poll和epoll就是干这个的。将用户socket对应的fd注册进epoll，然后epoll帮你监听哪些socket上有消息到达，这样就避免了大量的无用操作。此时的socket应该采用非阻塞模式。

这样，整个过程只在调用select、poll、epoll这些调用的时候才会阻塞，收发客户消息是不会阻塞的，整个进程或者线程就被充分利用起来，这就是事件驱动，所谓的reactor模式。

> 针对上面距离，在redis中表现为：

有30个redis客户端（考生）与redis服务器的网络连接模块（监考老师）保持TCP连接，客户端会不定时的发送请求给服务器，当有一个redis客户端发起请求，会触发unix系统像epoll这样的系统调用，Redis的I/O 多路复用模块封装了底层的epoll这样的 I/O 多路复用函数，然后转发到相应的事件处理器。

文件事件处理器使用 I/O 多路复用模块同时监听多个 FD（文件描述符），当 accept、read、write 和 close 文件事件产生时，文件事件处理器就会回调 FD 绑定的事件处理器。

虽然整个文件事件处理器是在单线程上运行的，但是通过 I/O 多路复用模块的引入，实现了同时对多个 FD 读写的监控，提高了网络通信模型的性能，同时也可以保证整个 Redis 服务实现的简单。

### 2. redis可能出现的问题：

+ 缓存雪崩
  
+ 缓存穿透
  
+ 数据库和缓存双写一致性问题

### 3. Redis持久化方式有哪些？并比较

### 4.Redis过期回收策略和内存淘汰机制

### 5.redis主从复制机制

> 分布式系统的CAP理论

### 6.redis对事务的支持

### 7.redis的哨兵模式

### 8.redis实现分布式锁

## git

### 1. fast-forward

### 2. merge与rebase区别


