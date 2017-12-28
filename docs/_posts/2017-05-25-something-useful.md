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

## cpp

### 1.init

~~~cpp
getline(cin, line);

vector<int> v(n, val);
vector<int> v = {1, 2, 3, 4};

set<int> iset(ivec.begin(), ivec.end());
set1.insert(set2.begin(), set2.end());

auto map_it = map.begin();
cout << map_it->first << map_it->second;

auto ret = map.insert(make_pair(key, value));
if(!ret.second)
	ret.first->second++;

class cmp{
public:
	bool operator()(pair<ll,ll> p1, pair<ll,ll> p2){
		return p1.first*p1.second > p2.first*p2.second;	// 由小到大
	}
};
priority_queue<pair<ll, ll>, vector<pair<ll, ll>>, cmp> q;

auto comp = [](pair<int,int> a, pair<int,int> b) { return a.second > b.second; };
priority_queue<pair<int, int>, vector<pair<int,int>>, decltype(comp)> p(comp);

reverse(v.begin(), v.end());
sort(v.begin(), v.end());
max_element(v.begin(), v.end());

numeric_limits<int>::max()
~~~

### 2.transformation

~~~cpp
int i = stoi(s);	//stof, stod, stol ,stoll...
string s = to_string(i);

strcpy(arr, string.c_str());
string s = arr;

vector<string> split(const  string& s, const string& delim) {
    vector<string> elems;
    size_t pos = 0;
    size_t len = s.length();
    size_t delim_len = delim.length();
    if (delim_len == 0) return elems;
    while (pos < len) {
        int find_pos = s.find(delim, pos);
        if (find_pos < 0) {
            elems.push_back(s.substr(pos, len - pos));
            break;
        }
        elems.push_back(s.substr(pos, find_pos - pos));
        pos = find_pos + delim_len;
    }
    return elems;
}

~~~
## 算法

### dp递推
- f(n) = f(n-1) + C
- f(i, j) = f(i, k) + f(k, j) + C
