---
layout: post
title:  "effective c++ note"
date:   2017-09-17 14:00:00 +0800
toc: true
categories: [cpp]
tags: [note]
description: effective c++ note
---
## Table of Contents
- 让自己习惯C++
  - [02. 尽量以const、enum、inline、代替#define](#02)
  - [03. 尽可能使用const](#03)
  - [04. 确定对象使用前已被初始化](#04)
- 构造、析构、赋值运算符
  - [05. 了解c++默认调用哪些函数](#05)
  - [06. 禁止编译器生成的函数](#06)
  - [07. 虚析构函数](#07)
  - [09. 构造和析构函数不调用virtual成员函数](#09)
  - [11. 在operator=中处理自我赋值](#11)
- 资源管理
  - [13. 以对象管理资源](#13)
- 设计与声明
  - [20. 用常量引用传递代替值传递](#20)
  - [21. 不要返回引用对象](#21)
  - [23. 宁以非成员非友元代替成员函数](#23)
  - [24. 若所有参数皆需类型转换，请采用非成员函数](#24)
  - [25. 写不抛出异常的swap函数](#25)
- 实现
  - [27. 减少转型](#27)
  - [28. 避免返回handles指向对象内部成分](#28)
  - [29. 为异常安全而努力是值得的](#29)
  - [30. 透彻了解inline的里里外外](#30)
  - [31. 将文件间的编译依存关系降到最低](#31)
- 继承与面向对象设计
  - [33. 避免遮盖继承而来的名称](#33)
  - [34. 区分接口继承和实现继承](#34)
  - [35. 考虑virtual函数以外的其他选择](#35)
  - [37. 绝不重新定义继承而来的默认参数值](#37)
  - [39. 明智而审慎的使用private继承](#39)
  - [40. 明智而审慎的使用多重继承e](#40)
- 模板与泛型编程
  - [41. 了解隐式接口和编译期多态](#41)
  - [42. 了解typename的双重意义](#42)
  - [43. 学习处理模板化基类内的名称](#43)
  - [45. 运用成员函数模板接收所有兼容类型](#45)
  - [46. 需要类型转换时请为模板定义非成员函数](#46)
  - [47. 请使用traits class表现类型信息](#47)
- 定制new与delete
  - [49. 了解new-handle的行为](#49)
  - [50. 了解new和delete的合理替换时机](#50)
  - [51. 编写new和delete时需固守常规](#51)
  - [52. 写了定位new也要写定位delete](#52)

----

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
const char* const name = "Scott";   // 两个const
const string name("Scott");         // 建议用string

class A(){
private:
    static const int Num = 5;       // 常量声明式，类内设初值
    int arr[Num];
    static int a;                   // 静态变量，类外设初值
}
const int A::Num;                   // 常量定义式，由于在声明时有初值，此处不能再设值
int A::a = 10;
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
2. 成员变量初始化在构造函数之前，因此构造函数的最佳写法是用**初始化列表**；
如果成员变量是`const`或`reference`，则一定需要初值，不能被赋值；
成员变量以声明次序初始化，与初始化列表顺序无关。
3. 不同编译单元内的**非局部静态对象**初始化次序没有明确定义

解决方法：将非局部静态对象搬到函数内，函数返回指向这个对象的引用
~~~cpp
A& f(){
    static A a;
    return a;
}
~~~

<a name='05'></a>
### 05. 了解c++默认调用哪些函数
含有`reference`和`const`成员的类，编译器不会生成拷贝构造函数（原因：`reference`和`const`是不可改变的，因此禁止复制）。

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
    Bitmap *p = pb;             // 先创建副本，再处理
    pb = new Bitmap(*rhs.pb);
    delete p;
    return *this;
}

A& A::operator=(const A& rhs){
    A temp(rhs);
    swap(temp);                 // copy and swap 技术
    return *this;
}
~~~

<a name='13'></a>
### 13. 以对象管理资源

- 用`shared_ptr`管理资源，以便做到自动释放资源，即使中途会发生不可测的事件
- 用一个新的对象存储需要被自动释放的资源，依靠对象的析构函数释放资源。
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
}   // 支持 2 * Rational(...)
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

<a name='27'></a>
### 27. 减少转型

2种旧式转型与4种新式转型
~~~cpp
(T)expression
T(expression)
const_cast<T>(expression)	// 移除常量性
dynamic_cast<T>(expression)	// 执行运行期安全的向下转型，可能耗费巨大，类含有虚函数
reinterpret_cast<T>(expression)	// 执行低级转型，依赖于编译器，不可移植
static_cast<T>(expression)	// 强迫隐式转换
~~~

~~~cpp
class Derive: public Bass{
public:
	virtual void f(){
		Base::f();		      // 调用Base的f()
		static_cast<Base>(*this).f(); // 将此对象转型成Base类，然后调用f()
	}
}
~~~

<a name='28'></a>
### 28. 避免返回handles指向对象内部成分
`handles`：引用、指针、迭代器

如果`const`成员函数返回内部成员的handles，则函数的调用者依然可以修改数据，即使函数声明为`const`。可以将返回类型声明为`const`解决此问题，但依然可能导致悬空指针。

<a name='29'></a>
### 29. 为异常安全而努力是值得的

异常安全的函数：
- 不泄露资源：`shared_ptr`，`Lock`类
- 不破坏数据

copy-and-swap:
~~~cpp
struct NeedChange{		// 将需要修改的成员置于新的类中
	shared_ptr<...> ...;
}
void A::change(istream& src){
	using std::swap
	Lock ml(&mutex);	// 互斥锁
	shared_ptr<NeedChange> pNew(new NeedChange(*pSrc));	// 获得副本
	pNew->...;		// 修改副本
	swap(pSrc, pNew)	// 置换
}
~~~

<a name='30'></a>
### 30. 透彻了解inline的里里外外

inline：将对函数的每一个调用都替换成函数本体。不需要承受函数调用导致的额外开销，却可能导致代码膨胀。

定义：
- 类内定义
- 类外加上关键词`inline`

`template`和`inline`通常置于**头文件**内，因为编译器需要在编译时知道函数长什么样。

编译器拒绝对复杂代码进行`inline`，如循环、递归、虚函数。编译器通常不对**通过函数指针调用**的进行`inline`。

`inline`无法随着程序库的升级而升级，改变`f()`将需要对函数重新编译，而动态链接则简单得多。

大多数调试器无法调试`inline`，因为无法设置断点。

<a name='31'></a>
### 31. 将文件间的编译依存关系降到最低

~~~cpp
class B;
class A{
private:
	B b;	// 与类B产生关系，编译时需要B的定义式
}		// 当B变化时，A需要重新编译

// method 1
class AImpl;
class A{
private:
	shared_ptr<AImpl> pImpl; // 编译时不需要B的定义式，实现接口分离
}

// method 2
// 将A写成虚基类
~~~

<a name='33'></a>
### 33. 避免遮盖继承而来的名称

`public`继承会继承基类的所有成员，覆盖基类的成员，`private`不同。

~~~cpp
class B{
public:
	void f(){..}
	void f(int x){..}
}

class D: private B{
public:
	void f(){..}		// 覆盖基类所有f
	void f(){ 
		using B::f();	// 使B的函数可见，相当于只覆盖一部分
		f();
	}
}
~~~

<a name='34'></a>
### 34. 区分接口继承和实现继承
#### #声明一个纯虚函数是为了让派生类继承其函数接口
告诉派生类必须继承函数`f()`，但不干涉它怎么实现
#### #声明一个非纯虚函数是为了让派生类继承其函数接口和默认实现
告诉派生类必须继承函数`f()`，如不继承，就用基类的默认实现

有些人认为接口和默认实现应该分离
~~~cpp
class B{
public:
	virtual void f(...) = 0
}
void B::f() {...}		// 提供基类的默认实现

class D: public B{
public:
	void f(...) { B::f();}	// 使用基类的默认实现
	void f(...) { ...}	// 自己实现
}
~~~
#### #声明一个非虚函数是为了让派生类继承其函数接口和一份强制实现

<a name='35'></a>
### 35. 考虑virtual函数以外的其他选择
#### #由非虚接口实现模板方法模式
> A的派生类可以重新定义`g()`，但不能调用它；A确可以调用

允许派生类重新定义虚函数，赋予它们“如何实现”的能力，而基类控制函数“何时调用”。
~~~cpp
class A{
public:
	void f() const{
		g();		// 可以做些其他事
	}
private:
	virtual void g() const {...}
}
~~~
#### #由函数指针实现策略模式
- 同一类的不同实体可以有不同的函数
- 某个实体的函数可以在运行期变更
- 纯粹依赖于类的`public`接口的信息，如果需要`private`信息就会有问题

~~~cpp
class A;
void f_default(const A&);

class A{
public:
	typedef void (*fp)(const A&);
	explicit A(...);
	void f() const { return p(*this);}
private:
	fp = p
}
~~~
#### #由function实现策略模式
~~~cpp
class A;
void f_default(const A&);

class A{
public:
	typedef function<void (const A&)> fp;
	explicit A(...);
	void f() const { return p(*this);}
private:
	fp = p
}
~~~
#### #古典策略模式
将策略函数实现为一个派生类

<a name='37'></a>
### 37. 绝不重新定义继承而来的默认参数值
虚函数动态绑定，默认参数值**静态绑定**。如果默认参数值也动态绑定，则编译器的效率会大幅降低。

当你想要实现默认参数的机制时，所有类需要有相同的默认参数，显得冗余又相互依赖。解决办法是使用之前的设计模式，如非虚接口
~~~cpp
class A{
public:
	void f(int x=1) const{	// 不被继承，使得默认参数始终为1
		g(x);
	}
private:
	virtual void g(x) const = 0;
}

class B: public A{
private:
	virtual void g(x) const;
}
~~~

<a name='39'></a>
### 39. 明智而审慎的使用private继承

`private`继承：
- 派生类对象不会转化成基类对象
- 基类中的成员在派生类中都是`private`

`private`继承与复合都能实现“根据某物实现出”，尽量使用复合，除非牵扯到`protected`成员和虚函数。

~~~cpp
class Timer{
public:
	virtual void onTick() const;
}

// Widget想利用Timer的计时功能，但本身不是个计时器，因此要private继承
class Widget: private Timer{
private:
	virtual void onTick() const;
}

// 复合模型
class Widget{
private:
	class WidgetTimer: public Timer{
	public:
		virtual void onTick() const;
	};
	WidgetTimer timer;
}
~~~

两者不同点：
- private继承中，`Widget`的派生类不能调用虚函数，却能**定义**虚函数（见35）；在复合模型中，`WidgetTimer`是私有成员，`Widget`的派生类无法调用，也就不能定义虚函数。
- private继承中，`Widget`依赖于`Timer`，需要`Timer`的定义式；在复合模型中，`WidgetTimer`可单独存放，在`Widget`中保存指向`WidgetTimer`的指针，使**编译依赖性**最小化。
- 当派生类想要访问基类的`protected`成员，或重新定义虚函数时，使用private继承。
- 当类A不带任何数据（非静态成员、虚函数、虚基类）时，继承类A的对象不占空间（空白积累最优化），包含类A的对象占1字节。类A可以含有`typedef`、`enum`、`static`成员变量。

<a name='40'></a>
### 40. 明智而审慎的使用多重继承

当存在**钻石型多重继承**时，应该声明**虚继承**。

由于虚继承比普通继承更耗费资源，因此有两条建议：
- 必要时使用虚继承
- 虚基类中尽可能避免放置数据

> 一个正当用途：public继承接口，private继承某个协助实现的类

<a name='41'></a>
### 41. 了解隐式接口和编译期多态

- 类：显示接口、运行期多态
- 模板：隐式接口、编译期多态
- 显示接口：由函数签名式（函数名称、参数类型、返回类型）构成
- 隐式接口：由有效表达式构成

<a name='42'></a>
### 42. 了解typename的双重意义
1. 声明模板参数时，`class`和`typename`可以互换
2. 模板内出现的名称如果相依于某个模板参数，称之为**从属名称**。如果从属名称在**类**内呈嵌套状，称为**嵌套从属名称**。c++解析器在模板中遭遇嵌套从属名称，它便假设这个名称不是个类型，除非加上关键字`typename`。

~~~cpp
template<typename T> f(){
	T::const_iteration iter(...);		// wrong
	typename T::const_iteration iter(...);	// right
}
~~~

> 例外：`typename`不能出现在**基类列表**的嵌套从属名称之前，也不能在**成员初始化列表**中作为基类的修饰符。

~~~cpp
template<typename T>
class D: public B<T>::Nested{			// 基类列表，不加
public:
	explicit D(int x): Base<T>::Nested(x){	// 成员初始化列表，加
		typename Base<T>::Nested temp;	// 加
	}
}
~~~

另一个常见例子
~~~cpp
template<typename T>
void f(T iter){
	typename std::iterator_traits<T>::value_type temp(*iter);
	// 换一种写法
	typedef typename std::iterator_traits<T>::value_type value_type;
	value_type temp(*iter);
}
~~~

<a name='43'></a>
### 43. 学习处理模板化基类内的名称

~~~cpp
template <typename T>
class Base{
public:
	void f() {...;}
}

template <typename T>
class Derive: public Base<T>{
public:
	void g(){
		f();			// error
		this->f();		// method 1
		using Base<T>::f();	// method 2
		f();
		Base<T>::f();		// method 3，虚函数会因此解绑定
	}
}
~~~

<a name='45'></a>
### 45. 运用成员函数模板接收所有兼容类型
#### 构造模板
带有基类-派生类关系的两类分别具体化某个`template`，产生出来的两个具现体不带有基类-派生类关系。模板可能被无限量的具体化，可能有无限量的构造函数，因此需要写一个**构造模板**。

~~~cpp
template<typename T>
class SmartPtr{
public:
	// 没有声明explicit，支持指针的隐式转换
	template<typename U> SmartPtr(const SmartPtr<U>& other)
	: heldptr(other.get()) {...}	// 支持U*到T*的转换
	T* get() const {return heldptr;}
private:
	T* heldptr;
}
~~~

#### 支持赋值操作
当声明泛化拷贝构造函数时，也要声明普通的拷贝构造函数。

~~~cpp
template<class T>
class shared_ptr{
public:
	template<class Y> explicit shared_ptr(Y* p);
	shared_ptr(shared_ptr const& r);
	template<class Y> shared_ptr(shared_ptr<Y> const& r);
	template<class Y> explicit shared_ptr(weak_ptr<Y> const& r);
	template<class Y> explicit shared_ptr(auto_ptr<Y>& r);
	shared_ptr& operator=(shared_ptr const& r);
	template<class Y> shared_ptr& operator=(shared_ptr<Y> const& r);
	template<class Y> shared_ptr& operator=(auto_ptr<Y>& r);
}
~~~

<a name='46'></a>
### 46. 需要类型转换时请为模板定义非成员函数
参考[24](#24)

> **函数模板**在实参推导时从不将**隐式类型转换**函数考虑在内。**类模板**不存在这种情况。

~~~cpp
template<typename T>
class Rational{
public:
	Rational(const T& numerator=0, const T& demoninator=1);
	const T numerator() const;
	const T demoninator() const;
	// 声明时Rational<T> 可以被替换成Rational
	friend const Rational operator*(const Rational& lhs, const Rational& rhs){
		return Rational(...);
	}// 这个是函数，不是函数模板
}
~~~

解释：为了让所有可能的类型转换发生于实参上，需要非成员函数；为了让函数被具现化，将它声明在类中；在类内声明函数，让其成为friend。内联函数可能会带来巨大开销，可以考虑令友元函数调用辅助函数。

<a name='47'></a>
### 47. 请使用traits class表现类型信息

- `traits`类使得类型相关信息在编译期可用
- 重载技术使得`traits`类可以在编译期对类型执行if-else检测

例子：对迭代器实现`advance`操作
~~~cpp
template<...>
class deque{
public:
	class iterator{
	public:	
		typedef random_access_iterator_tag iterator_category;
	};
}

template<typename IterT>
struct iterator_traits{
	typedef typename IterT::iterator_category iterator_category;
}

template<typename IterT>
struct iterator_traits<IterT*>{
	typedef random_access_iterator_tag iterator_category;
}

// 定义各种类型的重载版本
template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::random_access_iterator_tag){
	iter += d;
}

// 调用重载函数
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d){
	doAdvence(iter, d, typename std::iterator_traits<IterT>::iterator_cateroty());
}
~~~

<a name='49'></a>
### 49. 了解new-handler的行为

当`operator new`抛出异常之前，它会先调用客户指定的错误处理函数，称为new-handler。客户通过set_new_handler指定new-handler。
~~~cpp
typedef void (*new_handler)();				// new_handle为函数指针
new_handler set_new_handler(set_new_handler p) throw();	// 返回当前的new_handler，参数为新的new_handler
~~~

当`operator new`无法申请足够的内存时，它会不断调用new-handler函数。因此，一个设计良好的new-handler必须有一下特征：
- 让更多内存可被使用
- 调用新的new-handler
- 抛出bad_alloc

有时想为每个类设计专属的new-handler，只需令每个类声明自己的`set_new_handler`和`operator new`。自己声明的函数会调用系统的函数，要保证调用完后恢复系统函数的状态，因此可以利用资源处理类[(13)](#13)。
~~~cpp
// handler资源管理类
class NewHandlerHolder{
public:
	explicit NewHandlerHolder(std::new_handler nh) {...}
	~NewHandlerHolder() { std::set_new_handler(handler); }
}

class Widget{
public:
	static std::new_handler set_new_handler(std::new_handler p) throw();
	static void* operator new(std::size_t size) throw(std::bad_alloc);
private:
	static std::new_handler currentHandler;
}

// 定义
std::new_handler Widget::currentHandler = 0;
std::new_handler Widget::set_new_handler(std::new_handler p) throw(){
	std::new_handler oldHandler = currentHandler;
	currentHandler = p;
	return oldHandler;
}
void* Widget::operator new(std::size_t size) throw(std::bad_alloc){
	// std、widget保存currentHandler， h保存之前的handler，
	// h析构时将std设为之前的handler
	NewHandlerHolder h(std::set_new_handler(currentHandler));
	return ::operator new(size);
}
~~~
可以将类Widget重新定义成基类模板，具体的类只需继承自这个模板即可。

<a name='50'></a>
### 50. 了解new和delete的合理替换时机
定制自己的`operator new`的理由：
- 检测内存运用上的错误：可以超额分配内存，以额外空间放置位签名。`delete`时依据签名来判断内存是否出错，并找出错误的指针
- 收集内存使用上的数据
- 增加分配和归还的速度
- 将相关对象成簇集中，减少内存页错误

~~~cpp
static const int signature = 0xDEADBEEF;
typedef unsigned char Byte;

void* operator new(std::size_t size) throw(std::bad_alloc){
	using namespace std;
	size_t realsize = size + 2 * sizeof(int);
	void* pMem = malloc(realsize);
	if(!pMem) throw bad_alloc();
	
	*(static_cast<int*>(*pMem)) = signature;
	*(reinterpret_cast<int*>(static_cast<Byte*>(*pMem)+realsize-sizeof(int))) = signature;
	
	return static_cast<Byte*>(*pMem) + sizeof(int);
}
~~~
<a name='51'></a>
### 51. 编写new和delete时需固守常规

~~~cpp
//operator new 的继承版本
void* Base::operator new(std::size_t size) throw(std::bad_alloc){
	if(size != sizeof(Base))
		return ::operator new(size);	// 派生类调用系统版本
}
~~~

对于继承的`operator new[]`，我们能做的只是分配一块未加工的内存。

对于`operator delete`，只需记得分配失败时调用了`::operator new`，那么删除失败时也要调用`::operator delete`。

<a name='52'></a>
### 52. 写了定位new也要写定位delete
通常情况下，new会调用`operator new`操作，然后调用类的构造函数。如果构造函数出错，就会导致系统泄露，此时运行期系统会调用相应的`operator delete`，即参数类型和个数与之相对应的`operator delete`。若没有发生异常，系统调用标准delete。因此，对于定位new，要编写对应的定位delete和标准delete。

在类内编写`operator new`会遮盖所有全局的`operator new`函数。解决办法是将标准的`operator new`放入自定义的基类中，客户可以继承该基类。
~~~cpp
class StandardNewDeleteForms{
public:
	// 标准版本
	static void* operator new(std::size_t size) throw(std::bad_alloc)
		{ return ::operator new(size); }
	static void operator delete(void* pMem) throw()
		{ ::operator delete(pMem); }
	// 定位版本
	static void* operator new(std::size_t size, void* ptr) throw()
		{ return ::operator new(size, ptr); }
	static void operator delete(void* pMem, void* ptr) throw()
		{ ::operator delete(pMem, ptr); }
	// nothrow版本
	static void* operator new(std::size_t size, const std::nothrow_t& nt) throw()
		{ return ::operator new(size, nt); }
	static void operator delete(void* pMem, const std::nothrow_t& nt) throw()
		{ ::operator delete(pMem); }
}

class Widget: public StandardNewDeleteForms{
public:
	using StandardNewDeleteForms::operator new;
	using StandardNewDeleteForms::operator delete;

	static void* operator new(std::size_t size, std::ostream& logStream) throw(std::bad_alloc);
	static void operator delete(void* pMem, std::ostream& logStream) throw();
}
~~~



























