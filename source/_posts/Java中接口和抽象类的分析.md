---
title: Java中接口和抽象类的分析
date: 2020-10-01 20:44:52 
tags:
	- Java
categories: 技术
---

### 接口

接口是抽象方法的集合，如果一个类实现了某个接口，那么它就继承了这个接口的抽象方法

<!--more-->

```java
public interface MyInterface {

    // 接口当中只能声明一些方法（隐式的抽象方法），但是不能有方法体
    // 接口需要对应的实现类来实现具体方法
    public void printName(String name);

    public String getNmae();
}

public class MyInterfaceImpl implements MyInterface {

    // 抽象类实现接口的时候，需要实现接口当中声明的所有方法
    @Override
    public void printName(String name) {
        System.out.println(name);
    }

    @Override
    public String getNmae() {
        return "name";
    }
}
```

### 抽象类

抽象，顾名思义，就是具体的另一面。抽象类，主要是用来提炼子类的通用特性，是被用来创建具有继承关系的子类的模板。它本身不能被实例化，只能被用作子类的超类。需要实例化时必须重写抽象方法，变成一个具体类。

```java
public abstract class MyAbstractClass {

    // 抽象类当中可以实现默认方法
    public void printNum(int a) {
        System.out.println(a);
    }

    // 抽象方法只需要在抽象类当中声明，交由子类进行实现
    abstract void service(String req, String res);

}
// 继承抽象类的子类
public class MySubAbstractClass extends MyAbstractClass {
    
    public void doSomething() {
        System.out.println("aaa");
    }

    @Override
    void service(String req, String res) {
        // implementation
    }
}

public static void main(String[] args) {
    MyAbstractClass m = new MyAbstractClass() {
        @Override
        void service(String req, String res) {
            // implementation
        }
    };
}
```

### 抽象类和接口的对比

![](https://images2015.cnblogs.com/blog/1064302/201612/1064302-20161230090438195-1243745647.png)



## 相关问题

### 1、接口和抽象类有什么区别

在Java语言中，抽象类abstract class和接口interface`是抽象定义的两种机制`。

正是由于这两种机制的存在，才赋予了Java强大的面向对象能力。抽象类abstract class和接口interface在对于抽象定义方面具有很大的相似性，甚至可以相互替换。因此很多开发者在进行抽象定义时对二者的选择显得比较随意。其实，`两者之间还是有很大的区别`，对于它们的选择能反映出对问题本质的理解、对设计意图的理解。

具体如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190311215243146.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG5saWppbmdyYW4=,size_16,color_FFFFFF,t_70)

### 2、interface应用在什么场合

- 类与类之间`需要特定的接口进行协调`，`而不在乎其如何实现`。
- 作为能够实现特定功能的`标识`存在，也可以是什么接口方法都没有的纯粹标识。如序列化接口：`Serializable`
- 需要将一组类视为单一的类，而`调用者只通过接口来与这组类发生联系`。
- 需要实现特定的`多项功能`，`而这些功能之间可能完全没有任何联系`。
- 想实现多重继承，由于**Java不支持多继承**，子类不能够继承多个类，但可以实现多个接口

### 3、abstract class应用在什么场合

- 子类与子类之间有`共同的方法`（`甚至可以是空方法体`，然后由子类选择自己所感兴趣的方法来`覆盖 重写`），该方法写在抽象类中，避免每个子类再去写一遍；子类与子类之间`不同的方法`作为抽象方法，在抽象类中定义。
- 某些场合下，只靠纯粹的接口不能满足类与类之间的协调，还必需类中`表示状态的属性`来`区别不同的关系`。
- 一些方法是`共同的，与状态无关的，可以共享的`，`无需子类分别实现`，想提供默认实现；而另一些方法却需要各个子类根据自己特定的状态来实现特定的功能。