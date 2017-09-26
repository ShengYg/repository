---
layout: post
title:  "Java HashSet工作原理及实现"
date:   2017-05-07 12:00:00 +0800
categories: [java]
tags: [set]
description: java HashSet, LinkedHashSet, TreeSet
---

## Table of Contents

- [HashSet](#1)
- [LinkedHashSet](#2)
- [TreeSet](#3)

---

<a name='1'></a>
## HashSet

HashSet是基于HashMap来实现的，操作很简单，更像是对HashMap做了一次“封装”，而且只使用了HashMap的key来实现各种特性

~~~java
private transient HashMap<E,Object> map;
// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();
~~~

`map`是整个HashSet的核心，而`PRESENT`则是用来造一个假的value来用的。

### 基本操作

~~~java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}
public boolean contains(Object o) {
    return map.containsKey(o);
}
public int size() {
    return map.size();
}
~~~

<a name='2'></a>
## LinkedHashSet

LinkedHashSet是基于HashMap和双向链表的实现，其实现基本和LinkedHashMap一样。

~~~java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable

~~~

<a name='3'></a>
## TreeSet

TreeSet是基于TreeMap实现的，也非常简单，同样的只是用key及其操作，然后把value置为dummy的object。



