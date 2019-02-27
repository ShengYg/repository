---
layout: post
title:  "LeetCode calculator"
date:   2017-04-11 12:00:00 +0800
categories: [algorithms]
tags: [stack]
description: LeetCode 224, 227
---

### **Question 1:**

[LeetCode 224 Basic Calculator](https://leetcode.com/problems/basic-calculator/#/description)

Implement a basic calculator to evaluate a simple expression string.

The expression string may contain open `(` and closing parentheses `)`, the plus `+` or minus sign `-`, non-negative integers and empty spaces` `.

You may assume that the given expression is always valid.

Some examples:
~~~
"1 + 1" = 2
" 2-1 + 2 " = 3
"(1+(4+5+2)-3)+(6+8)" = 23
~~~

AC solution:
~~~python
def calculate(self, s):
    """
    :type s: str
    :rtype: int
    """
    stack = []
    res, num, sign = 0, 0, 1
    for c in s:
        if c.isdigit():
            num = num * 10 + int(c)
        elif c == '+':
            res += sign * num
            num = 0
            sign = 1
        elif c == '-':
            res += sign * num
            num = 0
            sign = -1
        elif c == '(':
            stack.append(res)
            stack.append(sign)
            sign = 1
            res = 0
        elif c == ')':
            res += sign * num
            num = 0
            res *= stack.pop() # sign
            res += stack.pop() # res
    if num:
        res += sign * num
    return res
~~~


### **Question 2:**

[LeetCode 227 Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/#/description)

Implement a basic calculator to evaluate a simple expression string.

The expression string contains only non-negative integers, `+`, `-`, `*`, `/` operators and empty spaces` `. The integer division should truncate toward zero.

You may assume that the given expression is always valid.

Some examples:
~~~
"3+2*2" = 7
" 3/2 " = 1
" 3+5 / 2 " = 5
~~~

AC solution:
~~~python
def calculate(self, s):
    """
    :type s: str
    :rtype: int
    """
    stack = []
    num, sign = 0, '+'
    for i in range(len(s)):
        c = s[i]
        if c.isdigit():
            num = num * 10 + int(c)
        if not c.isdigit() and c != ' ' or i == len(s)-1:
            if sign == '-':
                stack.append(-num)
            if sign == '+':
                stack.append(num)
            if sign == '*':
                stack.append(stack.pop() * num)
            if sign == '/':
                top = stack.pop()
                if top >= 0:
                    stack.append(top / num)
                else:
                    stack.append(-(abs(top) / num ))
            sign = c
            num = 0
    return sum(stack)
~~~

