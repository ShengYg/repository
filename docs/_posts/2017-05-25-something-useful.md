---
layout: post
title:  "something useful"
date:   2017-05-25 14:00:00 +0800
categories: [algorithms]
tags: [java]
description: 
---

## Table of Contents

- java
- 算法

---

## java

### 1. transformation

~~~java
int y = Integer.parseInt("40");
String y = String.valueOf(40);

char[] ret = (String).toCharArray();
String ret = String.valueOf(char[]);

String[] ret = (String).split(...);
String ret = (String[]).join(...);

import java.util.stream.*;
// list -> Integer[]
Integer[] arr_integer = list.toArray(new Integer[list.size()]);
// list -> int[]
int[] arr_int = list.stream().mapToInt(i->i).toArray();
// int[] -> Integer[]
Integer[] arr_integer = Arrays.stream(arr_int).boxed().toArray(Integer[]::new);
// Integer[] -> list
ArrayList<Integer> list = new ArrayList<Integer>(Arrays.asList(arr_integer));
// int[] -> list
ArrayList<Integer> list = Arrays.stream(arr_int).boxed().collect(Collectors.toCollection(ArrayList::new));

// list -> list
list.stream()...collect(Collectors.toCollection(ArrayList::new));
// list -> int[]
list.stream()...toArray();
// list -> Integer[]
list.stream()...toArray(Integer[]::new);
// int[] -> int[]
Arrays.stream(arr_int)...toArray();
// int[] -> list
Arrays.stream(arr_int).boxed()...collect(Collectors.toCollection(ArrayList::new));
~~~

### 2. BigInteger

Integer.MAX_VALUE = 2147483647;

BigInteger must support values in the range `-2^Integer.MAX_VALUE` (exclusive) to `+2^Integer.MAX_VALUE` (exclusive).

~~~java
import java.math.BigInteger;
BigInteger A = new BigInteger(String.valueOf(a));
~~~

### 3. tuple pairs

~~~java
import org.apache.commons.lang3.tuple.*;

ImmutableTriple t = new ImmutableTriple(l, m, r);
ImmutablePair p = new ImmutablePair(l, r);

class Pair<L, R>{
    L left;
    R right;
    Pair(L left, R right){
        this.left = left;
        this.right = right;
    }
}
~~~

### 4. lambda

~~~java
Collections.sort(list, (a, b) -> {
    if(a.left == b.left)
        return a.right - b.right;
    else
        return a.left - b.left;
});
~~~

### 5. map, reduce, filter
~~~java
List<Integer> map_num = list.stream().map(num -> num * 2).collect(Collectors.toList());
     
Optional<Integer> sum = list.stream().reduce((a, b) -> a + b);
sum.orElseGet(() -> 0);
     
List<Integer> filter_num = list.stream().filter(num -> num % 2 == 0).collect(Collectors.toList());
~~~

## 算法

### dp递推
- f(n) = f(n-1) + C
- f(i, j) = f(i, k) + f(k, j) + C
