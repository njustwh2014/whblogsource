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