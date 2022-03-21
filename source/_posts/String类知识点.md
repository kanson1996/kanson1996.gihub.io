---
title: String类相关知识点
date: 2020-08-14 22:04:47
tags:
	- Java
	- 基础知识
categories: 技术
---

### Java语言中String类是否可以被继承？

在Java中String类的定义是

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {...}
```

由此，可知String类是不可以被继承的，因为被final修饰的类是不可以被继承的

<!--more-->

### final关键字

可以用来修饰类、成员变量、方法

#### 修饰类

被final修饰的类就不能被其他类继承了，设计类的时候如果不想这个类被其他类继承或者考虑一些安全因素，可以使用final修饰，否则尽量不要把类设计成final的。同时，类中所有的方法也被隐式的变为final方法。

#### 修饰变量

首先，变量分为基本数据类型和引用数据类型。被final修饰的基础类型和引用类型都是不能再次赋值的，这就说明被final修饰的变量是不可变的，相当于常量。因此，final修饰的变量必须被初始化，初始化可以在声明变量的时候，也可以在构造函数中初始化。

##### final修饰的变量和普通变量有什么区别

* 被final修饰的变量在使用的时候可以直接替换，因为其本身在编译期就可以确定下来是固定不可变的，对应值就会被放到字符串常量池中，而普通变量在编译阶段是一个变量，不能确定对应的值。

* 只有在编译阶段确定下来的字符串才会被放到字符串常量池中

#### 修饰方法

* final修饰的方法，不能被子类覆盖
* 仍然可以正常被调用
* 可以实现重载

### static关键字

定义静态常量

```java
static final String NAME = "hello java"; 
```

static表示唯一，独此一份，表示静态；final用来表示不可变

二者结合就是静态常量，全局唯一，同时与final不同，只被static修饰的变量，其值可以被修改。



