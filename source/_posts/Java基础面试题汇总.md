---
title: Java基础面试题汇总
date: 2020-03-13 20:32:50
tags: 
    - Java 
    - Springboot
    - JVM
categories: 求职
---

## Java基础

### 线程间同步方式

[看看看看](https://www.cnblogs.com/xhjt/p/3897440.html)

+ 1.`synchronized`修饰方法和方法块
+ 2.使用特殊域变量`volatile`
 - volatile关键字为域变量的访问提供了一种免锁机制，
 - 使用volatile修饰域相当于告诉虚拟机该域可能会被其他线程更新，
 - 因此每次使用该域就要重新计算，而不是使用寄存器中的值
 - volatile不会提供任何原子操作，它也不能用来修饰final类型的变量
+ 3.使用重入锁`RetrentLock`
  
  在JavaSE5.0中新增了一个java.util.concurrent包来支持同步。ReentrantLock类是可重入、互斥、实现了Lock接口的锁，它与使用synchronized方法和快具有相同的基本行为和语义，并且扩展了其能力
  
  ReenreantLock类的常用方法有：
   - ReentrantLock() : 创建一个ReentrantLock实例
   - lock() : 获得锁
   - unlock() : 释放锁

  注：ReentrantLock()还有一个可以创建公平锁的构造方法，但由于能大幅度降低程序运行效率，不推荐使用
+ 4.使用`ThreadLocal`
  如果使用ThreadLocal管理变量，则每一个使用该变量的线程都获得该变量的副本，副本之间相互独立，这样每一个线程都可以随意修改自己的变量副本，而不会对其他线程产生影响。

  ThreadLocal与同步机制:
   - a.ThreadLocal与同步机制都是为了解决多线程中相同变量的访问冲突问题。
   - b.前者采用以"空间换时间"的方法，后者采用以"时间换空间"的方式

+ 5.[使用阻塞队列`LinkedBlockingQueue`](https://www.jianshu.com/p/9394b257fdde)

 - (1) LinkedBlockingQueue继承于AbstractQueue，它本质上是一个FIFO(先进先出)的队列。
 - (2) LinkedBlockingQueue实现了BlockingQueue接口，它支持多线程并发。当多线程竞争同一个资源时，某线程获取到该资源之后，其它线程需要阻塞等待。
 - (3) LinkedBlockingQueue是通过**单链表**实现的。
   + a) head是链表的表头。取出数据时，都是从表头head处插入。
   + b) last是链表的表尾。新增数据时，都是从表尾last处插入。
   + c) count是链表的实际大小，即当前链表中包含的节点个数。
   + d) capacity是列表的容量，它是在创建链表时指定的。
   + e) putLock是插入锁，takeLock是取出锁；notEmpty是“非空条件”，notFull是“未满条件”。通过它们对链表进行并发控制。

 LinkedBlockingQueue在实现“多线程对竞争资源的互斥访问”时，对于“插入”和“取出(删除)”操作分别使用了不同的锁。对于插入操作，通过“插入锁putLock”进行同步；对于取出操作，通过“取出锁takeLock”进行同步。 此外，插入锁putLock和“非满条件notFull”相关联，取出锁takeLock和“非空条件notEmpty”相关联。通过notFull和notEmpty更细腻的控制锁。
 
  - 若某线程(线程A)要取出数据时，队列正好为空，则该线程会执行notEmpty.await()进行等待；当其它某个线程(线程B)向队列中插入了数据之后，会调用notEmpty.signal()唤醒“notEmpty上的等待线程”。此时，线程A会被唤醒从而得以继续运行。 此外，线程A在执行取操作前，会获取takeLock，在取操作执行完毕再释放takeLock。
  - 若某线程(线程H)要插入数据时，队列已满，则该线程会它执行notFull.await()进行等待；当其它某个线程(线程I)取出数据之后，会调用notFull.signal()唤醒“notFull上的等待线程”。此时，线程H就会被唤醒从而得以继续运行。 此外，线程H在执行插入操作前，会获取putLock，在插入操作执行完毕才释放putLock。
     
+ 6.使用原子变量
  需要使用线程同步的根本原因在于对普通变量的操作不是原子的。

  那么什么是原子操作呢？
  
  原子操作就是指将读取变量值、修改变量值、保存变量值看成一个整体来操作即这几种行为要么同时完成，要么都不完成。

### 创建线程的几种方式

+ 1.实现Runnable接口
+ 2.实现Callable接口：带返回值 FutureTask
+ 3.继承Thread类：由于一般类都有继承，这种方式代价太高，导致类无法继承

### 线程池原理及参数含义

#### 线程池好处

+ 降低资源消耗：可以利用已创建的线程，降低创建线程和消耗线程带来的资源消耗
+ 提高响应速度：当请求到达时，可以不需要重新创建线程而直接执行
+ 提高线程可管理性：线程是类稀缺资源，无限制地创建线程会消耗过多资源并且会造成系统不稳定，利用线程池可以对线程进行统一的**分配、调优和监控**

#### 线程池工作原理(核心线程->等待队列->线程池是否已满->饱和策略)

+ 1.首先判断核心线程是否均在工作，如果均在工作则进入第2步，如果未均在工作，则创建一个新的线程来执行任务
+ 2.当核心线程均在执行任务时，对于新的任务则判断等待队列是否已满，如果满了则执行第3步，否则，将任务放入等待队列
+ 3.判断线程池线程是否均在执行任务(注意此时是所有线程而不只是核心线程)，如果均在执行任务，则交给饱和策略来处理任务，否则创建线程执行任务。

#### 线程池饱和策略

+ 1.AbortPolicy：为java线程池默认的阻塞策略，不执行此任务，而且直接抛出一个运行时异常，切记ThreadPoolExecutor.execute需要try catch，否则程序会直接退出。
+ 2.DiscardPolicy：直接抛弃，任务不执行，空方法
+ 3.DiscardOldestPolicy：从队列里面抛弃head的一个任务，并再次execute 此task。
+ 4.CallerRunsPolicy：在调用execute的线程里面执行此command，会阻塞入口 
+ 5.用户自定义拒绝策略（最常用）：实现RejectedExecutionHandler，并自己定义策略模式

[看看源码吧！！](https://juejin.im/post/5c33400c6fb9a049fe35503b)

### 死锁产生条件以及解决策略

#### 死锁产生的四个条件

+ 1. 互斥条件：一个资源每次只能被一个线程持有
+ 2. 请求与保持条件：一个进程因请求资源而等待时，不会释放已分配的资源。
+ 3. 不可剥夺条件：线程持有的资源，在未使用前不可剥夺
+ 4. 循环等待条件：若干个线程形成头尾相连的等待资源的关系

只要产生死锁，这四个条件必定成立，若破坏其中一个条件，死锁就不会发生了。

#### 解决死锁的策略

+ **预防死锁**
 - **资源一次性分配**：破坏请求和保持条件。　当某个资源只在进程结束时使用一小会，那么在进程运行期间，这个资源都被占用，资源利用率很低。比较好的方法是，进程开始时，只申请和使用进程启动的资源，在运行过程中不断申请新的资源，同时释放已经使用完的资源。
 - **可剥夺资源**：当进程新申请的资源不满足时，释放已经分配的资源。破坏不可剥夺条件。　在使用某些资源，比如打印机时，当强制剥夺已分配资源的时候，会导致打印机资源打印的信息不连续的问题。
 - **资源有序分配**：系统给进程编号，按某一顺序申请资源，释放资源则反序释放。破坏循环等待条件。
+ **避免死锁**
 **银行家算法**：分配资源前先评估风险，会不会在分配后导致死锁。　即分配给一个进程资源的时候，该进程能否全部返还占用的资源。
+ **检测死锁**
 建立资源分配表和进程等待表。
+ **解除死锁**
 从其他进程强制剥夺资源给死锁进程。可以直接撤销死锁进程，或撤销代价最小的进程。

### JUC下源码分析(偶尔带着看吧)

### Lock接口下实现的锁和`synchronized`关键字比较，各自优缺点

### AQS 

### 为什么要重写hashCode和equals方法

如果不重写，则会默认调用Object类的hashCode和equals方法，Object类的hashCode方法是根据对象内存首地址计算得到，equals方法也同样是根据对象内存首地址进行判断。这在很多场景下是不可用的。

### hashCode与equals区别

+ 规范1：若重写equals(Object obj)方法，有必要重写hashcode()方法，确保通过equals(Object obj)方法判断结果为true的两个对象具备相等的hashcode()返回值。说得简单点就是：“如果两个对象相同，那么他们的hashcode应该 相等”。不过请注意：这个只是规范，如果你非要写一个类让equals(Object obj)返回true而hashcode()返回两个不相等的值，编译和运行都是不会报错的。不过这样违反了Java规范，程序也就埋下了BUG。 
+ 规范2：如果equals(Object obj)返回false，即两个对象“不相同”，并不要求对这两个对象调用hashcode()方法得到两个不相同的数。说的简单点就是：“如果两个对象不相同，他们的hashcode可能相同”。 
根据这两个规范，可以得到如下推论： 
 - 1、如果两个对象equals，Java运行时环境会认为他们的hashcode一定相等。 
 - 2、如果两个对象不equals，他们的hashcode有可能相等。 
 - 3、如果两个对象hashcode相等，他们不一定equals。 
 - 4、如果两个对象hashcode不相等，他们一定不equals。 

### 集合类:HashMap和HashTable、ConcurrentHashMap(源码比较) **重点**

> HashMap是非线程安全，HashTable和ConcurrentHashMap是线程安全

> HashMap与HashTable比较

不同点 |HashMap|HashTable
---- | ---- | ----
实现原理 | 继承自AbstractMap类 | 继承自Dictionary(JDK1.0添加)没用过
初始化容量不同(二者负载因子均为0.75)|初始容量为16|初始容量为11
扩容机制不同(当前容量大于总容量*负载因子)|总容量翻倍(保持始终为2的幂)|总容量翻倍+1
迭代器不同|使用Iterator迭代器，是fail-fast的|使用Enumerator，不是fail-fast的

> HashMap底层原理 关键点

+ JDK1.8引入数组加链表+红黑树
+ 链表长度超过8时转成红黑树

> ConcurrentHashMap

+ JDK1.7 有segment分段锁
+ JDK1.8 放弃分段锁[为何？](https://cloud.tencent.com/developer/article/1509556)
+ CAS和synchronied
+ get和put操作



[看这篇屁话](https://mp.weixin.qq.com/s/AixdbEiXf3KfE724kg2YIw)

JDK1.7和JDK1.8对于二者改动及原因


### ArrayList、LinkedList、TreeMap、LinkedHashMap、HashSet

![Java集合框架图](Java基础面试题汇总/Java集合框架图.jpg)

需要了解底层数据结构和各容器之间的优劣比较

### 讲一下HashMap的原理，如果链表过长怎么办，如果想让Map按照put的顺序存放键值对应该使用什么类？

HashMap在存在hash冲突的情况下，会使用链表或红黑树存储hash冲突的key，一般当链表长度超过`TREEIFY_THRESHOLD`会将链表转成红黑树。

Map按照元素大小排序用TreeMap

Map按照元素put顺序用LinkedHashMap


### Synchronized实现原理，原子操作实现原理

[参考文章](https://juejin.im/post/5cee325e6fb9a07ee85c0c49)

**jdk对synchronized做了哪些优化：**

+ 适应性自旋锁：为了减少线程状态的改变带来的消耗，不停地执行当前线程

+ 锁消除：对于不可能存在共享资源竞争的锁进行消除

+ 锁粗化：对于连续多次加锁，精简到只有一次。

+ 轻量级锁：无竞争条件下，通过CAS消除同步互斥

+ 偏向锁：无竞争条件下消除同步互斥，连CAS操作都不需要

原子操作实现原理：总线锁 缓存锁 CAS

那么**CAS不足之处**：

+ ABA

+ 循环时间长会导致消耗大

+ 只能保证一个共享变量原子操作

### Session与Cookie区别

session是存储在服务器端,cookie是存储在客户端的,所以安全来讲session的安全性要比cookie高,然后我们获取session里的信息是通过存放在**会话cookie**里的sessionid获取的。又由于session是存放在服务器的内存中,所以session里的东西不断增加会造成服务器的负担,所以会把很重要的信息存储在session中,而把一些次要东西存储在客户端的cookie里。

cookie确切的说分为两大类分为**会话cookie和持久化cookie**,会话cookie确切的说是存放在客户端浏览器的内存中,所以说他的生命周期和浏览器是一致的,浏览器关了会话cookie也就消失了,然而持久化cookie是存放在客户端硬盘中,而持久化cookie的生命周期就是我们在设置cookie时候设置的那个保存时间,然后我们考虑一问题当浏览器关闭时session会不会丢失,从上面叙述分析session的信息是通过sessionid获取的,而sessionid是存放在会话cookie当中的,当浏览器关闭的时候会话cookie消失所以我们的sessionid也就消失了,但是session的信息还存在服务器端,这时我们只是查不到所谓的session但它并不是不存在。那么,session在什么情况下丢失,就是在服务器关闭的时候,或者是sessio过期,再或者调用了invalidate()的或者是我们想要session中的某一条数据消失调用session.removeAttribute()方法,然后session在什么时候被创建呢,确切的说是通过调用session.getsession来创建,这就是session与cookie的区别。

> Session的生命周期

+ Session何时生效

Session在用户第一次访问服务器时创建，注意只有访问JSP、Servlet等程序时才会创建Session，而访问HTML、image等静态文件时不会产生Session,可调用request.getSession(true)强制生成Session。

+ Session何时失效
 - 服务器会把长时间没有活动的Session从服务器内存中清除，此时Session便失效。Tomcat中Session的默认失效时间为20分钟。
 - 调用Session的invalidate方法。
        ```java
            HttpSession session = request.getSession();
            session.invalidate();//注销该request的所有session
        ```
 - session的过期时间是从什么时候开始计算的？是从一登录就开始计算还是说从停止活动开始计算？
    从session不活动的时候开始计算，如果session一直活动，session就总不会过期。从该Session未被访问,开始计时; 一旦Session被访问,计时清0;

### JDK1.8新特性

[参考文章](https://www.jianshu.com/p/5b800057f2d8)

> 语言特性

+ Lambda表达式和函数式接口
+ 接口的默认方法和静态方法
+ 方法引用
+ 重复注解`@Repeatable`
+ 更好的类型推断
+ 拓宽注解的应用场景

> 编译器特性

+ 参数名称
  
> 库

+ Optional
+ Streams
+ Date/Time API
+ Nashorn JavaScript引擎
+ Base64
+ 并行数组
+ 并发性

> 工具

+ Nashorn引擎：jjs
+ 类依赖分析器：jdeps

> 运行时(JVM)

使用Metaspace（JEP 122）代替持久代（PermGen space）。在JVM参数方面，使用-XX:MetaSpaceSize和-XX:MaxMetaspaceSize代替原来的-XX:PermSize和-XX:MaxPermSize。

### NIO原理

> NIO与传统IO比较

+ 面向缓冲和面向流
+ 非阻塞与阻塞
+ 选择器(单线程管理多个通道)
  
> NIO与IO适用场景不同

由于NIO是面向缓冲的这导致数据处理前都需要处理缓冲区中数据是否完整或者读取完毕。

NIO适用于连接数大，但每次只传送极少数的数据，如聊天服务器

IO适合于连接数少，但需要传输大量数据

> NIO原理

NIO核心 buffer channal selector

**小小tips：**

+ 同步与异步：同步是指用户空间是主动发起IO请求的一方，内核空间被动接受，而异步相反，用户空间线程向内核空间注册各种回调函数，由内核空间调用
+ 阻塞与非阻塞

**四种IO模型**

[参考文章](https://www.cnblogs.com/crazymakercircle/p/10225159.html#%E5%9B%9B%E7%A7%8D%E4%B8%BB%E8%A6%81%E7%9A%84io%E6%A8%A1%E5%9E%8B)

+ 同步阻塞(IO)
+ 同步非阻塞(NIO)
+ 多路复用  IO多路复用模型，就是通过一种新的系统调用，一个进程可以监视多个文件描述符，一旦某个描述符就绪（一般是内核缓冲区可读/可写），内核kernel能够通知程序进行相应的IO系统调用。
+ 异步IO模型


## JVM

### JVM GC收集器特点

[参考文章](https://crowhawk.github.io/2017/08/15/jvm_3/)

> 新生代收集器

收集器名称|垃圾回收算法|特点
----|----|----
Serial|复制算法| + HotSpot虚拟机运行在Client模式下的默认的新生代收集器<br> + 会暂停所有用户线程(stop the world)<br> + 简单而高效（与其他收集器的单线程相比），对于限定单个CPU的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得更高的单线程收集效率。
Parnew|复制算法| + 是Serial收集器的多线程版本<br> + 同样会暂停所有用户线程<br> + Server模式下首选的虚拟机新生代收集器<br> + 多CPU下性能会优于Serial收集器<br> + 只有Serial和Parnew能和CMS（Concurrent Mark Sweep收集器配合
Parallel Scavenge|复制算法|+ 并行的多线程新生代GC收集器<br> + CMS等收集器的关注点是尽可能缩短垃圾收集时用户线程的停顿时间，而Parallel Scavenge收集器的目标是达到一个可控制的吞吐量（Throughput）<br> + GC自适应的调节策略（GC Ergonomics）<br> + Parallel Scavenge收集器无法与CMS收集器配合使用，所以在JDK 1.6推出Parallel Old之前，如果新生代选择Parallel Scavenge收集器，老年代只有Serial Old收集器能与之配合使用。

> 老年代收集器

收集器名称|垃圾回收算法|特点
----|----|----
Serial Old|标记-整理算法| + 工作流程与新生代Serial收集器类似<br> + 也会暂停所有用户线程
Parallel Old|标记-整理算法|+ 多线程标记-整理算法<br> + 应用于**注重吞吐量以及CPU资源敏感**场合<br> + 配合新生代Parrallel Scavenge收集器
CMS收集器|标记-清除算法|+ 一种以获取**最短回收停顿时间**为目标的收集器，它非常符合那些集中在互联网站或者B/S系统的服务端上的Java应用，这些应用都非常重视服务的响应速度<br> 

**CMS(Concurrent Mark Swap)收集器**

+ 工作流程：
    - 初始标记：仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，需要“Stop The World”。
    - 并发标记：(不暂停用户线程)进行GC Roots Tracing的过程，在整个过程中耗时最长。
    - 重新标记：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短。此阶段也需要“Stop The World”。
    - 标记清除：并发
+ 优点
  CMS是一款优秀的收集器，它的主要优点在名字上已经体现出来了：**并发收集、低停顿**，因此CMS收集器也被称为并发低停顿收集器（Concurrent Low Pause Collector）。
+ 缺点
 - 对**CPU资源非常敏感** 其实，面向并发设计的程序都对CPU资源比较敏感。在并发阶段，它虽然不会导致用户线程停顿，但会因为占用了一部分线程（或者说CPU资源）而导致应用程序变慢，总吞吐量会降低。CMS默认启动的回收线程数是（CPU数量+3）/4，也就是当CPU在4个以上时，并发回收时垃圾收集线程不少于25%的CPU资源，并且随着CPU数量的增加而下降。但是当CPU不足4个时（比如2个），CMS对用户程序的影响就可能变得很大，如果本来CPU负载就比较大，还要分出一半的运算能力去执行收集器线程，就可能导致用户程序的执行速度忽然降低了50%，其实也让人无法接受。
 - **无法处理浮动垃圾（Floating Garbage）** 可能出现“Concurrent Mode Failure”失败而导致另一次Full GC的产生。由于CMS并发清理阶段用户线程还在运行着，伴随程序运行自然就还会有新的垃圾不断产生。这一部分垃圾出现在标记过程之后，CMS无法再当次收集中处理掉它们，只好留待下一次GC时再清理掉。这一部分垃圾就被称为“浮动垃圾”。也是由于在垃圾收集阶段用户线程还需要运行，那也就还需要预留有足够的内存空间给用户线程使用，因此CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，需要预留一部分空间提供并发收集时的程序运作使用。
 - **标记-清除算法导致的空间碎片** CMS是一款基于“标记-清除”算法实现的收集器，这意味着收集结束时会有大量空间碎片产生。空间碎片过多时，将会给大对象分配带来很大麻烦，往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象。

**G1(Garbage First)收集器**

+ 并行与并发
+ 空间整合
+ 分代收集
+ 可预测停顿

> 总结

收集器|串行、并行or并发|新生代or老年代|算法|目标|适用场景
----|----|----|----|----|----
Serial|串行|新生代|复制算法|响应速度优先|单CPU环境下的Client模式
Serial Old|串行|老年代|标记-整理算法|响应速度优先|单CPU环境下的Client模式、CMS的后备预案
Parnew|并行|新生代|复制算法|响应速度优先|多CPU环境时在Server模式下与CMS配合
Parallel Scavenge|并行|新生代|复制算法|吞吐量优先|在后台运算而不需要太多交互的任务
Parallel old|并行|老年代|标记-整理算法|吞吐量优先|在后台运算而不需要太多交互的任务
CMS|并发|老年代|标记-清除算法|响应速度优先|集中在互联网站或B/S系统服务端上的Java应用
G1|并发|both|标记-整理算法+复制算法|响应速度优先|面向服务端应用，将来替换CMS

### JVM双亲委派机制

[参考文章](https://blog.csdn.net/u011080472/article/details/51332866)

Java虚拟机中的一个类由其类加载器和二进制字节流class文件唯一确定。

> 什么是JVM双亲委派机制？

所谓双亲委派机制是Java推荐的类加载机制，其实现流程就是当需要加载一个类的时候，首先将该加载需求委托给父类加载器，如果父类不能完成类加载，则再调用子类加载器。

类加载器的继承关系：

启动类加载器<-扩展类记载器<-应用/系统类加载器<-用户自定义类加载器

> 为什么需要双亲委派模型？

举个例子来说明，例如`java.lang.Object`类在`rt.jar`中，是由启动类加载器加载，在双亲委派机制下，无论哪个类加载器要加载该类，最终都会委托给启动类加载器来加载，因此得到都是同一个类。相反，如果没有双亲委派机制，开发者在自己的classpath下定义一个`Object`类，并由自己的类加载器或者应用类加载器加载，得到是不同的`Object`类，程序就会存在好几个`Object`类，导致程序混乱。

> 什么时候需要破坏双亲委派机制？

JNDI服务：JNDI的目的就是对资源进行集中管理和查找，它需要调用独立厂商实现部部署在应用程序的classpath下的JNDI接口提供者(SPI, Service Provider Interface)的代码。

因此，为了破坏这种双亲委派机制，在**启动类里也能够使用系统类加载器对类进行加载**，Java设计了**线程上下文加载器**。在核心类进行类加载时，可以读取当前线程的线程上下文加载器，使用该加载器加载实现了SPI接口的类。

Java中所有涉及SPI的加载动作基本上都采用这种方式，例如JNDI,JDBC,JCE,JAXB和JBI等

### JVM 运行时数据区

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

用于存放**已被加载的类信息、常量、静态变量、即时编译器编译后的代码**等数据。

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

### JVM: 类加载机制

Java虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这就是虚拟机的加载机制。

[要看看哦](https://juejin.im/post/5a810b0e5188257a5c606a85#heading-1)

### JVM: 常见GC算法

> Java四种引用类型

```java
Object ref=new Object();//ref是Object对象的强引用

//将一个软引用指向Object对象，此时Object对象有两个引用
SoftReference<Object> sf=new SoftReference<Object>(ref);

ref=null;//移除强引用

Object object=ref.get();//获取目标对象

System.gc();//gc只有在内存不足时才会移除软引用对象
```

引用类型|取得目标方法|垃圾回收条件|是否可能内存泄漏
----|----|----|----
强引用|直接调用|不回收|可能
软引用|get()|视内存情况|不可能
弱引用|get()|gc即回收|不可能
虚引用|无法取得|不回收|可能

[不记得时，就看看吧](https://zhang0peter.com/2020/02/13/java-gc/)

> 标记算法

+ 引用计数法
  
  通过在对象中分配一个字段用来存储对象引用计数，一旦对象被引用，则引用计数加一，引用失效则减一，一旦引用计数为0，可以被回收

  - 优点：实现简单，过程高效，具有实时性

  - 缺点：需要额外资源维护，且无法解决对象循环依赖问题(Python中解决方法是采用标记-清除算法)

+ 可达性分析法
  
  维护一系列`GC Root`对象作为根，当一个对象经过引用链到达根节点时没有任何引用，则表明该对象不可用，可以被GC回收。

  可以当作`GC Root`的对象包括但不限于:

   - 方法中的局部变量
   - 静态变量、常量
   - JNI handles
  
  需要注意的是，在可达性分析算法中被判定不可达的对象还未真的判『死刑』，至少要经历两次标记过程：判断对象是否有必要执行finalize()方法；若被判定为有必要执行finalize()方法，之后还会对对象再进行一次筛选，如果对象能在finalize()中重新与引用链上的任何一个对象建立关联，将被移除出“即将回收”的集合。

> 回收算法

+ 标记清除算法: 会产生空间碎片，当出现连续空间不足够时，虚拟机会感知内存不足，则再次触发GC，而且标记清除算法效率不高，会导致GC时间占用过多，影响程序运行。
+ 复制算法：以空间换时间的算法，每次只用一半空间，且不存在空间碎片。在虚拟机内存的新生代中，大批对象死去只保留少数对象，因此适用于复制算法。
+ 标记整理算法：是在标记清除算法基础上加上对象移动这个整理过程，从而解决了空间碎片问题，但效率比标记清除还低，适用于老年代。
+ 分代收集算法

    从上面三种 GC 算法可以看到，并没有一种空间与时间效率都是比较完美的算法，所以只能做的是综合利用各种算法特点将其作用到不用的内存区域。

    js的V8引擎，Python, HotSpot 都采用了分代收集算法。

    目前JVM虚拟机根据对象存活周期不同划分内存区域，一般分为新生代，老年代。新对象一般情况都会优先分配在新生代，新生代对象若存活时间大于一定阈值之后，将会移到至老年代。新生代的对象都是存活时间短，老年代的对象存活时间长。

    新生代每次 GC 之后都可以回收大批量对象，所以比较适合复制算法，只需要付出少量复制存活对象的成本。这里内存划分并没有按照 1:1 划分，默认将会按照 8:1:1 划分成 Eden 与两块 Survivor 空间。每次使用 Eden 与一块 Survivor 空间，这样我们只是闲置 10% 内存空间。不过我们每次回收并不能保证存活对象小于 10%,在这种情况下就需要依靠老年代的内存分配担保。当 Survivor 空间并不能保存剩余存活对象，就将这些对象通过分配担保进制移动至老年代。

    老年代中对象存活率将会特别高，且没有额外空间进行分配担保，所以并不适合复制算法，所以需要使用标记-清除或标记-整理算法。

    大多数情况下，新的对象都分配在Eden区，当 Eden 区没有空间进行分配时，将进行一次 Minor GC，清理 Eden 区中的无用对象。清理后，Eden 和 From Survivor 中的存活对象如果小于To Survivor 的可用空间则进入To Survivor，否则直接进入老年代；Eden 和 From Survivor 中还存活且能够进入 To Survivor 的对象年龄增加 1 岁（虚拟机为每个对象定义了一个年龄计数器，每执行一次 Minor GC 年龄加 1），当存活对象的年龄到达一定程度（默认 15 岁）后进入老年代，可以通过 -XX:MaxTenuringThreshold 来设置年龄的值。

    当进行了 Minor GC 后，Eden 还不足以为新对象分配空间（那这个新对象肯定很大），新对象直接进入老年代。

    占 To Survivor 空间一半以上且年龄相等的对象，大于等于该年龄的对象直接进入老年代，比如 Survivor 空间是 10M，有几个年龄为 4 的对象占用总空间已经超过 5M，则年龄大于等于 4 的对象都直接进入老年代，不需要等到 MaxTenuringThreshold 指定的岁数。

    在进行 Minor GC 之前，会判断老年代最大连续可用空间是否大于新生代所有对象总空间，如果大于，说明 Minor GC 是安全的，否则会判断是否允许担保失败，如果允许，判断老年代最大连续可用空间是否大于历次晋升到老年代的对象的平均大小，如果大于，则执行 Minor GC，否则执行 Full GC。

    大对象（需要大量连续内存的对象）例如很长的数组，会直接进入老年代，如果老年代没有足够的连续大空间来存放，则会进行 Full GC。

### JVM内存模型

### JVM运行时内存区域

### 常见垃圾收集器(CMS、G1、ZGC)

### 常见的启动参数

### 内存溢出的分析过程

### JVM：`synchronized`和`volatile`关键字在JVM中的行为


## Spring

### Spring Aop实现方式，还有其他的什么方式实现

AOP 面向切面编程，是对面向对象的补充，处理系统中各个模块的横切关注点，常用于事务管理、日志模块、缓存等。

AOP实现的关键在于AOP框架自动创建AOP代理，常有以AspectJ为代表的静态代理，和以SpringAop为代表的动态代理

SpringAop的实现方式主要有通过反射的JDK动态代理和CGLIB(Code Generation Library).

+ JDK动态代理：核心是`InvocationHandler`接口和`Proxy`类，目标类必须是个实现接口的类

    - 基于标准的JDK动态代理

    - 只针对实现了接口的业务对象

+ CGLIB：对于没有实现接口的类，SpringAOP使用CGLIB来动态代理目标类，**注意CGLIB是通过继承来实现动态代理，对于final类不能通过CGLIB做动态代理**

    - 通过动态地对目标子类化来实现AOP代理

    - 需要指定`@EnableAspectJAutoProxy(proxyTargetClass = true)`来强制使用

    - 当目标类没有实现接口会默认使用CGLIB

**JDK动态代理示例**

```java
//一个接口

/**
 * @program:datastructureimpl
 * @description:服务接口
 * @author: Huan Wang(https://github.com/njustwh2014)
 * @create:2020-02-27 21:09
 **/
public interface Service {
    public void add(int num);
    public int search();
}

//接口对应实现类

/**
 * @program:datastructureimpl
 * @description:服务实现
 * @author: Huan Wang(https://github.com/njustwh2014)
 * @create:2020-02-27 21:11
 **/
public class ServiceImpl implements Service {
    private int num;

    public ServiceImpl(int num) {
        this.num = num;
    }

    @Override
    public void add(int num) {
        System.out.println("add...");
        this.num+=num;
    }

    @Override
    public int search() {
        return this.num;
    }
}

//InvocationHandler实现类 这里我类的名字拼错了


import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * @program:datastructureimpl
 * @description:Invocation实现类
 * @author: Huan Wang(https://github.com/njustwh2014)
 * @create:2020-02-27 21:13
 **/
public class MyInnovacation implements InvocationHandler {
    private Object target;
    public MyInnovacation(Object o) {
        super();
        this.target=o;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //方法之前前操作
        System.out.println(method.getName()+"执行前");
        Object result=method.invoke(target,args);//注意不是proxy，不然会循环invoke
        //方法执行后操作
        System.out.println(method.getName()+"执行后");
        return result;
    }

    public static void main(String[] args) {
        Service service=new ServiceImpl(10);
        MyInnovacation myInnovacation=new MyInnovacation(service);
        Service proxyService= (Service) Proxy.newProxyInstance(service.getClass().getClassLoader(),service.getClass().getInterfaces(),myInnovacation);

        proxyService.add(10);

        proxyService.add(20);

        System.out.println("current num in target: "+(int)proxyService.search());
    }
}
```

**CGLib示例**

```java
//目标类 不需要实现统一接口
/**
 * @program:datastructureimpl
 * @description:未实现统一接口类
 * @author: Huan Wang(https://github.com/njustwh2014)
 * @create:2020-02-27 21:37
 **/
public class Base {
    private int num;

    public Base(int num) {
        this.num = num;
    }

    //cglib动态代理的目标类必须要有无参构造器
    public Base() {
        this.num=0;
    }

    public void add(int num){
        this.num+=num;
    }

    public int search(){
        return num;
    }
}

//实现动态代理类CglibProxy，需要实现MethodInterceptor接口的intercept方法。该代理中在add方法前后加入了自定义的切面逻辑，目标类add方法执行语句为proxy.invokeSuper(object, args);
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * @program:datastructureimpl
 * @description:cglib动态代理类
 * @author: Huan Wang(https://github.com/njustwh2014)
 * @create:2020-02-27 21:39
 **/
public class CglibProxy implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("before....");
        Object result=methodProxy.invokeSuper(o,args);
        System.out.println("after...");
        return result;
    }
}

//代理工厂类 用于获取增强后的目标类

import org.springframework.cglib.proxy.Enhancer;

/**
 * @program:datastructureimpl
 * @description:增强类的工厂类
 * @author: Huan Wang(https://github.com/njustwh2014)
 * @create:2020-02-27 21:50
 **/
public class CglibProxyFactory {

    /**
    * @Description: 获得增强后的目标类，也就是切入逻辑advice之后的目标类
    * @Param: [proxy]
    * @return: cn.edu.seu.wh.aop.cglib.proxy.Base
    * @thorws:
    * @Author: Mr.Wang
    * @Date: 2020/2/27
    */
    public static Base getInstance(CglibProxy proxy){
        Enhancer enhancer=new Enhancer();
        enhancer.setSuperclass(Base.class);
        //回调方法的参数为代理类对象CglibProxy，最后增强目标类调用的是代理类对象CglibProxy中的intercept方法
        enhancer.setCallback(proxy);
        //此刻已是增强后的目标类，继承于源目标类
        Base base=(Base)enhancer.create();
        return base;
    }

    public static void main(String[] args) {
        CglibProxy cglibProxy=new CglibProxy();
        //base为增强后的目标类
        Base base=CglibProxyFactory.getInstance(cglibProxy);
        base.add(10);
        base.search();
    }
}
```

### IOC容器初始化过程

IOC初始化过程主要包括**Resource资源定位，BeanDefinition载入，BeanDefinition注册**三个步骤。

+ Resource资源定位

Resource资源定位是指BeanDefinition资源定位，IOC容器找数据的过程。Spring通过外部资源，如xml文件或注解形式定描述一个Bean对象，IOC容器第一步就是需要定位Resource资源，由ResourceLoader通过统一统一的Resource接口实现。

+ BeanDefinition载入

载入的过程就是把定义好Bean表示成IOC容器内部数据结构，即BeanDefinition.在配置文件中每一个Bean都对应着一个BeanDefinition对象。

通过BeanDefinitionReader读取，解析Resource定位的资源，将用户定义好的Bean表示成IOC容器的内部数据结构BeanDefinition。

在IOC容器内部维护着一个BeanDefinitionMap的数据结构，通过BeanDefinitionMap，IOC容器可以对Bean进行更好的管理

+ BeanDefinition注册

注册就是将前面的BeanDefition保存到Map中的过程，通过BeanDefinitionRegistry接口来实现注册。

IOC容器的初始化过程就是对Bean定义资源的定位、载入和注册，此时容器对Bean的依赖注入并没有发生。依赖注入时刻由`lazy-init=false`决定。

### BeanFactory与FactoryBean的区别

+ BeanFactory: Bean工厂，是一个工厂，是Spring IOC容器的最顶层接口，用于管理bean，即实例化、定位和配置应用程序中的对象以及建立这些对象间的依赖关系。
  
+ FactoryBean: 工厂Bean，是一个Bean。作用是产生其他Bean实例，需要提供一个工厂方法，该方法用于返回其他Bean实例。

### 接着说说BeanFactory与ApplicationContext区别

BeanFactory是Spring里面最顶层的接口，包含了各种Bean的定义，读取Bean配置文档，管理Bean的加载、实例化，控制Bean的生命周期，维护Bean之间的依赖关系。

ApplicationContext接口是BeanFactory的派生，除了提供BeanFactory所具有的功能外，还提供了更完整的框架功能：

+ 继承了MessageSource，支持国际化。

+ 提供了统一的资源文件访问方式。

+ 提供在Listener中注册Bean的事件。

+ 提供同时加载多个配置文件的功能。

+ 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层。

**ApplicationContext 三种常见的实现方式:**

+ FileSystemXmlApplicationContext：此容器从一个XML文件中加载Bean的定义，XML Bean 配置文件的全路径名必须提供给它的构造函数。

+ ClassPathXmlApplicationContext：此容器也从一个XML文件中加载Bean的定义，需要正确设置classpath因为这个容器将在classpath里找Bean配置。

+ WebXmlApplicationContext：此容器加载一个XML文件，定义了一个WEB应用的所有Bean。

**在创建Bean和内存方面的区别**

+ **BeanFactory采用的是延迟加载形式来注入Bean的**，即只有在使用到某个Bean时(调用`getBean()`)，才对该Bean进行加载实例化。这样，就不能发现一些存在于Spring配置中的问题。如果Bean的某一个属性没有注入，BeanFactory加载后，直至第一次使用调用getBean方法才会抛出异常。

+ ApplicationContext，它是在容器启动时，一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误，这样有利于检查所依赖属性是否注入。 ApplicationContext启动后预载入所有的单实例Bean，通过预载入单实例Bean ,确保当需要的时候，可以直接获取。
  
+ 相对于基本的BeanFactory，ApplicationContext不足之处是占用内存空间。当应用程序配置Bean较多时，程序启动较慢，因为其一次性创建了所有的Bean。

**二者优缺点对比**

> BeanFactory的优缺点：

+ 优点：应用启动的时候占用资源很少，对资源要求较高的应用，比较有优势；

+ 缺点：运行速度会相对来说慢一些。而且有可能会出现空指针异常的错误，而且通过Bean工厂创建的Bean生命周期会简单一些。


> ApplicationContext的优缺点：

+ 优点：所有的Bean在启动的时候都进行了加载，系统运行的速度快；在系统启动的时候，可以发现系统中的配置问题。

+ 缺点：把费时的操作放到系统启动中完成，所有的对象都可以预加载，缺点就是内存占用较大。

### Spring加载过程

![Springboot启动结构图](Java基础面试题汇总/springboot启动结构图.png)

[参考文章](https://blog.csdn.net/hfmbook/article/details/100507083)