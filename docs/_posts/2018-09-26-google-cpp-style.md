---
layout: post
title:  "google c++ style guide note"
date:   2018-09-26 14:00:00 +0800
categories: [cpp]
tags: []
description: 
---

- 目录
{:toc #markdown-toc}

## 头文件
### 头文件多重包含
- 问题：产生重复定义
- `#define`保护，注意宏名字不能冲突。编译器每次需要打开头文件判断是否冲突，编译时间较长
~~~cpp
// foo/src/bar/baz.h
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_
int a;
#endif
~~~
- `#pragma once`，由编译器提供保证

### 前置声明
~~~cpp
# include "A.h"
class B{
    A* p;
}

class A;
class B{
    A* p;
}
~~~
- 优点
    - 节省编译时间，`#include`会展开更多文件
    - 节省重新编译时间，修改头文件不需要重新编译
- 缺点（会带来诸多隐藏错误，增加程序员负担）
    - 妨碍头文件开发者
    - 存在继承关系时
- 建议：函数总是使用`#include`，类模板优先使用`#include`？？？？

### include顺序
1. 当前`.cc`文件对应的头文件（优先对当前的头文件进行测试）
2. C系统文件，C++系统文件
3. 其他库`.h`文件，基于平台的条件编译
4. 本项目包含`.h`文件

~~~cpp
#include "foo/public/fooserver.h"   // 优先位置

#include <sys/types.h>
#include <unistd.h>

#include <hash_map>
#include <vector>

#include "base/basictypes.h"
#include "base/commandlineflags.h"
#include "foo/public/bar.h"
~~~

## 作用域
### 匿名命名空间和静态变量
- 作用：用来定义一个不需要被外部引用的变量
- 在`.cc`中使用，不在`.h`中使用（会在每个引用的`.cc`文件中各自生成一个变量）

### 非成员函数、静态函数、全局函数
- 建议
    - **非成员函数放在命名空间**，不使用全局函数
    - 不要用类的静态函数模拟命名空间
- 原因：一个编译单元的函数被另一个编译单元调用时，会引入链接时的依赖性

### 静态变量、全局变量
- 静态生存周期的对象，包括**全局变量、静态变量、静态类成员变量、函数静态变量**，都必须是原生数据类型
- 原因：同一个编译单元内静态初始化优先于动态初始化，初始化、销毁顺序确定。不同的编译单元之间初始化和销毁顺序属于未明确行为。

## 函数
### 引用参数
- 输入参数必须为值或`const`引用
- 有时可能会用`const`指针，涉及到空指针或地址操作时

### 缺省参数
- 虚函数不能使用缺省函数，因为缺省值静态绑定

## 其他
### RTTI
- 不要滥用RTTI，除非是单元测试
- 好处：单元测试时，用来检测工厂方法新建对象是否为期望值

### 流
- 使用`printf`代替流，除非是记录日志
- 缺点：流被重载，输出时不进行类型检查；某些格式化操作效率很低

### 使用前置自增/自减
- 后置自增/自减会额外执行一次复制

