---
title: Mybatis自动生成的Example类用法
date: 2020-05-11 15:42:00
tags: [Mysql,Mybatis]
categories: Mysql
---

##  什么是Example类

mybatis-generator会为每个字段产生Criterion，为底层的mapper.xml创建动态sql。理论上通过Example类可以构造出任何筛选条件。

## 了解Example类的成员变量

```java
 //作用：升序还是降序
 //参数格式：字段+空格+asc(desc)
 protected String orderByClause;  
 //作用：去除重复
 //true是选择不重复记录，false，反之
 protected boolean distinct;
 //自定义查询条件
 //Criteria的集合，集合中对象是由or连接
 protected List<Criteria> oredCriteria;
 //内部类Criteria包含一个Cretiron的集合，
 //每一个Criteria对象内包含的Cretiron之间是由  AND连接的
 public static class Criteria extends GeneratedCriteria {
  protected Criteria() {super();}
 }
 //是mybatis中逆向工程中的代码模型
 protected abstract static class GeneratedCriteria {......}
 //是最基本,最底层的Where条件，用于字段级的筛选
 public static class Criterion {......}
```

## Example使用前准备

比如我的Example是根据user表生成的，UserMapper属于dao层，UserMapper.xml是对应的映射文件。

> UserMapper接口

```java
long countByExample(UserExample example);
List<User> selectByExample(UserExample example);
```

## 查询记录数

```java
long count = userMapper.countByExample(example);
```

类似于

```sql
select count(*) from user;
```

## where条件查询或多条件查询

```java
example.setOrderByClause("age asc"); //升序
 example.setDistinct(false); //不去重
 if(!StringUtils.isNotBlank(user.getName())){
 Criteria.andNameEqualTo(user.getName());
 }
 if(!StringUtils.isNotBlank(user.getSex())){
 Criteria.andSexEqualTo(user.getSex());
 }
 List<User> userList=userMapper.selectByExample(example);
```

类似于

```sql
select * from user where name={#user.name} and sex={#user.sex} order by age asc;
```

```java
 UserExample.Criteria criteria1 = example.createCriteria();
 UserExample.Criteria criteria2 = example.createCriteria();
 if(!StringUtils.isNotBlank(user.getName())){
 Criteria1.andNameEqualTo(user.getName());
 }
 if(!StringUtils.isNotBlank(user.getSex())){
 Criteria2.andSexEqualTo(user.getSex());
 }
 Example.or(criteria2);
 List<User> userList=userMapper.selectByExample(example);
```

类似于

```sql
select * from user where name={#user.name} or sex={#user.sex} ;
```

## 模糊查询

```java
 if(!StringUtils.isNotBlank(user.getName())){
 criteria.andNameLIke(‘%’+name+’%’);
 }
 List<User> userList=userMapper.selectByExample(example);
```

类似于

```sql
select * from user where name like %{#user.name}%;
```

## 分页查询

```java
 int start = (currentPage - 1) * rows;
 //分页查询中的一页数量
 example.setPageSize(rows); 
 //开始查询的位置
 example.setStartRow(start);  
 List<User> userList=userMapper.selectByExample(example);
```

类似于

```sql
select * from user where name like %{#user.name}%
```

