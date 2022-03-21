---
title: JDK源码分析之Map
date: 2019-10-09 20:44:52 
tags: Java
---

![avatar](https://images.gitbook.cn/Fpy4Na_uWi3rK9M8kOcgYK7_uXrK)

java.util.Map

<!--more-->

### Map 简介

Map是java.util提供的一个接口，是存储键值对映射的容器。AbstractMap是由abstract修饰的Map的抽象实现。

Map 常用的实现类如下：

- **Hashtable**：Java 早期提供的一个哈希表实现，它是线程安全的，不支持 null 键和值，因为它的性能不如 ConcurrentHashMap，所以很少被推荐使用。
- **HashMap**：最常用的哈希表实现，如果程序中没有多线程的需求，HashMap 是一个很好的选择，支持 null 键和值，如果在多线程中可用 ConcurrentHashMap 替代。
- **TreeMap**：基于红黑树的一种提供顺序访问的 Map，自身实现了 key 的自然排序，也可以指定 Comparator 来自定义排序。
- **LinkedHashMap**：HashMap 的一个子类，保存了记录的插入顺序，可在遍历时保持与插入一样的顺序。



### HashMap 数据结构

HashMap 底层的数据是数组被称为哈希桶，每个桶存放的是链表，链表中的每个节点，就是 HashMap 中的每个元素。在 JDK 8 当链表长度大于等于 8 时，就会转成红黑树的数据结构，以提升查询和插入的效率。

> HashMap结构：哈希数组+链表/红黑树，key和value均可以为null
> 存储元素时，需要调用key的hashCode()方法，计算出一个哈希值
> 1.哈希值相同的元素，必定位于同一个哈希槽（链）上，但不能确定这两个元素是不是同位元素
> 在进一步判断key如果相等（必要时需要调用equals()方法）时，才能确定这两个元素属于同位元素
> 如果是存储同位元素，需要考虑是否允许覆盖旧值的问题
> 2.哈希值不同的元素，它们也可能位于同一个哈希槽（链）上，但它们肯定不是同位元素
>
> ConcurrentHashMap结构：哈希数组+链表/红黑树，key和value均不能为null，ConcurrentHashMap的操作方式跟HashMap是基本统一的，不同之处在于，ConcurrentHashMap是线程安全的，其中用到了无锁编程（1.7是分段锁，1.8是CAS）。获取map.size，里面有一个final修饰的sumCount()方法，在基于CAS更新的baseCount基础上，不断遍历累加大小为2的幂的数组CounterCell（本质是一个分开计数的容器）中元素的value值。

HashMap 数据结构，如下图：

![](https://images.gitbook.cn/54a52ca0-ccc7-11e9-b229-e35eb1d6e740)

#### 1.7和1.8之间的变化

1.7之前是数组+链表，数组节点是一个Entry的内部类

> 问题：
>
> 1.数据插入使用了头插法，导致了resize扩容问题，resize调用了transfer方法，把里面的Node进行了一个rehash，这个过程可能会造成链表的循环，就可能在下一次get()的时候出现死循环的情况；
>
> 2.因为没有加锁，多线程运行时候，无法保证线程安全

1.8之后是数组+链表+红黑树，把原来的一个Entry节点改成了Node节点（静态内部类），整个put过程做了一个优化

> 扩容机制：
>
> capacity容量：HashMap初始化的时候，如果没有设置capacity，默认的容量是16，loadFactor是0.75，会计算出来一个扩容的阈值threshold。
>
> 判断当前map.size()是否>阈值，如果大于会新创建一个2倍大小容量来将原来的size进行resize()

ConcurrentHashMap集合容器，对比HashTable, Synchronized, Lock, Collection.Synchronized线程同步方式

并发度是更高的，HashTable是直接对内部的方法做了Synchronized，加了一把对象锁

ConcurrentHashMap只会锁住当前获取到的Entry所在节点的值，锁的粒度是更细的，并且在上锁的时候使用的是CAS+Synchronized，因此效率是更高的，并发度是更高的，并发支持更好。

#### JDK1.6之后对Synchronized的优化

锁升级：无锁>偏向锁>自旋锁>重量级锁

>  先判断是否需要加锁，不涉及并发处理就是无锁，其次需要锁的时候支持偏向锁，获取到资源的线程，会让它再次获取锁，如果没有获取到就升级成一个轻量级的锁，CAS的乐观锁；如果CAS没有设置成功它会进行一个自旋，自旋到一定的次数之后才会升级成一个Synchronized的重量级的锁。

### hash算法优化

```java
    // JDK1.8源码参考
	static final int hash(Object key) {
        int h;
        // 原哈希值与右移16位的值进行亦或运算
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

> 1111 1111 1111 1111 1010 1110 0011 1001
>
> 0000 0000 0000 0000 1111 1111 1111 1111
>
> 1111 1111 1111 1111 0101 0001 1100 0110
>
> int类型，32位（4字节）长度
>
> 高低16位都参与运算，最终结果等价于保留高16位，低16位由高低16位亦或求得结果
>
> 相当于对低16位进行了转码处理，可以融合高低16位的特征，尽量避免后续的hash冲突

### 寻址算法优化

(n-1) & hash -> 数组中的某个位置

直接用哈希值对数组长度取模，性能比较差，为了优化这个寻址过程，确保hash&(n-1)和hash%n效果是一样的，但是与运算的性能要比hash对n取模高很多。

### hash碰撞

数组 + 链表， 遍历一遍是O(n)，链表很长的话，性能比较差。

优化：链表长度达到8之后，会把链表转换为红黑树，遍历**红黑树**查找某个元素时，需要O(logn)，性能比链表更高一些。

### HashMap 重要方法

#### 1）添加方法：put(Object key, Object value)

执行流程如下：

- 对 key 进行 hash 操作，计算存储 index；
- 判断是否有哈希碰撞，如果没碰撞直接放到哈希桶里，如果有碰撞则以链表的形式存储；
- 判断已有元素的类型，决定是追加树还是追加链表，当链表大于等于 8 时，把链表转换成红黑树；
- 如果节点已经存在就替换旧值；
- 判断是否超过阀值，如果超过就要扩容。

put() 执行流程图如下：

![enter image description here](https://images.gitbook.cn/727836f0-ccc7-11e9-a9bd-857608719494)

#### 2）获取方法：get(Object key)

执行流程如下：

- 首先比对首节点，如果首节点的 hash 值和 key 的 hash 值相同，并且首节点的键对象和 key 相同（地址相同或 equals 相等），则返回该节点；
- 如果首节点比对不相同、那么看看是否存在下一个节点，如果存在的话，可以继续比对，如果不存在就意味着 key 没有匹配的键值对。

### 相关问题

#### 1.Map 常见实现类有哪些？

答：Map 的常见实现类如下列表：

- Hashtable：Java 早期提供的一个哈希表实现，它是线程安全的，不支持 null 键和值，因为它的性能不如 ConcurrentHashMap，所以很少被推荐使用；
- HashMap：最常用的哈希表实现，如果程序中没有多线程的需求，HashMap 是一个很好的选择，支持 null 键和值，如果在多线程中可用 ConcurrentHashMap 替代；
- TreeMap：基于红黑树的一种提供顺序访问的 Map，自身实现了 key 的自然排序，也可以指定的 Comparator 来自定义排序；
- LinkedHashMap：HashMap 的一个子类，保存了记录的插入顺序，可在遍历时保持与插入一样的顺序。

#### 2.使用 HashMap 可能会遇到什么问题？如何避免？

答：HashMap 在并发场景中可能出现死循环的问题，这是因为 HashMap 在扩容的时候会对链表进行一次倒序处理，假设两个线程同时执行扩容操作，第一个线程正在执行 B→A 的时候，第二个线程又执行了 A→B ，这个时候就会出现 B→A→B 的问题，造成死循环。
解决的方法：升级 JDK 版本，在 JDK 8 之后扩容不会再进行倒序，因此死循环的问题得到了极大的改善，但这不是终极的方案，因为 HashMap 本来就不是用在多线程版本下的，如果是多线程可使用 ConcurrentHashMap 替代 HashMap。

