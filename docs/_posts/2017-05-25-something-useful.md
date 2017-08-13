---
layout: post
title:  "something useful"
date:   2017-05-25 14:00:00 +0800
categories: [algorithms]
tags: [java]
description: 
---


### 1. transformation

~~~java
int y = Integer.parseInt("40");
String y = String.valueOf(40);

char[] ret = (String).toCharArray();
String ret = String.valueOf(char[]);

String[] ret = (String).split(...);
String ret = (String[]).join(...);

int[] ret = (List).toArray(new int[0]);
Integer[] arr_integer = Arrays.stream(arr_int).boxed().toArray(Integer[]::new);
ArrayList<Integer> ret = new ArrayList<Integer>(Arrays.asList(arr_integer));
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
~~~
