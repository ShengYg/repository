---
layout: post
title:  "Python itertools"
date:   2017-07-20 12:00:00 +0800
categories: [python]
tags: []
description: Python itertools 常用函数总结
---

~~~
## 链式迭代展开
> list(chain('123', '456'))
['1', '2', '3', '4', '5', '6']

## 条件选择
> list(compress('ABCDEF', [1,0,1,0,1,1]))
['A', 'C', 'E', 'F']

## 两种map
> list(imap(pow, (2,3,10), (5,2,3)))
[32, 9, 1000]
> list(starmap(pow, [(2,5), (3,2), (10,3)]))
[32, 9, 1000]

## 之字形迭代，可多个
> list(izip_longest('1234', 'abc', fillvalue='-'))
[('1', 'a'), ('2', 'b'), ('3', 'c'), ('4', '-')]
~~~

~~~
> a = [1,2,3]

## 所有排列情况，相当于for循环
> list(product(a, repeat=2))
[(1, 1), (1, 2), (1, 3), (2, 1), (2, 2), (2, 3), (3, 1), (3, 2), (3, 3)]

## 排列，有序
> list(permutations(a))
[(1, 2, 3), (1, 3, 2), (2, 1, 3), (2, 3, 1), (3, 1, 2), (3, 2, 1)]
> list(permutations(a,2))
[(1, 2), (1, 3), (2, 1), (2, 3), (3, 1), (3, 2)]

## 组合，无序
> list(combinations(a, 2))
[(1, 2), (1, 3), (2, 3)]

## 组合，无序，包含重复元素
> list(combinations_with_replacement(a, 2))
[(1, 1), (1, 2), (1, 3), (2, 2), (2, 3), (3, 3)]
~~~
