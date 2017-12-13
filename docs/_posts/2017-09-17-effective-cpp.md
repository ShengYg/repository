---
layout: post
title:  "effective cpp note"
date:   2017-09-17 14:00:00 +0800
toc: true
categories: [cpp]
tags: [note]
description: effective c++ note
---

<a name='02'></a>
### 02. 尽量以const、enum、inline、代替#define
#### const

~~~cpp
#define RATIO 1.5
const double ratio=1.5;
~~~
`#define`是预处理器，当出现编译错误信息时，错误信息是`1.5`，而非`RATIO`。通常将其定义成`const`常量。

两种特殊情况：
1. 常量指针。由于常量定义式通常在头文件中，因此有必要将指针声明为`const`。
2. class专属常量。通常c++的任何东西都要有定义式，但class静态常量除外，只要不取它的地址。

~~~cpp
const char* const name = "Scott";	// 两个const
const string name("Scott");	// 建议用string

class A(){
private:
	static const int Num = 5;	// 常量声明式
	int arr[Num];
}
const int A::Num;			// 常量定义式，由于在声明时有初值，此处不能再设值
~~~
#### enum
`enum`的行为更像`#define`而非`const`。
~~~cpp
class A(){
private:
	enum { Num = 5 };	// 让Num成为5的一个记号。
	int arr[Num];
}
~~~
#### inline
想要用宏定义函数时，将其改成`template inline`函数。

<a name='03'></a>
### 03. 尽可能使用const
~~~cpp
const char* const p = "hello";	// 第一个const表示指针指向的数据是const
				// 第二个const表示指针本身是const
const int& f (const& int) const { }	// 最后一个const表示成员函数不修改成员值
~~~
#### const成员函数
成员函数只是常量性不同，也会被重载。const成员函数不能修改任何非`static`成员变量。

两种情况：
1. 修改部分成员：`mutable`成员在const成员函数中也会被修改
2. 进行同样操作的`const`和`non-const`函数会冗余：让non-const版本调用const版本

~~~cpp
class A{
private:
	mutable int a;
}

class A{
public:
	const char& operator[](int pos) const { }
	char& operator[](int pos){
		return const_cast<char&>(static_cast<const A&>(*this)[pos]);
	}
// 第一次在*this加上const，是的它调用const版本函数。第二次将函数返回值的const去掉。
}
~~~

<a name='04'></a>
### 04. 确定对象使用前已被初始化
1. 在所有对象使用之前先初始化

2. 成员变量初始化在构造函数之前，因此构造函数的最佳写法是用初始化列表

如果成员变量是`const`或`reference`，则一定需要初值，不能被赋值。

成员变量以声明次序初始化，与初始化列表顺序无关。

3.不同编译单元内的**非局部静态对象**初始化次序没有明确定义

解决方法：将非局部静态对象搬到函数内，函数返回指向这个对象的引用
~~~cpp
A& f(){
	static A a;
	return a;
}
~~~

<a name='05'></a>
### 05. 了解c++默认调用哪些函数
含有`reference`和`const`成员的类，编译器不会生成拷贝构造函数。

基类的拷贝成员函数为`private`，派生类不会生成拷贝构造函数。

<a name='06'></a>
### 06. 禁止编译器生成的函数

禁止拷贝构造函数，就将其声明为`private`，并且不实现它
~~~cpp
class A{
private:
	A(const A&);
	A& operator=(const A&);
}
~~~
如果在成员函数或友元函数中调用它，会发生连接器错误。可以通过定义一个基类将错误提前至编译器。
~~~cpp
class Uncopyable{
protected:
	Uncopyable() {}
	~Uncopyable() {}
private:
	Uncopyable(const Uncopyable&) {};
	Uncopyable& operator=(const Uncopyable&){};
}

class A: private Uncopyable { }
~~~

<a name='07'></a>
### 07. 虚析构函数
- 带有多态性质的基类应该有虚析构函数。只要类有虚函数，就要有虚析构函数。
- 如果类不是基类，或不具备多态性，就不该有虚析构函数。

<a name='09'></a>
### 09. 构造和析构函数不调用virtual成员函数

构造函数期间，virtual成员函数不具备多态性，不会下降至派生类成员函数。

<a name='11'></a>
### 11. 在operator=中处理自我赋值
~~~cpp
A& A::operator=(const A& rhs){
	Bitmap *p = pb;		// 先创建副本，再处理
	pb = new Bitmap(*ths.pb);
	delete p;
	return *this;
}

A& A::operator=(const A& rhs){
	A temp(rhs);
	swap(temp);		// copy and swap 技术
	return *this;
}
~~~

<a name='13'></a>
### 13. 以对象管理资源

- 用`shared_ptr`管理资源，以便做到自动释放资源，即使中途会发生不可测的事件
- 在资源管理类中发生拷贝操作时，采用智能指针而非普通指针，避免错误释放资源
- 获得原始指针：`get()`函数，注意此时内存并没有释放
- 用独立语句将新建对象置入智能指针中

~~~cpp
f(shared_ptr<A>(new A()), g());
// g在何时调用不确定。如果g调用出现异常，可能使A的指针丢失
~~~

<a name='20'></a>
### 20. 用常量引用传递代替值传递

- 避免对象的复制以及相应的拷贝析构操作
- 避免对象切割。指针可以实现派生类到基类的转换，值传递不行。

~~~cpp
void f(A a);
void f(const A& a);
~~~

<a name='21'></a>
### 21. 不要返回引用对象

返回局部变量：引用对象不存在
返回堆上对象：无法释放堆对象

<a name='23'></a>
### 23. 宁以非成员非友元代替成员函数

- 非成员非友元函数有更强的封装性：便利函数
- 将便利函数放在多个头文件但隶属于同一个命名空间。类成员函数做不到。

<a name='24'></a>
### 24. 若所有参数皆需类型转换，请采用非成员函数
令类支持类型转换通常是个糟糕的想法。
~~~cpp
class Rational{
public:
	Rational(int numerator=0, int demoninator=1);
	// 没有声明explicit，支持int->Rational隐式转换
	int numerator() const;
	int demoninator() const;
private:
	...
}

const Rational operator*(const Rational& lhs, const Rational& rhs){
	return Rational(...);
}	// 支持 2 * Rational(...)
~~~

<a name='25'></a>
### 25. 写不抛出异常的swap函数

默认的`swap`函数进行了3次复制，有时相当耗时。

#### 实现类的swap
~~~cpp
class A{
public:
	void swap(A& other){
		using std::swap;
		swap(p, other.p);
	}
private:
	Ap* p;
}

namespace std{
	// 采用非成员函数实现swap
	template<> void swap<A>(A& a, A& b){
		a.swap(b);
	}
}
~~~
上述例子，在类`A`中用`std::swap`实现指针的交换，在`std`空间实现`swap`的特例化。`template<>`表示对`std::swap`实现特例化，`swap<A>`表示特例化成`A`版本。

#### 实现模板类的swap

~~~cpp
namespace std{
	template<typename T> 
	void swap<A<T>>(A<T>& a, A<T>& b){// 错误：函数模板不支持偏特化
		a.swap(b);
	}
}

namespace std{
	template<typename T> 
	void swap(A<T>& a, A<T>& b){// 通常可以重载函数，但std禁止
		a.swap(b);
	}
}

namespace A{ // 解决：添加一个新名字空间
	template<typename T>
	class A{ };

	template<typename T>
	void swap(A<T>& a, A<T>& b){
		a.swap(b);
	}
}

// 调用时
using std::swap;	// 令std::swap可见
swap(...)		// 调用时不加std
~~~




