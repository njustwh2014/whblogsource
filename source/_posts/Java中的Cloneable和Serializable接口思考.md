---
title: Java中的Cloneable和Serializable接口思考
date: 2019-12-06 09:57:06
tags:
    - Java
    - Cloneable
    - Serializable
categories: Java
---

## Cloneable接口

**clone()方法**：允许在**堆**中克隆出一块与原对象一样的对象，并将这个对象的地址赋予新的引用。

Java中一个类要实现clone()方法，必须实现Cloneable接口，否则在调用clone()方法时会报CloneNotSupportException异常。

> 需要说明：

1. 对象拷贝返回的是一个新对象，而不是一个引用。
2. 拷贝对象与用 new操作符返回的新对象的区别就是这个拷贝已经包含了一些原来对象的信息，而不是对象的初始信息。

示例：
```java
class CloneClass implements CLoneable{
    public int a;
    public Object clone(){
        CloneClass o=null;
        try{
            o=(CloneClass)super.clone();//native方法，不是new出来然后赋值，这样效率低下。
        }catch(CloneNotSupportException e){
            e.printStackTrace();
        }
        return o;
    }
}
```

> 值得注意的地方

1. 为了实现clone功能，CloneClass类实现了Cloneable接口，这个接口属于java.lang 包，java.lang包已经被缺省的导入类中，所以不需要写成java.lang.Cloneable；
2. 重写了clone()方法；

3. 在clone()方法中调用了super.clone()，这也意味着无论clone类的继承结构是什么样的，super.clone()直接或 间接调用了java.lang.Object类的clone()方法。

4. Object类的clone()方法是一个native方法，native方法的效率一般来说都是远高于java中的非native方法。这也解释了为什么要用Object中clone()方法而不是先new一个对象，然后把原始对象中的信息赋到新对象中，虽然这也实现了clone功能，但效率较低。

5. Object类中的clone()方法还是一个protected属性的方法。这也意味着如果要应用clone()方法，必须继承Object类，在Java中所有的类是缺省继承Object类的，也就不用关心这点了。

6. 重写clone()方法。还有一点要考虑的是为了让其它类能调用这个clone类的clone()方法，重写之后要把clone()方法的属性设置为public。

> Java中clone()方法克隆出的对象是浅克隆(拷贝)

+ 浅拷贝（浅克隆）复制出来的对象的所有变量都含有与原来的对象相同的值，而所有的对其他对象的引用仍然指向原来的对象。
  
+ 深拷贝（深克隆）复制出来的所有变量都含有与原来的对象相同的值，那些引用其他对象的变量将指向复制出来的新对象，而不再是原有的那些被引用的对象。换言之，深复制把要复制的对象所引用的对象都复制了一遍。

> 那么利用clone()方法是否可以实现深克隆吗？

我们先来看看浅克隆示例:
```java
public class Student implements Cloneable {

    private String name;
    private int age;
    private Teacher teacher;

    public Student(String name, int age, Teacher teacher) {
        this.name = name;
        this.age = age;
        this.teacher = teacher;
    }

    public Student clone() {
        Student student = null;
        try {
            student = (Student) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return student;
    }
}
```

深克隆示例:
```java
public class Student implements Cloneable {

    private String name;
    private int age;
    private Teacher teacher;

    public Student(String name, int age, Teacher teacher) {
        this.name = name;
        this.age = age;
        this.teacher = teacher;
    }

    public Student clone() {
        Student student = null;
        try {
            student = (Student) super.clone();
            Teacher teacherClone=this.teacher.clone();//这就需要Teacher类也实现了Cloneable接口。
            student.setTeacher(teacherClone);
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return student;
    }
}
```

从以上示例可以看出，利用clone()方法可以实现深克隆，需要逐层实现Cloneable接口，这种方法如果继承链比较长，过程会比较麻烦。

此时我们就需要简便地实现深克隆：利用Serializable接口。

## Serializable接口

> 序列化
对象的寿命通常随着生成该对象的程序的终止而终止，而有时候需要把在内存中的各种**对象的状态**（也就是实例变量，不是方法，区别于**类的状态**）保存下来，并且可以在需要时再将对象恢复。Java提供了一种保存对象状态的机制，那就是序列化。

Java序列化技术可以将一个对象的状态写入一个Byte 流里（序列化），并且可以从其它地方把该Byte流里的数据读出来（反序列化）。

> 什么时候需要序列化？

+ 想把内存中的对象状态保存到一个文件中或者数据库中时候

+ 想把对象通过网络进行传播的时候

> 如何序列化

只要一个类实现Serializable接口，那么这个类就可以序列化。

示例:
```java
class Person implements Serializable{
    private static final long serialVersionUID=1L;
    private String name;
    private int age;
    public Person(String name,int age){
        this.name=name;
        this.age=age;
    }
    public String toString(){
        return "name:"+name+"\t age:"+age;
    }
}
```
通过ObjectOutputStream 的writeObject()方法把这个类的对象写到一个地方（文件），再通过ObjectInputStream 的readObject()方法把这个对象读出来。

```java
//序列化对象
File file=new File("file"+File.separator+"out.txt");

FileOutputStream fos=null;

try{
    fos=new FileOutputStream(file);
    ObjectOutputStream oos=null;
    try{
        oos=new ObjectOutputStream(fos);
        Person person=new Person("huanhuan",23);
        System.out.println(person.toString());

        //写入对象
        oos.writeObject(person);
        oos.flush();
    }catch(IOException e){
        e.printStackTrace();
    }finally{
        try{
            oos.close();
        }catch(IOException e){
            System.out.println("oos关闭失败:"+e.getMessage());
        }
    }
}catch(FileNotFoundException e){
    System.out.println("找不到文件:"+e.getMessage());
}finally{
    try {
        fos.close();
    } catch (IOException e) {
        System.out.println("fos关闭失败："+e.getMessage());
    }
}

// 反序列化对象

FileInputStream fis=null;
try{
    fis=new FileInputStream(file);
    ObjectInputStream ois=null;
    try{
        ois=new ObjetcInputStream(fis);
        try{
            Person person=(Person)ois.readObject();
            System.out.println(person);
        }catch (ClassNotFoundException e) {
                e.printStackTrace();
        } 
    } catch (IOException e) {
            e.printStackTrace();
    }finally{
        try {
            ois.close();
        } catch (IOException e) {
            System.out.println("ois关闭失败："+e.getMessage());
        }
    }
}catch (FileNotFoundException e) {
    System.out.println("找不到文件："+e.getMessage());
} finally{
    try {
        fis.close();
    } catch (IOException e) {
        System.out.println("fis关闭失败："+e.getMessage());
    }
}
```

> serialVersionUID

注意到上面程序中有一个serialVersionUID，实现了Serializable接口之后，Eclipse/IDEA就会提示你增加一个serialVersionUID，虽然不加的话上述程序依然能够正常运行。

序列化ID在Eclipse/IDEA下提供了两种生成策略：

+ 一个是固定的 1L

+ 一个是随机生成一个不重复的long类型数据（实际上是使用JDK工具，根据类名、接口名、成员方法及属性等来生成）
  
上面程序中，输出对象和读入对象使用的是同一个Person类。

如果是通过网络传输的话，如果Person类的serialVersionUID不一致，那么反序列化就不能正常进行。例如在客户端A中Person类的serialVersionUID=1L，而在客户端B中Person类的serialVersionUID=2L 那么就不能重构这个Person对象。

试图重构就会报java.io.InvalidClassException异常，因为这两个类的版本不一致，local class incompatible，重构就会出现错误。

如果没有特殊需求的话，使用用默认的1L就可以，这样可以确保代码一致时反序列化成功。那么随机生成的序列化ID有什么作用呢，有些时候，通过改变序列化ID可以用来限制某些用户的使用。

> 静态变量序列化

序列化只能保存对象的非静态成员交量，不能保存任何的成员方法和静态的成员变量，而且序列化保存的只是变量的值，对于变量的任何修饰符都不能保存。

如果把Person类中的name定义为static类型的话，试图重构，就不能得到原来的值，只能得到null。说明对静态成员变量值是不保存的。这其实比较容易理解，序列化保存的是对象的状态，**静态变量属于类的状态**，因此 序列化并不保存静态变量。

> transient关键字

经常在实现了 Serializable接口的类中能看见transient关键字。 transient关键字的作用是：阻止实例中那些用此关键字声明的变量持久化；当对象被反序列化时（从源文件读取字节序列进行重构），这样的实例变量值不会被持久化和恢复。

当某些变量不想被序列化，同是又不适合使用static关键字声明，那么此时就需要用transient关键字来声明该变量。

```java
class Person implements Serializable{   

    private static final long serialVersionUID = 1L;

    transient String name;
    int age;

    public Person(String name,int age){
        this.name = name;
        this.age = age;
    }   
    public String toString(){
        return "name:"+name+"\tage:"+age;
    }
}
```

在被反序列化后，transient 变量的值被设为初始值，如 int 型的是 0，对象型的是 null。

**注**：对于某些类型的属性，其状态是瞬时的，这样的属性是无法保存其状态的。例如一个线程属性或需要访问IO、本地资源、网络资源等的属性，对于这些字段，我们必须用transient关键字标明，否则编译器将报措。

> 序列化中的继承问题

当一个父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口；

一个子类实现了Serializable接口，它的父类都没有实现Serializable接口，要想将父类对象也序列化，就需要让父类也实现Serializable接口。

第二种情况中：如果父类不实现Serializable接口的话，就需要有默认的无参的构造函数。这是因为创建java对象的时候需要先有父对象，才有子对象，反序列化也不例外。在反序列化时，为了构造父对象，只能调用父类的无参构造函数作为默认的父对象。因此当我们取父对象的变量值时，它的值是调用父类无参构造函数后的值。在这种情况下，在序列化时根据需要在父类无参构造函数中对变量进行初始化，否则的话，父类变量值都是默认声明的值，如int型的默认是0，string型的默认是null。

```java
class People{
    int num;
    public People(){}           //默认的无参构造函数，没有进行初始化
    public People(int num){     //有参构造函数
        this.num = num;
    }
    public String toString(){
        return "num:"+num;
    }
}
class Person extends People implements Serializable{    

    private static final long serialVersionUID = 1L;

    String name;
    int age;

    public Person(int num,String name,int age){
        super(num);             //调用父类中的构造函数
        this.name = name;
        this.age = age;
    }
    public String toString(){
        return super.toString()+"\tname:"+name+"\tage:"+age;
    }
}
```

> Serializable克隆

Java可以把对象序列化写进一个流里面，反之也可以把对象从序列化流里面读取出来，但这一进一出，这个对象就不再是原来的对象了，就达到了克隆的要求。

```java
public class Student implements  Serializable {

    private String name;
    private int age;
    private Teacher teacher;

    public Student(String name, int age, Teacher teacher) {
        this.name = name;
        this.age = age;
        this.teacher = teacher;
    }

    public Student serializableClone() throws IOException, ClassNotFoundException {
        Student clone;

        ByteArrayOutputStream bo = new ByteArrayOutputStream();
        ObjectOutputStream oo = new ObjectOutputStream(bo);
        oo.writeObject(this);
        ByteArrayInputStream bi = new ByteArrayInputStream(bo.toByteArray());
        ObjectInputStream oi = new ObjectInputStream(bi);
        clone = (Student) oi.readObject();

        return clone;
    }
}
```

通过把对象写进ByteArrayOutputStream里，再把它读取出来。注意这个过程中所有涉及的对象都必须实现Serializable接口，由于涉及IO操作，这种方式的效率会比前面的低。

## 参考

> [Java中 Cloneable 、Serializable 接口详解](https://blog.csdn.net/xiaomingdetianxia/article/details/74453033)

> [JAVA 基础 - clone浅克隆与深克隆](https://juejin.im/post/58ae7be02f301e0068ef31f0)

> [Java中如果clone为什么必须实现Cloneable接口](https://cnbin.github.io/blog/2016/02/23/java-zhong-ru-guo-clonewei-shi-yao-bi-xu-shi-xian-cloneablejie-kou/)