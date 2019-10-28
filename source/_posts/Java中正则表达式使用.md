---
title: Java中正则表达式使用
date: 2019-10-28 11:15:43
tags: 
    - Java
    - Regex
categories: Java
img: http://pzpoejx7j.bkt.clouddn.com/regex.png
---
## 什么是正则表达式
正则表达式是一个定义**搜索模式**的字符串。

## 示例

|正则表达式|匹配|
|:-------|:---------------------------|
|this is text|精确匹配到字符串this is text|
|this\s+is\s+text|匹配单词this和is后跟一个或多个空格|
|^\d+(\.\d+)?|^ 定义模式必须匹配字符串的开始，d+ 匹配一个或多个数字，? 表明小括号内的语句是可选的，\. 匹配 "."，小括号表示分组。例如匹配："5"、"1.5" 和 "2.21"|

## 正则表达式编写规则

### 常见匹配符号
|正则表达式|描述|
|:---------|:-------|
|.|匹配所有单个字符，除了换行(Linux换行符\n, Windows换行符\r\n)|
|^regex|正则必须匹配字符串开头|
|regex$|正则必须匹配字符串结尾|
|[abc]|复选集定义，匹配a或b或c|
|[abc][vz]|复选集定义，匹配字母 a 或 b 或 c，后面跟着 v 或 z|
|[^abc]|当插入符 ^ 在中括号中以第一个字符开始显示，则表示否定模式。此模式匹配所有字符，除了 a 或 b 或 c|
|[a-d1-7]|范围匹配，匹配字母 a 到 d 和数字从 1 到 7 之间，但不匹配 d1|
|XZ|X后直接跟着Z|
|X\|Z|匹配X或Z|

### 元字符
元字符是一个预定义的字符。

|正则表达式|描述|
|:---------|:-------|
|\d|匹配一个数字，是 [0-9] 的简写|
|\D|匹配一个非数字，是 [^0-9] 的简写|
|\s	|匹配一个空格，是 [ \t\n\x0b\r\f] 的简写|
|\S	|匹配一个非空格|
|\w	|匹配一个单词字符（大小写字母、数字、下划线），是 [a-zA-Z_0-9] 的简写|
|\W	|匹配一个非单词字符（除了大小写字母、数字、下划线之外的字符），等同于 [^\w]|


### 限制符
限定符定义了一个元素可以发生的频率。

|正则表达式	|描述	|举例|
|:---------|:-------|:--------|
|*	|匹配 >=0 个，是 {0,} 的简写|	X* 表示匹配零个或多个字母 X，.* 表示匹配任何字符串|
|+	|匹配 >=1 个，是 {1,} 的简写|	X+ 表示匹配一个或多个字母 X|
|?	|匹配 1 个或 0 个，是 {0,1} 的简写	|X? 表示匹配 0 个或 1 个字母 X|
|{X}	|只匹配 X 个字符|	\d{3} 表示匹配 3 个数字，.{10} 表示匹配任何长度是 10 的字符串|
|{X,Y}	|匹配 >=X 且 <=Y 个|	\d{1,4} 表示匹配至少 1 个最多 4 个数字|
|*?|	如果 ? 是限定符 * 或 + 或 ? 或 {} 后面的第一个字符，那么表示非贪婪模式（尽可能少的匹配字符），而不是默认的贪婪模式||	

### 分组与反向应用
小括号()可以达到对正则表达式分组的效果。

模式分组后会在正则表达式中创建反向引用。反向引用会保存匹配模式分组的字符串片断，这使得我们可以获取并使用这个字符串片断。

Java中，在以正则表达式替换字符串的语法中，是通过 $ 来引用分组的反向引用，$0 是匹配完整模式的字符串（注意在 JavaScript 中是用 $& 表示）；$1 是第一个分组的反向引用；$2 是第二个分组的反向引用，以此类推。

```java
public static void main(String[] args) {
    String str="Hello ,world .";
    String pattern="(\\w)(\\s+)([.,])";
    System.out.println(str.replaceAll(pattern,"$1$3"));//Hello,World.
}
```
上面的例子中，我们使用了 [.] 来匹配普通字符 . 而不需要使用 [\\.]。因为正则对于 [] 中的 .，会自动处理为 [\.]，即普通字符 . 进行匹配。

### 仅分组但无反向引用
当我们在小括号 () 内的模式开头加入 ?:，那么表示这个模式仅分组，但不创建反向引用。

```java
String str = "img.jpg";
// 分组且创建反向引用
//        Pattern pattern = Pattern.compile("(jpg|png)");
//        Matcher matcher = pattern.matcher(str);
//        while (matcher.find()) {
//            System.out.println(matcher.group());//jpg
//            System.out.println(matcher.group(1));//jpg
//        }

// 分组但不创建反向引用
Pattern pattern = Pattern.compile("(?:jpg|png)");
Matcher matcher = pattern.matcher(str);
while (matcher.find()) {
    System.out.println(matcher.group());//jpg
    System.out.println(matcher.group(1));//Exception in thread "main" java.lang.IndexOutOfBoundsException: No group 1
}
```

### 分组的反向引用副本
Java 中可以在小括号中使用 ?<name> 将小括号中匹配的内容保存为一个名字为 name 的副本。

```java
String str="@wanghuan is a cool boy.";
Pattern pattern=Pattern.compile("@(?<first>\\w+\\s)");
Matcher matcher=pattern.matcher(str);
while(matcher.find()){
    System.out.println(matcher.group());//@wanghuan
    System.out.println(matcher.group(1));//wanghuan
    System.out.println(matcher.group("first"));//wanghuan
}
```

### 否对先行断言
我们可以创建否定先行断言模式的匹配，即某个字符串后面不包含另一个字符串的匹配模式。

否定先行断言模式通过 (?!pattern) 定义。比如，我们匹配后面不是跟着 "b" 的 "a"：
```java
Pattern pattern=Pattern.compile("a(?!a)");
```

### 指定正则表达式的模式
可以在正则开头指定模式修饰符。

> (?i) 使正则忽略大小写。


> (?s) 表示单行模式（"single line mode"）使正则的 . 匹配所有字符，包括换行符。


> (?m) 表示多行模式（"multi-line mode"），使正则的 ^ 和 $ 匹配字符串中每行的开始和结束。

### java中反斜杠
反斜杠 \ 在 Java 中表示转义字符，这意味着 \ 在 Java 拥有预定义的含义。

这里例举两个特别重要的用法：

+ 在匹配 . 或 { 或 [ 或 ( 或 ? 或 $ 或 ^ 或 * 这些特殊字符时，需要在前面加上 \\，比如匹配 . 时，Java 中要写为 \\.，但对于正则表达式来说就是 \.。


+ 在匹配 \ 时，Java 中要写为 \\\\，但对于正则表达式来说就是 \\。


**注意：** Java 中的正则表达式字符串有两层含义，首先 Java 字符串转义出符合正则表达式语法的字符串，然后再由转义后的正则表达式进行模式匹配。

### 易错示例

> [jpg|png] 表示匹配字符j或p或g或p或n或g中的任意一个字符

> (jpg|png) 表示匹配jpg或png

## 在字符串中使用正则表达式

### 内置的字符串正则处理方法
在 Java 中有四个内置的运行正则表达式的方法，分别是 matches()、split())、replaceFirst()、replaceAll()。注意 replace() 方法不支持正则表达式。

|方法|描述|
|:--|:--|
|s.matches("regex")	|当仅且当正则匹配整个字符串时返回 true|
|s.split("regex")	|按匹配的正则表达式切片字符串|
|s.replaceFirst("regex", "replacement")	|替换首次匹配的字符串片段|
|s.replaceAll("regex", "replacement")	|替换所有匹配的字符|

### 示例
```java
System.out.println("wxj".matches("wxj"));
System.out.println("----------");

String[] array = "w x j".split("\\s");
for (String item : array) {
    System.out.println(item);
}
System.out.println("----------");

System.out.println("w x j".replaceFirst("\\s", "-"));
System.out.println("----------");

System.out.println("w x j".replaceAll("\\s", "-"));
```

result:
```bash
true
----------
w
x
j
----------
w-x j
----------
w-x-j
```
## 参考文章
> [Java 正则表达式详解](https://segmentfault.com/a/1190000009162306)

