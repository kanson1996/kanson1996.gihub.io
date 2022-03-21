---
title: Spring事务传播机制
date: 2019-10-24 21:51:12
tags:
	- Spring
	- 事务
categories: 技术
---

#### Spring事务传播机制

1、事务抽象，特性：

①REQUIRED：如果当前没有事务就**需要**创建一个新事务，否则就加入该事务，最常用也是Spring默认设置

<!--more-->

```java
    @Transactional(propagation = Propagation.REQUIRED)
	public void methodA() {
         // 7种事务传播机制：默认都是REQUIRED 
        methodB();
    }

    @Transactional(propagation = Propagation.REQUIRED)
	public void methodB() {
        // do something
    }
```

使用默认的事务传播机制，执行过程如下：调用A方法开启一个事务执行方法A的代码，紧接着执行方法B的代码，提交或者回滚事务（如果方法B出现了异常，则方法A最终回滚整体单个事务）

②SUPPORTS：**支持**当前事务，如果当前存在事务，就加入该事务，否则以非事务执行

```java
    @Transactional(propagation = Propagation.REQUIRED)
    public void methodA() {
         // 7种事务传播机制：默认都是REQUIRED 
        methodB();
    }

    @Transactional(propagation = Propagation.SUPPORTS)
    public void methodB() {
        // do something
    }
```

直接调用方法B不会执行事务

③MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，否则就**强制**抛出异常

```java
    @Transactional(propagation = Propagation.REQUIRED)
    public void methodA() {
         // 7种事务传播机制：默认都是REQUIRED 
        methodB();
    }

    @Transactional(propagation = Propagation.SUPPORTS)
    public void methodB() {
        // do something
    }
```

直接调用方法B会报错

④REQUIRES_NEW：创建新事务，无论是否存在事务，都创建新事务

```java
    @Transactional(propagation = Propagation.REQUIRED)
    public void methodA() {
         // 7种事务传播机制：默认都是REQUIRED 
        methodB();
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void methodB() {
        // do something
    }
```

调用方法A会开启事务1，执行完A里面的代码后，开启事务2执行方法B里面的代码，提交或回滚事务2，最后提交或回滚事务1。事务1和事务2互不影响

⑤NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，则把当前事务**挂起**（一般很少用到）

⑥NEVER：以非事务方式执行操作，如果当前存在事务，则抛出异常

⑦NESTED：嵌套事务：外层的事务如果回滚，会导致内层的事务也回滚，但内层的事务如果回滚仅仅回滚自己的代码

#### 相关思考：

1.有一段业务逻辑，A调用B，如果A出错了仅仅回滚A，不能回滚B，这时必须得用REQUIRES_NEW传播机制，让A和B的事务是不同的两个事务

2.A调用B，如果出错，B只能回滚自己，而A可以带着B一起回滚，这时得用NESTED来嵌套事务

Tips: Spring事务隔离级别

>  Spring默认的是数据库存储引擎的事务隔离级别，mysql默认隔离级别为可重复读


