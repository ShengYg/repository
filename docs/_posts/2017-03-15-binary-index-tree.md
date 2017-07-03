---
layout: post
title:  "Binary Indexed Trees 树状数组"
date:   2017-03-15 12:00:00 +0800
categories: [algorithms]
tags: [tree]
description: 本文翻译自TopCoder上的一篇文章： Binary Indexed Trees 
---

本文翻译自TopCoder上的一篇文章： [Binary Indexed Trees](https://www.topcoder.com/community/data-science/data-science-tutorials/binary-indexed-trees/)

## 1. 基本思想

![]({{ site.url }}/assets/pic/5_00.png)

<div class="fig figcenter fighighlight">
  <img src="{{ site.url }}/assets/pic/5_00.png">
  <div class="figcaption">假设我们要得到索引为13的累积频率(即c[13])，在二进制表示中，13=1101。因此， 我们可以这样计算：c[1101]=tree[1101]+tree[1100]+tree[1000]</div>
</div>

假设我们要得到索引为13的累积频率(即c[13])，在二进制表示中，13=1101。因此， 我们可以这样计算：c[1101]=tree[1101]+tree[1100]+tree[1000]
