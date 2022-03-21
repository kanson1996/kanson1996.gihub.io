---
title: Spring设计模式
date: 2019-10-23 21:51:12
tags:
	- Spring
	- 设计模式
categories: 技术
---

### Spring设计模式

#### 工厂模式

spring ioc的核心设计模式的思想体现，本身就是一个大的工厂，把所有的bean实例都放在了spring 容器里（大工厂），如果要使用bean就直接找spring容器就好了，自己不用创建对象。

<!--more-->

```java
public class MyController {
    // 原先每次都要自己new一个
	private MyService mySerice = new MyService();
    // 使用spring @Autowire注解自动注入一个，实际底层是依靠的工厂模式去创建的
    private MyService mySerice = MyServiceFactory.getMyService();
}

public class MyServiceFactory {
    public static MyService getMyService() {
        return new MyServiceImpl();
    }
}
```

#### 单例模式

spring默认对每个bean都走的单例模式，确保一个类在系统运行期间只有一个实例对象，只有一个bean

```java
public class MyService {
	
	private static volatile Myservice myService; // 禁止指令重排序
	
	public stativ MyService getInstance() {
		if (myService == null) {
			synchronized(MyService.class) {
                if (myService == null) {
                    myService = new MyService();
                }
            }
		}
        return myService;
	}
}
```

#### 代理模式

AOP的核心模式，如果要对一些类的方法切入一些增强的代码，会创建一些动态代理的对象，让你对那些目标对象的访问，先经过动态代理对象，动态代理对象先执行一些增强的代码，再调用你的目标对象。在设计模式里，就是代理模式的体现和运用，实现一些增强的访问。