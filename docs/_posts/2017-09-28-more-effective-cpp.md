---
layout: post
title:  "more effective c++ note"
date:   2017-09-28 14:00:00 +0800
toc: true
categories: [cpp]
tags: [note]
description: more effective c++ note
---

## Table of Contents
- 第一章 基础议题
  - [01. 指针和引用的区别](#01)
  - [03. 不要对数组使用多态](#03)
  - [04. 避免无用的缺省构造函数](#04)
- 第二章 运算符
  - [05. 谨慎定义类型转换函数](#05)
  - [08. 理解不同含义的new和delete](#08)
- 第三章 异常
  - [09. 使用析构函数防止资源泄漏](#09)
  - [10. 在构造函数中防止资源泄漏](#10)
  - [11. 禁止异常信息传递到析构函数外](#11)
  - [12. 理解“抛出一个异常”与“传递一个参数”间的差异](#12)
  - [14. 审慎使用异常规格](#14)
- 第四章 效率
  - [19. 理解临时对象的来源](#19)
  - [24. 理解虚拟函数、多继承、虚基类和 RTTI 所需的代价](#24)
- 第五章 技巧
  - [25. 将构造函数和非成员函数虚拟化](#25)
  - [27. 要求或禁止在堆中产生对象](#27)
  - [30. 代理类](#30)
  - [31. 让函数根据一个以上的对象来决定怎么虚拟](#31)

- 第六章 杂项



---

<a name='01'></a>
### 01. 指针和引用的区别

- 引用不能指向空值，所以引用必须初始化；指针不初始化虽然危险，但合法。
- 引用效率高，使用前不需要检测合法性。
- 引用总是指向初始化对象，而指针可以改变。
- 重载时返回原对象的引用，这样可以直接修改值。如果用指针，修改值前还需解引用。

~~~cpp
int a = 10;
int b = 20;
int& r = 1;
int* p = &a;
r = b;	// r是a的引用，a的值变为20
p = &b;	// p指向b

v[2] = 10;	// 重载[]返回引用
*v[2] = 10;	// 重载[]返回指针
~~~

<a name='03'></a>
### 03. 不要对数组使用多态
~~~cpp
class BST{...}
class BalancedBST{...}
void printBSTArray(ostream& s, const BST array[], int num) { ... }
~~~

数组通过指向数组起始位置的指针来操纵数组，无法确定派生类与基类的大小。语言规范中说通过一个基类指针来删除一个含有派生类对象的数组，结果将是不确定的。

> 解决：不要从一个具体类派生出另一个具体类

<a name='04'></a>
### 04. 避免无用的缺省构造函数

没有缺省构造函数会遇到的问题：
- 没有一种办法能在建立**对象数组**时给构造函数传递参数。
  - 建立对象的指针数组，对每个指针调用构造函数
  - `operator new`开辟原始内存空间，然后调用构造函数
- 许多基于模板的容器类（由于其设计不当）需要缺省构造函数。

<a name='05'></a>
### 05. 谨慎定义类型转换函数
可能导致编译器进行隐式类型转换的函数：
- 单参数构造函数，或多个参数，但剩余参数均有缺省值
- 隐式类型转换运算符

~~~cpp
class Rational{
public:
	Rational(int numerator = 0, int denominator = 1);
	operator double() const{
		return (double)numerator/denominator;
	}
	// 解决办法
	explicit Rational(int numerator = 0, int denominator = 1);
	double asDouble() const{
		return (double)numerator/denominator;
	}
}
~~~

<a name='08'></a>
### 08. 理解不同含义的new和delete

不同的`new`操作：
- 在堆上建立一个对象，应该用`new`操作符
- 仅仅想分配内存，应该调用`operator new`函数，它不会调用构造函数
- 想控制堆对象被建立时的内存分配过程，你应该写你自己的`operator new`函数，然后使用`new`操作符，`new`操作符会调用你定制的`operator new`
- 在一块已经获得指针的内存里建立一个对象，应该用`placement new`，指针作为另一个参数

<a name='09'></a>
### 09. 使用析构函数防止资源泄漏
异常有时会终止程序，导致内存泄露：
- 用一个对象存储需要被自动释放的资源，依靠对象的析构函数释放资源。
- shared_ptr

<a name='10'></a>
### 10. 在构造函数中防止资源泄漏

构造函数抛出异常，会使已构造部分（指针、引用）无法释放。可以在构造函数内捕获所有异常，释放资源，并继续传递异常。
~~~cpp
class BookEntry {
public:
	BookEntry(const string&, const string&);
	~BookEntry();
private:
	Image *theImage;
	AudioClip *theAudioClip;
	void cleanup(); 
};

void BookEntry::cleanup(){	// 释放资源函数
	delete theImage;
	delete theAudioClip;
}

BookEntry::BookEntry(const string& imageFileName, const string& audioClipFileName)
:theImage(0), theAudioClip(0){
	try {
		if (imageFileName != "") {
			theImage = new Image(imageFileName);
		}
		if (audioClipFileName != "") {
			theAudioClip = new AudioClip(audioClipFileName);
		}
	}
	catch (...) { 	// 构造函数中释放资源
		cleanup();
		throw;
	}
}
BookEntry::~BookEntry(){// 析构函数中释放资源
	cleanup();
}
~~~
如果成员是常量指针，就需要在初始化列表中赋予初值。解决办法是用`shared_ptr`封装原指针。这样析构时也不需要做任何事。

~~~cpp
class BookEntry {
private:
	const shared_ptr<Image> theImage;
	const shared_ptr<AudioClip> theAudioClip;
};

BookEntry::BookEntry(const string& imageFileName, const string& audioClipFileName)
:theImage(make_shared<Image>(imageFileName)), 
theAudioClip(make_shared<AudioClip>(audioClipFileName)){ }

BookEntry::~BookEntry() { }
~~~

<a name='11'></a>
### 11. 禁止异常信息传递到析构函数外
两种情况下会调用析构函数：
- 正常情况下删除一个对象
- 异常传递的堆栈辗转开解（stack-unwinding）过程中，由异常处理系统删除一个对象。如果在此过程中发生异常，程序直接终止

坏处：
- 如果导致程序直接终止，局部变量不会析构
- 析构函数可能不完全运行

<a name='12'></a>
### 12. 理解“抛出一个异常”与“传递一个参数”间的差异

- 捕获异常时，都会对异常进行拷贝。如果是传值捕获异常，还会再拷贝一次。
- 异常支持的参数类型转换少，包括基类与派生类、类型化指针到无类型指针
- 异常参数类型匹配是最近匹配，函数是最优匹配


~~~cpp
catch (Widget& w){
	...
	throw;		// 重抛出，不会拷贝
}

catch (Widget& w){
	...
	throw w;	// 抛出，会拷贝
}
~~~

通过指针捕获异常，则异常对象无法释放。通过值捕获异常，则异常对象存在派生类到基类的截断。因此最好通过引用捕获异常。

<a name='14'></a>
### 14. 审慎使用异常规格

如果一个函数抛出一个不在异常规格范围里的异常，系统在**运行时**能够检测出错误，函数`unexpected`将被调用，直接终止。编译器在编译时只能部分检测。如：一个函数调用另一个函数，并且后者可能抛出违反前者规格的异常，编译器不会检测。

- 避免在带有类型参数的模板内使用异常规格
- 如果在一个函数内调用其它没有异常规格的函数时应该去除这个函数的异常规格
- 处理系统本身抛出的异常，可以用其他异常代替。

~~~cpp
class UnexpectedException {};

void convertUnexpected() { 
	throw UnexpectedException();
}

set_unexpected(convertUnexpected);	// 用convertUnexpected替换缺省的unexpected函数
~~~

<a name='19'></a>
### 19. 理解临时对象的来源

来源：建立一个没有命名的非堆对象，包括为了使函数成功调用而进行**隐式类型转换**和**函数返回对象**时。

当通过**传值**方式传递对象或传递**常量引用**参数时，会产生临时对象。C++禁止为非常量引用产生临时对象。
~~~cpp
const string &temp = "c++";	// right
string &temp = "c++";		// wrong
~~~

可以通过重载避免隐式类型转换；也可以对返回值进行优化，**返回构造函数**而不是直接返回对象，因为C++允许编译器优化**不出现的**临时变量。

<a name='24'></a>
### 24. 理解虚拟函数、多继承、虚基类和 RTTI 所需的代价
#### 代价一：虚函数表
虚函数表vtbl：通常是一个函数指针数组，里面是指向虚函数实现体的指针。每个类有对应的一个表。

通常一个类只需要一个vtbl，如果有多个obj文件，vtbl应该位于哪个obj文件中：
- 每一个可能需要vtbl的obj文件生成一个vtbl拷贝，连接程序然后去除重复的拷贝，在最后的可执行文件或程序库里就为每个vtbl保留一个实例。
- 启发式算法：要在一个obj文件中生成一个类的vtbl，要求该obj文件包含该类的第一个**非内联、非纯虚拟函数**定义（也就是类的实现体）。因此，要避免把虚函数声明为内联函数。

#### 代价二：虚表指针
虚表指针vptr：类的数据成员，指向对应类的vtbl。每一个具体对象有一个指针。

通过指针调用虚函数时，编译器的工作：
1. 通过对象的vptr找到类的vtbl
2. 找到对应vtbl内的指向被调用函数的指针
3. 调用第二步找到的的指针所指向的函数

#### 代价三：无法内联
内联函数是在编译期用被调用函数体代替函数调用指令，显然，虚函数不能内联。

#### RTTI
RTTI的**类型信息**是直接在vtbl的基础上做的，可以看成是vtbl的一个值。

<a name='25'></a>
### 25. 将构造函数和非成员函数虚拟化

虚拟构造函数：因为它能建立新对象，行为与构造函数相似，而且因为它能建立不同类型的对象，我们称它为虚拟构造函数。类似的有虚拷贝函数。

~~~cpp
class Shape {
public:
	virtual Shape* clone()  const = 0;
	virtual Shape* create() const = 0;
};

class Circle : public Shape {
public:
	Circle* clone()  const { return new Circle(*this); }
	Circle* create() const { return new Circle(); }
};

class Square : public Shape {
public:
	Square* clone() const { return new Square(*this);}
	Square* create() const { return new Square();}
};

class A {
private:
	list<Shape*> components;
	static Shape* readShape(istream& str);
};
A::A(istream& str) {
	while (str) {
		components.push_back(readShape(str));
	}
}
~~~

<a name='26'></a>
### 26. 限制某个类所能产生的对象数量

单例模式：
~~~cpp
class Printer {
public:
	static Printer& thePrinter();
private:
	Printer();
	Printer(const Printer& rhs);
};

Printer& Printer::thePrinter() {
	static Printer p;
	return p;
}
~~~
四个注意点：
- `Printer`对象是位于函数里的静态成员而不是在类中的静态成员。在类中的静态对象实际上总是被构造（和释放），即使不使用该对象。与此相反，只有第一次执行函数时，才会建立函数中的静态对象。
- `thePrinter()`不能是内联函数。内联函数可能在程序内被复制，这种复制也包括函数内的静态对象。
- `Printer`构造函数是`private`，意味着它没有派生类。
- 不适合情况：建立p1，释放p1，建立p2，释放p2

改进：建立任意数量的对象
~~~cpp
class Printer {
public:
	class TooManyObjects{};		// 对象过多，抛出异常
	static Printer * makePrinter();	// 伪构造函数
	static Printer * makePrinter(const Printer& rhs);
	~Printer();
	void f();
private:
	static size_t numObjects;
	static const size_t maxObjects = 10;
	Printer();
	Printer(const Printer& rhs);
};

size_t Printer::numObjects = 0;
Printer::Printer(){
	if (numObjects >= maxObjects) { throw TooManyObjects();}
	++numObjects;
}
Printer * Printer::makePrinter() { return new Printer; }
~~~

一个具有对象计数功能的基类：支持任意数量的对象计数
~~~cpp
template<class BeingCounted>
class Counted {
public:
    class TooManyObjects{};     // 用来抛出异常
    static int objectCount() { return numObjects; }
protected:
    Counted();
    Counted(const Counted& rhs);
    ~Counted() { --numObjects; }
private:
    static int numObjects;
    static const size_t maxObjects;
    void init();        // 避免构造函数的
};

template<class BeingCounted>
Counted<BeingCounted>::Counted() { init(); }
template<class BeingCounted>
Counted<BeingCounted>::Counted(const Counted<BeingCounted>&) { init(); }
template<class BeingCounted>
void Counted<BeingCounted>::init() {
    if (numObjects >= maxObjects) throw TooManyObjects();
    ++numObjects;
}

class Printer: private Counted<Printer> {	// private继承，隐藏Counted的实现细节
public:
    static Printer * makePrinter();
    static Printer * makePrinter(const Printer& rhs);
    ~Printer();
    using Counted<Printer>::objectCount;    	// 原本函数是private
    using Counted<Printer>::TooManyObjects;
private:
    Printer();
    Printer(const Printer& rhs);
};
~~~

<a name='27'></a>
### 27. 要求或禁止在堆中产生对象
#### #要求在堆中建立对象
**非堆对象**在定义它的地方被自动构造，在生存时间结束时自动被释放，所以只要禁止使用隐式的构造函数和析构函数，就可以实现这种限制。

让析构函数成为`private`，让构造函数成为`public`。
~~~cpp
class A {
public:
	A();
	void destroy() const { delete this; }
private:
	~A();
};

A* p = new A;
p->destroy();
~~~

这种方法有个缺点，它也禁止了继承和包容。解决办法是将析构函数声明为`protected`（同时它的构
造函数还保持`public`）。

#### #禁止堆对象

~~~cpp
class A {
private:
	static void *operator new(size_t size);
	static void operator delete(void *ptr);
};
~~~
派生类如果没有重新定义`operator new`，则无法新建对象。

<a name='30'></a>
### 30. 代理类

用代理类分开`string`的`[]`操作的左值、右值版本。
~~~cpp
class String {
public:
	class CharProxy {
	public:
		CharProxy(String& str, int index);
		CharProxy& operator=(const CharProxy& rhs); 	// 左值
		CharProxy& operator=(char c); 			// 左值
		operator char() const; 				// 右值
	private:
		String& theString;
		int charIndex;
	};
	const CharProxy operator[](int index) const{
		return CharProxy(const_cast<String&>(*this), index);
	}
	CharProxy operator[](int index){
		return CharProxy(*this, index);	
	}
	friend class CharProxy;
private:
	RCPtr<StringValue> value;
};
~~~

<a name='31'></a>
### 31. 让函数根据一个以上的对象来决定怎么虚拟
#### #用查找表实现二重调度
~~~cpp
class SpaceShip: public GameObject {
public:
	virtual void collide(GameObject& otherObject);
	virtual void hitSpaceShip(GameObject& otherObject);
	virtual void hitSpaceStation(GameObject& otherObject);
	virtual void hitAsteroid(GameObject& otherobject);
private:
	typedef void (SpaceShip::*HitFunctionPtr)(GameObject&);
	static HitFunctionPtr lookup(const GameObject& whatWeHit);
	typedef map<string, HitFunctionPtr> HitMap;
	static HitMap* initializeCollisionMap();
};

void SpaceShip::collide(GameObject& otherObject){ 
	HitFunctionPtr hfp = lookup(otherObject);
	if (hfp) {
		(this->*hfp)(otherObject);
	}
	else {
		throw CollisionWithUnknownObject(otherObject);
	}
}

SpaceShip::HitFunctionPtr SpaceShip::lookup(const GameObject& whatWeHit) {	// 静态查找表
	static shared_ptr<HitMap> collisionMap(initializeCollisionMap());	// 查找表初始化一次
	HitMap::iterator mapEntry = collisionMap.find(typeid(whatWeHit).name());
	if (mapEntry == collisionMap.end())
		return 0;
	return (*mapEntry).second;

}

SpaceShip::HitMap * SpaceShip::initializeCollisionMap() {
	HitMap *phm = new HitMap;
	(*phm)["SpaceShip"] = &hitSpaceShip;
	(*phm)["SpaceStation"] = &hitSpaceStation;
	(*phm)["Asteroid"] = &hitAsteroid;
	return phm;
}

void SpaceShip::hitSpaceShip(GameObject& spaceShip) {
	SpaceShip& otherShip = dynamic_cast<SpaceShip&>(spaceShip);
	// process ...
}
~~~

#### #使用非成员函数处理碰撞函数
使用非成员函数，增加新的类时，原来的类不需要重新编译。

~~~cpp
#include "SpaceShip.h"
#include "SpaceStation.h"
#include "Asteroid.h"
namespace { 	// 无名命名空间，类似于文件范围的static变量
	// 所有的碰撞情况
	void shipAsteroid(GameObject& spaceShip, GameObject& asteroid);
	void shipStation(GameObject& spaceShip, GameObject& spaceStation);
	void asteroidStation(GameObject& asteroid, GameObject& spaceStation);
	....

	typedef void (*HitFunctionPtr)(GameObject&, GameObject&);
	typedef map< pair<string,string>, HitFunctionPtr > HitMap;
	pair<string,string> makeStringPair(const char *s1, const char *s2);
	HitMap * initializeCollisionMap();
	HitFunctionPtr lookup(const string& class1, const string& class2);
}

void processCollision(GameObject& object1, GameObject& object2) {
	HitFunctionPtr phf = lookup(typeid(object1).name(), typeid(object2).name());
	if (phf) 
		phf(object1, object2);
	else 
		throw UnknownCollision(object1, object2);
}
~~~

虽然如此设计，但仍不支持继承，有新的派生类时，需向查找表增加项。






















