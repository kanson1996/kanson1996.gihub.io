---
title: Spring容器的启动过程及生命周期
date: 2019-10-08 20:44:52 
tags:
	- Spring
categories: 技术
---



### Spring容器的启动过程及生命周期

### Spring Container启动过程

> 创建和配置实例
>
> 刷新实例

# Spring容器的启动全流程

Spring容器的启动流程如下，这是我在看源码过程中自己总结的流程图，如有错误，还望评论区指点：

![img](https://img2020.cnblogs.com/blog/1771072/202009/1771072-20200909164014978-2077560080.png)

接下来附上源码：

> 为什么是refresh方法命名，而不是init命名呢？
>
> 其实，在ApplicaitonContext建立起来之后，可以通过refresh进行重建，将原来的ac销毁，重新执行一次初始化操作，用refresh更加贴切。



Bean的创建和销毁

- [doGetBean全流程](https://www.cnblogs.com/summerday152/p/13639896.html#dogetbean全流程)
- [createBean](https://www.cnblogs.com/summerday152/p/13639896.html#createbean)
- doCreateBean
  - [createBeanInstance 创建实例](https://www.cnblogs.com/summerday152/p/13639896.html#createbeaninstance-创建实例)
  - [populateBean 填充属性](https://www.cnblogs.com/summerday152/p/13639896.html#populatebean-填充属性)
  - [initializeBean 回调方法](https://www.cnblogs.com/summerday152/p/13639896.html#initializebean-回调方法)



<!--more-->

### 循环依赖

N个类循环(嵌套)引，即N个Bean互相引用对方，最终形成`闭环`，表示对象之间的相互依赖关系

如果在日常开发中我们用new对象的方式，若构造函数之间发生这种**循环依赖**的话，程序会在运行时一直循环调用**最终导致内存溢出**，StackOverflowError

```java
public class A {
	public A() {
        B b = new B();
    }
}

public class B {
    public B() {
        A a = new A();
    }
}
```

无法解决构造器/构造方法的循环依赖，因为一开始就实例化了，初始化

参考文章 https://cloud.tencent.com/developer/article/1497692



对于Spring解决循环依赖的认识

// 循环依赖：调用某实例对象的方法时，方法内部又涉及到另一个类的实例化，而B当中又依赖了A，所以这时先返回一个半成品的B，等A完成了实例化之后，再返回B的实例化，最终初始化

##### 1、构造器注入循环依赖

```java
@Service
public class A {
    public A(B b) {
    }
}
@Service
public class B {
    public B(A a) {
    }
}
```

结果：项目启动失败抛出异常`BeanCurrentlyInCreationException`

构造器注入构成的循环依赖，此种循环依赖方式**是无法解决的**，只能抛出`BeanCurrentlyInCreationException`异常表示循环依赖。这也是构造器注入的最大劣势

`根本原因`：Spring解决循环依赖依靠的是Bean的“中间态”这个概念，而这个中间态指的是`已经实例化`，但还没初始化的状态。而构造器是完成实例化的东东（不存在中间态），所以构造器的循环依赖无法解决

##### 2、field属性注入（setter方法注入）循环依赖

这种方式是我们**最最最最**为常用的依赖注入方式（所以猜都能猜到它肯定不会有问题啦）：

```java
@Service
public class A {
    @Autowired
    private B b;
}

@Service
public class B {
    @Autowired
    private A a;
}
```

**结果：项目启动成功，能够正常work**

##### 3、`prototype` field属性注入循环依赖

`prototype`在平时使用情况较少，但是也并不是不会使用到，因此此种方式也需要引起重视。

```javascript
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Service
public class A {
    @Autowired
    private B b;
}

@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Service
public class B {
    @Autowired
    private A a;
}
```

结果：**需要注意的是**本例中**启动时是不会报错的**（因为非单例Bean`默认`不会初始化，而是使用时才会初始化），所以很简单咱们只需要手动`getBean()`或者在一个单例Bean内`@Autowired`一下它即可

```javascript
// 在单例Bean内注入
    @Autowired
    private A a;
```

这样子启动就报错：

```javascript
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'mytest.TestSpringBean': Unsatisfied dependency expressed through field 'a'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'a': Unsatisfied dependency expressed through field 'b'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'b': Unsatisfied dependency expressed through field 'a'; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'a': Requested bean is currently in creation: Is there an unresolvable circular reference?

	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:596)
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:90)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:374)
```

如何解决？？？ 可能有的小伙伴看到网上有说使用`@Lazy`注解解决：

```javascript
    @Lazy
    @Autowired
    private A a;
```

此处负责任的告诉你这样是解决不了问题的(**可能会掩盖问题**)，`@Lazy`只是延迟初始化而已，当你真正使用到它（初始化）的时候，依旧会报如上异常。

对于Spring循环依赖的情况总结如下：

1. 不能解决的情况： 1. 构造器注入循环依赖 2. `prototype` field属性注入循环依赖
2. 能解决的情况： 1. field属性注入（setter方法注入）循环依赖

## Spring解决循环依赖的原理分析

**`Spring的循环依赖的理论依据基于Java的引用传递`**，当获得对象的引用时，**对象的属性是可以延后设置的**。（但是构造器必须是在获取引用之前，毕竟你的引用是靠构造器给你生成的，儿子能先于爹出生？哈哈）

#### Spring创建Bean的流程

首先需要了解是Spring它创建Bean的流程，我把它的大致调用栈绘图如下： 

![](https://ask.qcloudimg.com/http-save/yehe-6158873/oepgq3cnb0.png?imageView2/2/w/1620)

 对Bean的创建最为核心三个方法解释如下：

- `createBeanInstance`：实例化，其实也就是调用对象的**构造方法**实例化对象
- `populateBean`：填充属性，这一步主要是对bean的依赖属性进行注入(`@Autowired`)
- `initializeBean`：回到一些形如`initMethod`、`InitializingBean`等方法

从对`单例Bean`的初始化可以看出，循环依赖主要发生在**第二步（populateBean）**，也就是field属性注入的处理。

#### Spring容器的`'三级缓存'`

在Spring容器的整个声明周期中，单例Bean有且仅有一个对象。这很容易让人想到可以用缓存来加速访问。 从源码中也可以看出Spring大量运用了Cache的手段，在循环依赖问题的解决过程中甚至不惜使用了“三级缓存”，这也便是它设计的精妙之处~

`三级缓存`其实它更像是Spring容器工厂的内的`术语`，采用三级缓存模式来解决循环依赖问题，这三级缓存分别指：

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
	...
	// 从上至下 分表代表这“三级缓存”
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256); //一级缓存
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16); // 二级缓存
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16); // 三级缓存
	...
	
	/** Names of beans that are currently in creation. */
	// 这个缓存也十分重要：它表示bean创建过程中都会在里面呆着~
	// 它在Bean开始创建时放值，创建完成时会将其移出~
	private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap<>(16));

	/** Names of beans that have already been created at least once. */
	// 当这个Bean被创建完成后，会标记为这个 注意：这里是set集合 不会重复
	// 至少被创建了一次的  都会放进这里~~~~
	private final Set<String> alreadyCreated = Collections.newSetFromMap(new ConcurrentHashMap<>(256));
}
```



注：`AbstractBeanFactory`继承自`DefaultSingletonBeanRegistry`~

1. `singletonObjects`：用于存放完全初始化好的 bean，**从该缓存中取出的 bean 可以直接使用**
2. `earlySingletonObjects`：提前曝光的单例对象的cache，存放原始的 bean 对象（尚未填充属性），用于解决循环依赖
3. `singletonFactories`：单例对象工厂的cache，存放 bean 工厂对象，用于解决循环依赖

