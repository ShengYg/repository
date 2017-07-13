---
layout: post
title:  "Java String"
date:   2017-05-10 12:00:00 +0800
categories: [java]
tags: [string]
description: java String
---

## Table of Contents

- [不可变String](#1)
- [重载‘+’与StringBuilder](#2)
- [常用String操作](#3)
- [正则表达式](#4)
- [扫描输入](#5)

---

<a name='1'></a>
## 不可变String

String对象是**不可变的**。String类中每一个看起来会修改String的方法，实际上都是创建了一个新的String对象。

<a name='2'></a>
## 重载‘+’与StringBuilder

不可变性会带来效率问题。用于String的‘+’与‘+=’是Java中仅有的两个重载过得操作符，而Java并不允许程序员重载任何操作符。

~~~java
String s = "abc" + "def";
~~~

上述代码运行时，编译器会创建`StringBuilder`对象，并为每个字符串调用`append()`方法，最后调用`toString()`生成结果。

~~~java
public String implicit(String[] fields){
    String result = "";
    for(int i = 0; i < fields.length; i++)
        result += fields[i];
    return result;
}

public String explicit(String[] fields){
    StringBuilder result = new StringBuilder();
    for(int i = 0; i < fields.length; i++)
        result.append(fields[i]);
    return result.toString();
}
~~~

当在`toString()`方法中使用循环时，，每经过一次循环，就会创建一个新的`StringBuilder`对象。这种情况下，最好显示的创建一个`StringBuilder`对象，用它来构造会后结果。

`StringBuffer`是线程安全的，因此开销更大。

<a name='3'></a>
## 常用String操作

~~~java
char	charAt(int index)
int	    length()

boolean	contains(CharSequence s)
boolean	endsWith(String suffix)
boolean	startsWith(String prefix)
int	    compareTo(String anotherString)

String	    concat(String str)
String	    substring(int beginIndex)
String	    join(CharSequence delimiter, CharSequence... elements)
String[]    split(String regex)

char[]	toCharArray()

boolean	matches(String regex)
String	replace(char oldChar, char newChar)
String	replaceAll(String regex, String replacement)
~~~

<a name='4'></a>
## 正则表达式

在Java中，`\\`表示“我要插入一个正则表达式的反斜线”。因此如果要表示一位数字，正则表达式应该是`\\d`。如果想插入普通的反斜线，应该是`\\\\`。

字符类

~~~
.               任意字符
[abc]	        a|b|c
[^abc]	        ^(a|b|c)
[a-zA-Z]	
[a-d[m-p]]	    [a-d|m-p]
\d	            [0-9]
\D	            [^0-9]
\s	            空白字符: [ \t\n\x0B\f\r]
\w	            [a-zA-Z_0-9]
~~~

边界匹配

~~~
^	            一行的起始
$	            一行的结束
\b	            词的边界
\B	            非词的边界
\G	            前一匹配结束
~~~

量词

- 贪婪型：为所有可能模式发现尽可能多的匹配
- 勉强型：用**问号**来指定匹配模式所需的最少字符数
- 占有型：（只在Java中可用）当正则表达式应用于字符串时，在匹配失败时会回溯。占有型量词不会保存中间状态，防止回溯，执行起来更有效。

|贪婪型|勉强型|占有型|如何匹配|
|---|---|---|---|
|X?|X??|X?+|一个或零个X|
|X*|X*?|X*+|零个或多个X|
|X+|X+?|X++|一个或多个X|
|X{n}|X{n}?|X{n}+|n个X|
|X{n,}|X{n,}?|X{n,}+|至少n个X|
|X{n,m}|X{n,m}?|X{n,m}+|n-m个X|

Pattern和Matcher

~~~java
import java.util.regex.*;

String s = "..."
String regex = "..."
Pattern p = Pattern.compile(regex);
Matcher m = p.matcher(s);
~~~

编译后的Pattern对象提供了`split()`方法，它从匹配regex的地方分割字符串，返回String数组。

~~~java
Matcher m = Pattern.compile("\\w+").matcher("...")
while(m.find())
    System.out.println(m.group());
~~~

组(Groups)

组号0表示整个字符串，组号1表示第一对括号括起的组，依次类推。
在表达式`A(b(C))D`中，组0是`ABCD`，组1是`BC`，组2是`C`。

~~~java
String	group()             // 返回前一次匹配操作的第0组
String	group(int group)
int	    start()             // 先前匹配的起始位置的索引
int	    end()
~~~

<a name='5'></a>
## 扫描输入

#### Scanner

默认情况下，Scanner根据空白字符对输入进行分词，也可以自己指定。

~~~java
Scanner scanner= new Scanner("...");
while(scanner.hasNextInt())
    System.out.println(scanner.nextInt());
~~~














