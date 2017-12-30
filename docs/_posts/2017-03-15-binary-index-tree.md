---
layout: post
title:  "Binary Indexed Trees 树状数组"
date:   2017-03-15 12:00:00 +0800
categories: [algorithms]
tags: [tree]
description: 本文转载自[树状数组(Binary Indexed Trees)]
---

本文转载自[树状数组(Binary Indexed Trees)](http://www.hawstein.com/posts/binary-indexed-trees.html)，有整理。

> 注意：`数组index从1开始`

## 符号含义

- BIT: 树状数组
- MaxVal: 问题规模或数组大小n
- f[i]: 索引为i的频率值，即原始数组中第i个值。
- c[i]: 索引为i的累积频率值，c[i]=f[1]+f[2]+…+f[i]
- tree[i]: 索引为i的BIT值(下文会介绍它的定义)

## 1. 基本思想

![]({{ site.baseurl }}/assets/pic/5_00.png)

假设我们要得到索引为13的累积频率(即c[13])，在二进制表示中，`13=1101`。因此， 我们可以这样计算：c[1101]=tree[1101]+tree[1100]+tree[1000]。

## 2. 分离出最后的1

最后的1表示一个整数的二进制表示中，从左向右数最后的那个1。

由于我们经常需要将一个二进制数的最后的1提取出来，因此采用一种高效的方式来做这件事是十分有必要的。

令num是我们要操作的整数。我们知道，对一个数取负等价于对该数的二进制表示取反加1。我们可以通过与操作(符号&)将num中最后的1分离出来。

> num & -num

## 3. 读取累积频率

给定索引`idx`，如果我们想获取累积频率即`c[idx]`，我们只需初始化`sum=0`, 然后当`idx>0`时，重复以下操作：`sum`加上`tree[idx]`, 然后将`idx`最后的1去掉。

~~~java
int sum(int idx):
	ret = 0;
	for(int i = idx; i > 0; i -= i & -i) {
		ret += tree[i];
	}
	return ret;
~~~

二维
~~~java
int sum(int row, int col, int[][] arr) {
    int ret = 0;
    for(int i = row; i > 0; i -= i & -i) {
        for(int j = col; j > 0; j -= j & -j) {
            ret += tree[i][j];
        }
    }
    return ret;
}
~~~

例如，当idx=13时，初始sum=0，`c[1101]=f[1]+...+f[13]=tree[1101]+tree[1100]+tree[1000]`

![]({{ site.baseurl }}/assets/pic/5_01.png)

## 4. 改变某个位置的频率并且更新数组

在读取累积频率一节，我们每累加一次`tree[idx]`，就将`idx`最后一个1移除， 然后重复该操作。而如果我们改变了f数组，比如`f[idx]`增加`val`，我们则需要为当前索引的tree数组增加`val`：`tree[idx]　+= val`。然后`idx`更新为`idx`加上其最后的一个`1`。当`idx`不大于`MaxVal`时，不断重复上面的两个操作。

~~~cpp
void update(int idx, int val):
	for(int i = idx; i <= N; i += i & -i) {
		tree[i][j] += val;
	}
~~~

二维
~~~java
void update(int row, int col, int val, int N, int[][] arr) {
    for(int i = row; i <= N; i += i & -i) {
        for(int j = col; j <= N; j += j & -j) {
            tree[i][j] += val;
        }
    }
}
~~~

![]({{ site.baseurl }}/assets/pic/5_02.png)

## 5. 读取某个位置的实际频率

要得到`f[idx]`是一件非常容易的事：`f[idx] = read[idx] - read[idx-1]`。即前`idx`个数的和减去前`idx-1`个数的和， 然后就是`f[idx]`了。这种方法的时间复杂度是`2*O(log n)`。下面我们将重新写一个函数， 来得到一个稍快一点的版本。

假如我们要求f[12]，很明显它等于c[12]-c[11]。根据上文讨论的规律，有如下的等式:

~~~
c[12]=c[1100]=tree[1100]+tree[1000]
c[11]=c[1011]=tree[1011]+tree[1010]+tree[1000]
f[12]=c[12]-c[11]=tree[1100]-tree[1011]-tree[1010]
~~~

c[12]和c[11]中包含公共部分，而这个公共部分在实际计算中是可以不计算进来的。

~~~python
def get_single(idx):
    sum = tree[idx]
    if idx > 0:
        z = idx - idx & -idx
        idx--
        while idx != z:
            sum -= tree[idx]
            idx -= idx & -idx
    return sum
~~~







