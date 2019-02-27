---
layout: post
title:  "something useful"
date:   2017-05-25 14:00:00 +0800
categories: [java]
tags: [note]
description: 
---

### 1. init

~~~java
ArrayList<Integer> list = new ArrayList<>(Arrays.asList(0,1,2));

// HashMap key1
HashMap<String, Integer> map = new HashMap<>();
map.put(list.toString(), 0);

// HashMap key2
final class Bigram {
    private final String word1, word2;
    public Bigram(String word1, String word2) {
        this.word1 = word1;
        this.word2 = word2;
    }

    public String getWord1() {
        return word1;
    }
    public String getWord2() {
        return word2;
    }

    @Override
    public int hashCode() {
        return word1.hashCode() ^ word2.hashCode();
    }

    @Override
    public boolean equals(Object obj) {
        return (obj instanceof Bigram) && ((Bigram) obj).word1.equals(word1)
                                       && ((Bigram) obj).word2.equals(word2);
    }
}
~~~

### 2. transformation

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

### 3. BigInteger

Integer.MAX_VALUE = 2147483647;

BigInteger must support values in the range `-2^Integer.MAX_VALUE` (exclusive) to `+2^Integer.MAX_VALUE` (exclusive).

~~~java
import java.math.BigInteger;
BigInteger A = new BigInteger(String.valueOf(a));
~~~

### 4. tuple pairs

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

### 5. lambda

~~~java
Collections.sort(list, (a, b) -> {
    if(a.left == b.left)
        return a.right - b.right;
    else
        return a.left - b.left;
});

// 2D array sort
Arrays.sort(cir, new Comparator<float[]>() {
    public int compare(float[] a, float[] b) {
        return Float.compare(a[2], b[2]);
    }
});
Arrays.sort(array, Comparator.comparingDouble(a -> a[1]));
~~~

### 6. map, reduce, filter
~~~java
List<Integer> map_num = list.stream().map(num -> num * 2).collect(Collectors.toList());
     
Optional<Integer> sum = list.stream().reduce((a, b) -> a + b);
sum.orElseGet(() -> 0);
     
List<Integer> filter_num = list.stream().filter(num -> num % 2 == 0).collect(Collectors.toList());
~~~
