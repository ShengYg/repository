---
layout: post
title:  "effective modern c++"
date:   2017-10-16 14:00:00 +0800
toc: true
categories: [cpp]
tags: [note]
description: effective modern c++
---

## Table of Contents
- 第1章 推断类型
  - [01. 理解模板类型推断](#01)
  - [02. 理解auto类型推断](#02)
  - [03. 理解decltype](#03)
  - [04. 懂得如何查看已推断类型](#04)
- 第6章 lambda函数
  - [31. 避免使用默认捕获模式](#31)
  - [32. 使用初始化捕获来把对象移动到闭包](#32)

---

<a name='01'></a>
### 01. 理解模板类型推断
C++11的auto自动推断变量的方式是以**模板推断**为基础的，模板推断的规则也应用在auto上，所以理解掌握模板推断的规则对于我们C++程序员来说很重要。
~~~cpp
template <typename T> 
void f(ParamType param);

f(expr);
~~~
编译器需要根据expr的来推断两个类型，一个是T， 一个是ParamType，这两个类型通常是不一样的。类型T的推导不仅仅依赖于expr，也依赖于ParamType，具体有以下3种情况：
- ParamType是指针类型或者引用类型，但不是[通用引用](#24)(universal references)类型
- ParamType是通用引用(universal references)类型
- ParamType既不是指针类型也不是引用类型

#### #情况1 ParamType是指针类型或者引用类型，但不是通用引用类型
- 如果传进来的参数expr是一个引用类型，忽视引用的部分
- 通过模式匹配expr的类型来决定ParamType的类型从而决定T的类型

~~~cpp
template <typename T>
void f(T& param); 

int x = 27;
const int cx = x;
const int &rx = x;

f(x);	// T 的类型为int, ParamType的类型为int&
f(cx);	// T 的类型为const int, ParamType的类型为const int&
f(rx);	// T 的类型为const int, ParamType的类型为const int&
~~~

#### #情况2 ParamType是通用引用类型
具体情况见[条款24](#24)，这里记住两条准则：
- 如果expr是一个左值，那么T和ParamType会被推断为左值引用
- 如果expr是一个右值，那么会用正常的推断方式(情况1)

~~~cpp
template <typename T>
void f(T&& param);

int x = 27;
const int cx = x;
const int &rx = x;

f(x);	// x是左值，所以T 和ParamType会被推断为int &类型
f(cx);	// cx是左值，所以T和ParamType会被推断为const int &类型
f(rx);	// rx是左值，所以T和 ParamType会被推断为const int &类型
f(27);	// 27是右值，根据情况1，T的类型会被推断为int、ParamType会被推断为int &&
~~~

#### #情况3 ParamType既不是指针类型也不是引用类型
- 如果expr的类型是引用类型，那么忽略引用
- 如果expr的类型是const的，把const也忽略了，还会忽略volatile

忽略const，是因为传进的参数expr尽管不可以改变值，但并不意味着它们的拷贝不可以。

~~~cpp
template <typename T>
void f(T param);

int x = 27;
const int cx = x;
const int &rx = x;

f(x);	// 易知T和ParamType的类型都是int
f(cx);	// 忽略const，T和ParamType的类型都是int
f(rx);	// 忽略了引用后再忽略const,T和ParamType的类型都是int
~~~

#### #情况4 数组作为参数
分为两种情况：
~~~cpp
const char name[] = "ABCDEFG";

template <typename T>
void f(T param);
f(name);	// T -> const char*

template <typename T>
void f(T& param);
f(name);	// T -> const char[8]
~~~

利用这个特点可以写一个模板，用来返回数组的长度
~~~cpp
template <typename T, std::size_t N>
constexpr std::size_t arraySize(T (&)[N]) noexcept {
	return N;
}
~~~
将变量声明为`constexpr`，使编译器来确定变量值是否为常量表达式。

#### #情况5 函数作为参数
与数组类似
~~~cpp
void someFunc(int, double);

template <typename T>
void f1(T param)；
template <typename T>
void f2(T& param)；

f1(someFunc);	// 传值，ParamType 类型为void (*)(int, double)
f2(someFunc);	// 引用语义，ParamType类型为void (&)(int, double)
~~~

<a name='02'></a>
### 02. 理解auto类型推断
> auto类型推断和模板类型推断一致，除了一个例外：`{}`

当我们想声明定义一个int类型时，有4种方式：
~~~cpp
int x1 = 27;
int x2(27);
int x3 = { 27 };
int x4{ 27 };
auto x3 = { 27 };	// x3为std::initializer_list<int>，而非int
auto x4{ 27 };
~~~

在模板推断中
~~~cpp
template <typename T> 
void f(T param);
f({ 11, 23, 9 });	// error

template <typename T> 
void f(std::initializer_list<T> initList);
f({ 11, 23, 9 });	// T -> int
~~~

在C++14中，可以把**函数的返回值**类型或者**lambdad中的参数**用auto声明，但是这两种情况下，类型推断原则应用的是**模板类型推断**的规则。

~~~cpp
auto createInitList(){
	return  { 1, 2, 3 };	// wrong
}

std::vector<int> v;
auto resetV = [&v](const auto& newValue) { v = newValue; };
resetV({ 1, 2, 3});		// wrong
~~~

<a name='03'></a>
### 03. 理解decltype
> `decltype`几乎总是机械地返回变量或者表达式的确切类型，并没有进行类型推断

在c++11中，`decltype`最主要的用途可能是用来声明模板函数的返回值类型，而这些模板的返回值是取决于传进来的参数。
~~~cpp
// C++11
template <typename Container, typename Index>
auto authAndAccess(Container &c, Index i) -> decltype(c[i]) {
	authenticateUser();
	return c[i];
}
~~~
这是c++11的返回类型后置语法，`auto`并没有进行类型推断，返回类型取决于`->`之后的参数。C++14可以推断所有函数的返回类型，因此可以省略`->`。

~~~cpp
// C++14
template <typename Container, typename Index>
decltype(auto)		// Attention!!!
authAndAccess(Container &c, Index i) {
    authenticateUser();
    return c[i];
}
~~~
绝大多数容器的`operator[]`函数的返回类型为`T&`，使用模板参数类型推断后会变成`T`。因此在C++14中制定了`decltype(auto)`，返回确切类型。当然`decltype(auto)`不只是可以用在函数返回值类型上，它也可以用在声明变量上。

#### #优化authAndAccess函数
1. 使用通用引用同时满足绑定左值和右值，见[条款24](#24)
2. 使用`std::forward`来转发通用引用，见[条款25](#25)

~~~cpp
// C++14最终版本
template <typename Container, typename Index>
decltype(auto) 
authAndAccess(Container &&c, Index i) {
    authenticateUser();
    return std::forward<Container>(c)[i];
}
~~~

#### #一个例外
`decltype`几乎总是返回你期待的类型，但也有例外。由于`decltype`行为很难理解，因此举一个简单的例子。
~~~cpp
int x = 0;
// decltype(x) -> int
// decltype((x)) -> int&
~~~

<a name='04'></a>
### 04. 懂得如何查看已推断类型

#### #编译器诊断
~~~cpp
template <typename T>
class TD;
TD<decltype(x)> xType;
~~~
类`TD`并没有定义，因此实例化时编译器会报错，提供x的类型

#### #运行时输出
`typeid`的信息不准确，因为它也是通过传值方式传递的。
~~~cpp
std::cout << typeid(x).name() << std::endl;
~~~

使用`Boost.TypeIndex`得到准确信息
~~~cpp
#include <boost/type_index.hpp>
template <typename T>
void f(const T& param){
    using std::cout;
    using boost::typeindex::type_id_with_cvr;
    cout << "T =     "
         << type_id_with_cvr<T>().pretty_name()
         << '\n';

    cout << "param = "
         << type_id_with_cvr<decltype(param)>().pretty_name();
}
~~~


<a name='31'></a>
### 31. 避免使用默认捕获模式
#### #默认引用捕获缺点：
引用捕获会导致闭包包含一个**局部变量的引用**或者一个**形参的引用**。如果一个由lambda创建的闭包的生命期超过了局部变量或者形参的生命期，那么闭包的引用将会空悬。
~~~cpp
using FilterContainer = std::vector<std::function<bool(int)>>;
FilterContainer filters;

void f(){	// 离开f()，divisor生命期结束，造成引用空悬
	int divisor = 10;
	filters.emplace_back([&divisor](int value) { return value % divisor == 0; });
}
~~~
解决这个问题的一种办法是对divisor使用默认的值捕获模式。但是，总的来说，默认以值捕获不是对抗空悬的长生不老药。
#### #默认值捕获缺点：
第一，如果你用值捕获了个**指针**，你在`lambda`创建的闭包中持有这个指针的拷贝，但你不能阻止`lambda`外面的代码删除指针指向的内容，从而导致你拷贝的指针空悬。
~~~cpp
void Widget::addFilter() const {
	// 捕获了*this，出问题
	filters.emplace_back([=](int value) { return value % divisor == 0; });
	// 捕获了this->divisor
	auto divisorCopy = divisor;
	filters.emplace_back([divisorCopy](int value) { return value % divisorCopy == 0; });
	// C++14 广义lambda捕获
	filters.emplace_back([divisor=divisor](int value) { return value % divisor == 0; });
}
~~~

第二，给你一种捕获了某些变量的错觉
~~~cpp
void addDivisorFilter(){
	static auto divisor = 10;
	// 实际上并没有捕获到divisor
	filters.emplace_back([=](int value){ return value % divisor == 0; });
	++divisor;
};
~~~

<a name='32'></a>
### 32. 使用初始化捕获来把对象移动到闭包
C++14直接支持将对象**移动到闭包**，可以避免有些对象昂贵的拷贝操作。**初始化捕获**可以做C++11捕获格式能做的所有事情，唯一不能表示的是默认捕获模式，不过[条款31](#31)解释过无论如何你都应该远离默认捕获模式。

使用初始化捕获让你有可能指定：
1. 成员变量的名字（由lambda生成的闭包类的成员变量）
2. （初始化成员变量的）表达式 


#### #例子：通过初始化捕获来把`std::unique_ptr`移动到闭包内
~~~cpp
class Widget {
public:
    bool isValidated() const;
};
auto pw = std::make_unique<Widget>();	//创建Widget
...              			// 配置*pw
auto func = [pw = std::move(pw)]{ return pw->isValidated(); }
~~~
`pw = std::move(pw)`的意思是：在闭包中创建一个成员变量pw，然后用对局部变量pw使用`std::move`的结果初始化那个成员变量。

如果配置pw不是必需的，即，如果`std::make_unique`创建的Widget对象的状态已经适合被lambda捕获，那么局部变量pw是不必要的，因为闭包类的成员变量可以直接被`std::make_unique`初始化
~~~cpp
auto func = [pw = std::make_unique<Widget>()]{ return pw->isValidated(); };
~~~

在C++11中，只能手动创建一个类，
~~~cpp
class IsVal{
public:
	using DataType = std::unique_ptr<Widget>;
	explicit IsVal(DataType&& ptr): pw(std::move(ptr)) {}
	bool operator()() const{ return pw->isValidated()}
private:
	DataType pw;
};

auto func = IsVal(std::make_unique<Widget>());
~~~
或者是采用绑定（见下一部分）
~~~cpp
auto func = std::bind(
	[](const std::unique_ptr<Widget>& pw){ return pw->isValidated() },
	std::make_unique<Widget>()
);
~~~

#### #在C++11中模仿移动捕获
关键点：
1. 在一个C++11闭包中移动构造一个对象是不可能的，但在**绑定对象**中移动构造一个对象是有可能的。
2. 在C++11中模仿移动捕获需要在一个绑定对象内移动构造出一个对象，然后把该移动构造对象以**引用**传递给lambda。
3. 因为绑定对象`bind`的生命期和闭包`lambda`的生命期相同，可以把绑定对象中的对象（即除可执行对象外的实参的拷贝）看作是闭包里的对象。

~~~cpp
std::vector<double> data;
// C++14
auto func = [data = std::move(data)]{/* uses of data */};
// C++11
auto func = std::bind(
	[](const std::vector<double>& data) { /* uses of data */ },
	std::move(data)
);
~~~

<a name='33'></a>
### 33. 




