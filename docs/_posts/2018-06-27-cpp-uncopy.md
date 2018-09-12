---
layout: post
title:  "CPP：DISALLOW_COPY_AND_ASSIGN"
date:   2018-06-27 14:00:00 +0800
categories: [cpp]
tags: []
description: C++宏定义限制类的拷贝、赋值构造函数
---

有时候，进行类设计时，会发现某个类的对象是独一无二的，没有完全相同的对象，也就是对该类对象做副本没有任何意义。因此，需要限制编译器自动生动的拷贝构造函数和赋值构造函数。一般参用下面的宏定义的方式进行限制，代码如下：

~~~cpp
// A macro to disallow the copy constructor and operator= functions 
// This should be used in the priavte:declarations for a class
#define    DISALLOW_COPY_AND_ASSIGN(TypeName) \
    TypeName(const TypeName&);                \
    TypeName& operator=(const TypeName&)

class Test {
public:
    Test(int t);
    ~Test();
private:
    DISALLOW_COPY_AND_ASSIGN(Test);
};
~~~

声明私有的拷贝构造函数和赋值构造函数，但不去定义实现它们，有三方面的作用：
1. 声明了拷贝构造函数和赋值函数，阻止了编译器暗自创建的专属版本。
2. 声明了private，阻止了外部对它们的调用。
3. 不定义它们，可以保证成员函数和友元函数调用它们时，产生一个连接错误。

上述解决方法，面对在成员函数和友元函数企图拷贝对象时，会产生**连接期间**错误。遵循错误发现越早越好的原则，我们希望将连接期错误移至**编译期**。

~~~cpp
class Uncopyable {
protected:
    Uncopyable() {}
    ~Uncopyable() {}
private:
    Uncopyable(const Uncopyable&);
    Uncopyable& operator=(const Uncopyable&);
};
class Test:private Uncopyable{
...                            //class不再声明copy构造函数或copy assignment操作符
};
~~~

当任何人(包括member函数或friend函数)尝试拷贝Test对象时，编译器便试着生成一个copy构造函数和一个copy assignment操作符。编译器自动生成这些函数时，会调用其基类的对应函数，而基类中这些函数是private，因而那些调用会被编译器拒绝，产生编译器错误。