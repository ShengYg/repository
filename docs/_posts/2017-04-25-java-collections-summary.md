---
layout: post
title:  "java集合类总结"
date:   2017-04-25 12:00:00 +0800
categories: [java]
tags: [collections]
description: java集合类总结
---

![]({{ site.baseurl }}/assets/pic/8_00.png)


#### ArrayList

~~~java
boolean	add(E e)
void	add(int index, E element) 
E	remove(int index)
E	set(int index, E element)
E	get(int index)
List<E>	subList(int fromIndex, int toIndex)

boolean	contains(Object o)
int	indexOf(Object o)
boolean	isEmpty()
int	size()
~~~

#### LinkedList

LinkedList can be used as **Stack**, **Queue**

~~~java
void	addFirst(E e)
void	addLast(E e)
E	getFirst()
E	getLast()
E	removeFirst()
E	removeLast()
~~~

#### PriorityQueue

~~~java
PriorityQueue(Comparator<? super E> comparator)

boolean	add(E e)
boolean	contains(Object o)
E	peek() // Retrieves, but does not remove, the head of this queue
E	poll() // Retrieves and removes the head of this queue
boolean	remove(Object o)
~~~

#### HashSet, LinkedHashSet(insertion-order)

~~~java
boolean	add(E e)
boolean	remove(Object o)
boolean	removeAll(Collection<?> c)

boolean	contains(Object o)
boolean	containsAll(Collection<?> c)
boolean	isEmpty()
int	size()
~~~

#### TreeSet

The elements are **ordered** using their natural ordering, or by a **Comparator** provided at set creation time, depending on which constructor is used.

~~~java
TreeSet(Comparator<? super E> comparator) // sorted according to the specified comparator.

E	first()
E	last()
E	floor(E e) //the greatest element less than or equal to the given element
E	ceiling(E e) //the least element greater than or equal to the given element
E	pollFirst()
E	pollLast()
~~~

#### HashMap, LinkedHashMap

~~~java
V	put(K key, V value)
V	putIfAbsent(K key, V value)
V	get(Object key)
V	getOrDefault(Object key, V defaultValue)
V	remove(Object key)
boolean	remove(Object key, Object value)
V	replace(K key, V value)
boolean	replace(K key, V oldValue, V newValue)
boolean	containsKey(Object key)
boolean	containsValue(Object value)

Set<K>		keySet()
Collection<V>	values()
boolean		isEmpty()
int		size()
~~~

