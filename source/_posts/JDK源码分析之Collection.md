---
title: JDK源码分析之Collection
date: 2019-10-08 20:44:52 
tags: Java
---

![enter image description here](https://images.gitbook.cn/ae489970-ca62-11e9-bd50-998f3938aecb)

集合的继承关系图中，看出集合的根节点是 Collection，而 Collection 下又提供了两大常用集合，分别是：

<!--more-->

- List：使用最多的有序集合，提供方便的新增、修改、删除的操作；
- Set：集合不允许有重复的元素，在许多需要保证元素唯一性的场景中使用。

集合使用：

#### 1）Vector

Vector 是 Java 早期提供的线程安全的有序集合，实现了同步但是效率低已经不用了，如果不需要线程安全，不建议使用此集合，毕竟同步是有线程开销的。Stack继承了Vector

使用示例代码：

```java
Vector vector = new Vector();
vector.add("dog");
vector.add("cat");
vector.remove("cat");
System.out.println(vector);
```

程序执行结果：`[dog]`

#### 2）ArrayList

ArrayList 是最常见的非线程安全的有序集合，因为内部是**数组**存储的，所以**随机访问**效率很高，但非尾部的插入和删除性能较低，如果在中间插入元素，之后的所有元素都要后移。ArrayList 的使用与 Vector 类似。

#### 3）LinkedList

LinkedList 是使用**双向链表**数据结构实现的，因此**增加和删除**效率比较高，而随机访问效率较差。

LinkedList是个双向链表，它同样可以被当作**栈、队列或双端队列**来使用

LinkedList 除了包含以上两个类的操作方法之外，还新增了几个操作方法，如 offer() 、peek() 等，具体详情，请参考以下代码：

```java
LinkedList linkedList = new LinkedList();
// 添加元素
linkedList.offer("bird");
linkedList.push("cat");
linkedList.push("dog");
// 获取第一个元素
System.out.println(linkedList.peek());
// 获取第一个元素，并删除此元素
System.out.println(linkedList.poll());
System.out.println(linkedList);
```

程序的执行结果：

```
dog
dog
[cat, bird]
```

#### 4）HashSet

HashSet 是一个没有重复元素的集合。虽然它是 Set 集合的子类，实际却为 **HashMap** 的实例，相关源码如下：

```
public HashSet() {
    map = new HashMap<>();
}
```

因此 HashSet 是**无序集合**，没有办法保证元素的顺序性。

HashSet 默认容量为 **16**，每次扩充 **0.75** 倍，相关源码如下：

```
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}
```

HashSet 的使用与 Vector 类似。

#### 5）TreeSet

TreeSet 集合实现了**自动排序**，也就是说 TreeSet 会把你插入数据进行自动排序。

示例代码如下：

```
TreeSet treeSet = new TreeSet();
treeSet.add("dog");
treeSet.add("camel");
treeSet.add("cat");
treeSet.add("ant");
System.out.println(treeSet);
```

程序执行结果：`[ant, camel, cat, dog]`

可以看出，TreeSet 的使用与 Vector 类似，只是实现了自动排序。

#### 6）LinkedHashSet

LinkedHashSet 是按照元素的 hashCode 值来决定元素的存储位置，但同时又使用链表来维护元素的次序，这样使得它看起来像是按照插入顺序保存的。

### 相关问题

#### 1.List 和 Set 有什么区别？

- List 允许有多个 null 值，Set 只允许有一个 null 值；
- List 允许有重复元素，Set 不允许有重复元素；
- List 可以保证每个元素的存储顺序，Set 无法保证元素的存储顺序。

#### 2.Collection 和 Collections 有什么区别？

答：Collection 和 Collections 的区别如下：

- Collection 是集合类的上级接口，继承它的主要有 List 和 Set；
- Collections 是针对集合类的一个帮助类，它提供了一些列的静态方法实现，如 Collections.sort() 排序、Collections.reverse() 逆序等。

#### 3.LinkedHashSet 如何保证有序和唯一性？

答：LinkedHashSet 底层数据结构由哈希表和链表组成，链表保证了元素的有序即存储和取出一致，哈希表保证了元素的唯一性。

#### 4.HashSet 是如何保证数据不可重复的？

答：HashSet 的底层其实就是 HashMap，只不过 HashSet 实现了 Set 接口并且把数据作为 K 值，而 V 值一直使用一个相同的虚值来保存，我们可以看到源码：

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;// 调用 HashMap 的 put 方法,PRESENT 是一个至始至终都相同的虚值
}
```

由于 HashMap 的 K 值本身就不允许重复，并且在 HashMap 中如果 K/V 相同时，会用新的 V 覆盖掉旧的 V，然后返回旧的 V，那么在 HashSet 中执行这一句话始终会返回一个 false，导致插入失败，这样就保证了数据的不可重复性。