---
title: 开源规则引擎Drools
date: 2020-06-04 19:19:46
tags:[Drools]
categories: Drools
---

[demo地址](https://github.com/njustwh2014/drools-demo)

##  为什么需要[规则引擎](https://www.ibm.com/developerworks/cn/opensource/os-drools/index.html)

规则引擎起源于基于规则的专家系统。利用规则引擎可以将商业决策者的商业运营决策逻辑和应用开发者的技术决策分离。我们可以把商业决策放在中心数据库或其他统一地方，从而能够在运行时动态的管理和修改，改善了开发和运营效率。

目前Java开源的规则引擎主要有：Drools、Easy Rules、Mandarax、IBM ILOG。使用最广泛并且开源的是Drools。


### 规则引擎的优点

#### 声明式编程
规则可以很容易地解决困难的问题，并得到解决方案的验证。与代码不同，规则以较不复杂的语言编写; 业务分析师可以轻松阅读和验证一套规则。

#### 逻辑和数据分离
数据位于“域对象”中，业务逻辑位于“规则”中。根据项目的种类，这种分离是非常有利的。

#### 速度和可扩展性
写入Drools的Rete OO算法已经是一个成熟的算法。在Drools的帮助下，您的应用程序变得非常可扩展。如果频繁更改请求，可以添加新规则，而无需修改现有规则。

#### 知识集中化
通过使用规则，您创建一个可执行的知识库（知识库）。这是商业政策的一个真理点。理想情况下，规则是可读的，它们也可以用作文档。


## rete 算法
Rete 算法最初是由卡内基梅隆大学的 Charles L.Forgy 博士在 1974 年发表的论文中所阐述的算法 , 该算法提供了专家系统的一个高效实现。自 Rete 算法提出以后 , 它就被用到一些大型的规则系统中 , 像 ILog、Jess、JBoss Rules 等都是基于 RETE 算法的规则引擎

Rete 在拉丁语中译为”net”，即网络。Rete 匹配算法是一种进行大量模式集合和大量对象集合间比较的高效方法，通过网络筛选的方法找出所有匹配各个模式的对象和规则。

其核心思想是将分离的匹配项根据内容动态构造匹配树，以达到显著降低计算量的效果。Rete 算法可以被分为两个部分：规则编译和规则执行。当Rete算法进行事实的断言时，包含三个阶段：匹配、选择和执行，称做 match-select-act cycle。

## DRools特点

Drools 是一个基于Charles Forgy’s的RETE算法的，易于访问企业策略、易于调整以及易于管理的开源业务规则引擎，符合业内标准，速度快、效率高。 业务分析师人员或审核人员可以利用它轻松查看业务规则，从而检验是否已编码的规则执行了所需的业务规则。

Drools 是用Java语言编写的开放源码规则引擎，使用Rete算法对所编写的规则求值。Drools允许使用声明方式表达业务逻辑。可以使用非XML的本地语言编写规则，从而便于学习和理解。并且，还可以将Java代码直接嵌入到规则文件中，这令Drools的学习更加吸引人。

### Drools优点：

+ 非常活跃的社区支持
+ 易用
+ 快速的执行速度
+ 在 Java 开发人员中流行
+ 与 Java Rule Engine API（JSR 94）兼容

## Drools相关概念：

+ 事实（Fact）：对象之间及对象属性之间的关系
+ 规则（rule）：是由条件和结论构成的推理语句，一般表示为if…Then。一个规则的if部分称为LHS，then部分称为RHS。
+ 模式（module）：就是指IF语句的条件。这里IF条件可能是有几个更小的条件组成的大条件。模式就是指的不能在继续分割下去的最小的原子条件。

Drools通过**事实、规则和模式**相互组合来完成工作，drools在开源规则引擎中使用率最广，但是在国内企业使用偏少，保险、支付行业使用稍多。

## 简单demo

drools有专门的[规则语法drl](http://einverne.github.io/post/2019/03/drools-syntax.html),用于描述规则如何执行。

简单demo主要实现当status=hello时，输出hello world，并将status置为bye，当status=bye时，输出goodbye。

用java语言可以描述如下：

```java
if(message.status==Message.HELLO){
    System.out.println(message.message);
    message.status=Message.GOODBYE;
}else if(message.status==Message.GOODBYE){
    System.out.println(message.message);
}
```

用drl语言描述，drl文件需要放在`src/main/resources/..`目录下

```drl
package rules

import cn.edu.seu.wh.drools.simple.demo.entity.Message;


global java.util.List list

rule "Hello World"
    dialect "mvel"
    salience 3
    when
        m:Message(status==Message.HELLO,msg:message)
    then
        System.out.println(msg);
        list.add(m);
        modify(m){message="Goodbye cruel world!",status=Message.GOODBYE};
end

rule "Good Bye"
    dialect "java"
    when
        Message(status==Message.GOODBYE,msg:message)
    then
        System.out.println(msg);
end
```

+ package与java语言类似，但package不必和物理路径一致。
+ import可以导入java类完整路径或静态方法
+ rule 规则名称，需要保持唯一性，可以设置重复次数，默认重复无数次。
+ salience 用于设置优先级，数字越大优先级越高，不设置时执行顺序随机。
+ when 条件语句  就是当达到什么条件的时候
+ then 根据条件的结果，来执行具体动作
+ end 规则结束

需要一个配置文件告诉代码drl文件在哪里，在drools中这个文件是kmodule.xml，放置在`src/main/resources/META-INF`下

kmodule.xml内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<kmodule xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://www.drools.org/xsd/kmodule">


    <kbase name="HelloWorldKB" packages="rules">
        <ksession name="HelloWorldKS"/>
    </kbase>

</kmodule>
```

+ Kmodule 中可以包含一个到多个 kbase,分别对应 drl 的规则文件。
+ Kbase 需要一个唯一的 name,可以取任意字符串。
+ packages 为drl文件所在resource目录下的路径。**注意区分drl文件中的package与此处的package不一定相同**。多个包用逗号分隔。默认情况下会扫描 resources目录下所有(包含子目录)规则文件。**此目录下所有rule名称不能重复**
+ kbase的default属性,标示当前KieBase是不是默认的,如果是默认的则不用名称 就可以查找到该 KieBase,但每个 module 最多只能有一个默认 KieBase。
+ kbase 下面可以有一个或多个 ksession,ksession 的 name 属性必须设置,且必须唯一。

代码端处理如下：

```java
public class HelloWorldDemo {
    public static void main(String[] args) {
        // KieServices is the factory for all KIE services
        KieServices kieServices = KieServices.Factory.get();
        // KieContainer是重量级组件，建议复用
        KieContainer kieContainer = kieServices.getKieClasspathContainer();
        execute(kieContainer);
    }

    private static void execute(KieContainer kieContainer) {
        // 依赖KieContainer产生一个KieSession，它的定义和配置在 META-INF/kmodule.xml
        KieSession kieSession=kieContainer.newKieSession("HelloWorldKS");

        // 一旦KieSession生成，application可以和它交互
        // 这个demo里声明了一个global变量list 在rules/HelloWorld.drl文件里
        List<Object> list=new ArrayList<Object>();
        kieSession.setGlobal("list",list);

        // application也可以设置监听器
        kieSession.addEventListener(new DebugAgendaEventListener());
        kieSession.addEventListener(new DebugRuleRuntimeEventListener());

        // To setup a file based audit logger, uncomment the next line
        // KieRuntimeLogger logger = ks.getLoggers().newFileLogger( ksession, "./helloworld" );

        // To setup a ThreadedFileLogger, so that the audit view reflects events whilst debugging,
        // uncomment the next line
        // KieRuntimeLogger logger = ks.getLoggers().newThreadedFileLogger( ksession, "./helloworld", 1000 );


        // application插入facts into session
        final Message message=new Message();
        message.setMessage("Hello World!");
        message.setStatus(Message.HELLO);
        kieSession.insert(message);

        // 执行所有rules
        kieSession.fireAllRules();

        for(Object obj:list){
            System.out.println("Object in list:"+obj.toString());
        }


        // Remove comment if using logging
        // logger.close();

        kieSession.dispose();

    }
}
```

其中`Message`如下：

```java
@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
public class Message {
    public static final int HELLO=0;
    public static final int GOODBYE=1;
    private String message;
    private int status;
}

```

## 整合Spring，动态从数据库覆盖原有的规则

## 参考文献