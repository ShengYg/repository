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
- 第2章 auto
  - [05. 尽量用auto代替显式类型声明](#05)
  - [06. 当auto推断不合理时使用显式类型初始化语法](#06)
- 第3章 modern C++
  - [07. 创建对象时区分( )和{ }](#07)
  - [08. 用nullptr代替0和NULL](#08)
  - [09. 用别名声明(alias declaration)代替typedef](#09)
  - [10. 比起unscoped enums更偏爱scoped nums](#10)
  - [11. 用deleted functions代替private undefined的做法](#11)
  - [12. 把重写函数(overriding function)声明为override](#12)
  - [13. 比起iterator更偏爱const_iterator](#13)
  - [14. 把不发出异常的函数声明为noexcept](#14)
  - [15. 尽可能使用constexpr](#15)
  - [16. 使const成员函数成为线程安全函数](#16)
  - [17. 理解特殊成员函数的生成](#17)
- 第4章 智能指针
  - [18. 用std::unique_ptr管理独占所有权的资源](#18)
  - [19. 用std::shared_ptr管理共享所有权的资源](#19)
  - [20. 把std::weak_ptr当作类似std::shared_ptr的、可空悬的指针使用](#20)
  - [21. 比起直接使用new，更偏爱使用std::make_unique和std::make_shared](#21)
  - [22. 当使用Pimpl Idiom时，在实现文件中定义特殊成员函数](#22)
- 第5章 右值、移动、完美转发
  - [23. 理解std::move和std::forward](#23)
  - [24. 区分通用引用和右值引用](#24)
  - [25. 对右值引用使用std::move，对通用引用使用std::forward](#25)
  - [26. 避免对通用引用进行重载](#26)
  - [27. 熟悉替代重载通用引用的方法](#27)
  - [28. 理解引用折叠](#28)
  - [29. 假设移动操作是不存在的、不廉价的、不能用的](#29)
  - [30. 熟悉完美转发失败的情况](#30)
- 第6章 lambda函数
  - [31. 避免使用默认捕获模式](#31)
  - [32. 使用初始化捕获来把对象移动到闭包](#32)
  - [33. 对需要std::forward的auto&&参数使用decltype](#33)
  - [34. 比起std::bind更偏向使用lambda](#34)
- 第7章 并行API
- 第8章 技巧

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

使用`Boost.TypeIndex`可以得到准确的类型信息
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

<a name='05'></a>
### 05. 尽量用auto代替显式类型声明
用`auto`的好处：
1. 强制初始化
~~~cpp
auto x = 10;
~~~
2. 避免冗长的类型声明
~~~cpp
std::iterator_traits<It>::value_type currValue = *b
~~~
3. 有能力直接持有闭包
~~~cpp
std::function<bool(const std::unique_str<Widget> &, const std::unique_str<Widget> &)>
	derefUPLess = [](const std::unique_str<Widget> &p1, 
		const std::unique_str<Widget> &p2) 
		{ return *p1 < *p2; };
// C++14
auto derefLess = [](const auto &p1, const auto &p2) { return *p1 < *p2; };
~~~
`std::function`声明的变量通过实例化模板而持有闭包，使用的内存总是比`auto`类型推断的对象多。`std::function`由于其实现细节而限制了内联，并且是间接函数调用，所以几乎可以肯定的会比`auto`类型推断的对象中调用要慢。
4. 避免类型截断这个问题
~~~cpp
std::vector<int> v;
unsigned sz = v.size();
std::vector<int>::size_type sz = v.size();
~~~
`unsigned`是32位，而`std::vector<int>::size_type`与系统相关
5. 避免无心的错误
~~~
std::unordered_map<std::string,int> m;
for (const std::pair<const std::string, int> &p : m) { }
// 如果不加第二个const，会增加许多无用的转换
~~~

> 总结：尽量使用`auto`，并注意[条款02](#02)与[条款04](#04)的陷阱

<a name='06'></a>
### 06. 当auto推断不合理时使用显式类型初始化语法
#### 1. auto无法处理不想让用户知道的代理类
~~~cpp
std::vector<bool> features(const Widget &w) { }
bool highPriority = features(w)[5];
~~~
通常`operator[]`函数应该是返回`T&`类型，但是`std::vector<bool>`容器是以位(bit)的方式来存储bool变量，而C++无法引用位，因此只能返回一个行为类似`bool&`的对象。`std::vector<bool>::reference`的一种实现是该对象内有一个指向容器内字(word)数据结构的偏移量特定个位(bit)数的指针。使用`auto`推断时，`highPriority`是对`reference`的拷贝。当构造函数结束，`reference`析构，`highPriority`成了一个悬空指针。

类似的还有`Matrix sum = m1 + m2 + m3 + m4;`中，`operator+`会返回代理类

#### 2. 显式类型初始化语法
对于上述问题，`auto`推断出的类型是代理类，而不是代理类所代理的类。

**显式类型初始化语法**用auto声明变量，但是初始化表达式显式说明你想要auto推断的类型
~~~cpp
// 对代理类进行转换
auto highPriority = static_cast<bool>(features(w)[5]);
auto sum = static_cast<Matrix>(m1 + m2 + m3 + m4);
// 其他转换
auto ep = static_cast<float>(calcEpsilon())
~~~

<a name='07'></a>
### 07. 创建对象时区分( )和{ }
因为初始化的语法很混乱，而且有些情况无法实现，所以C++11提出了提出了统一初始化语法：**大括号初始化**。

~~~cpp
// 初始化
std::vector<int> v{1, 3, 5};

// 类内成员初始化
class Widget {
	int x{ 0 };
	int y( 0 );	// wrong
	int z = 0;
}

// 拷贝对象
std::atomic<int> ai1{ 0 };
std::atomic<int> ai2( 0 );
std::atomic<int> ai3 = 0;	// wrong

// 隐式类型转换
double x, y, z;
int sum1{x + y + z};	// wrong
int sum2(x + y + z);
int sum3 = x + y + z;

// most vexing parse
Widget w1(10);	// 带参构造函数
Widget w2();	// 歧义，声明函数，而非无参构造函数
Widget w3{};	// 无参构造函数
~~~

缺点：如果构造函数的形参带有`std::initializer_list`，调用构造函数时大括号初始化语法会强制使用带`std::initializer_list`参数的重载构造函数，包括正常的拷贝构造和赋值构造。

~~~cpp
std::vector<int> v1(10, 20);
std::vector<int> v2{10, 20};
~~~

在模板中问题更加明显。这正是标准库函数`std::make_unique`和`std::make_shared`所面临的一个问题[条款21](#21)。这些函数的解决办法是强制要求把参数写在圆括号内，然后在接口中说明这个决策。

~~~cpp
template <typename T, typename... Ts>
void doSomeWork(Ts&&... params) {
	T localObject(std::forward<Ts>(params)...);
	T localObject{std::forward<Ts>(params)...};
}

std::vector<int> v;
doSomeWork<std::vector<int>>(10,20);	// 歧义
~~~

<a name='08'></a>
### 08. 用nullptr代替0和NULL
> `nullptr`的实际类型是`std::nullptr_t`，是可以隐式转换为**所有类型的原生指针**。

例子：3个函数有不同的指针类型，通过`nullptr`设计模板实现重载
~~~cpp
int f1(std::shared_ptr<Widget> spw);
double f2(std::unique_ptr<Widget> upw);
bool f3(Widget *pw);

std::mutex f1m, f2m, f3m;	// f1，f2， f3的互斥锁
using MuxGuard = std::lock_guard<std::mutex>;

template <typename FuncType, typename MuxType, typename PtrType>
auto lockAndCall(FuncType func, MuxType &mutex, PtrType ptr) -> decltype(func(ptr)){
	MuxGuard g(mutex);
	return func(ptr);
}

auto result = lockAndCall(f3, f3m, nullptr);	// nullptr支持各种隐式转换
~~~

<a name='09'></a>
### 09. 用别名声明(alias declaration)代替typedef
一些简单的区别：
~~~cpp
typedef std::unique_ptr<std::unordered_map<std::string, std::string>> UPtrMapSS;
using UPtrMapSS = std::unique_ptr<std::unordered_map<std::string, std::string>>;

typedef void (*PF)(int, const std::string &);
using PF = void (*)(int, const std::string &);
~~~

> typedef不支持模板化，但别名声明(alias declaration)支持。

~~~cpp
// using
template <typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;

template <typename T>
class Widget {
	MyAllocList<T> list;	// 在模板中创建list
};

MyAllocList<Widget> lw;		// 创建list

// typedef
template <typename T>
struct MyAllocList { 
  typedef std::list<T, MyAlloc<T>> type;
};

template <typename T>
class Widget {
	typename MyAllocList<T>::type list;
};

MyAllocList<Widget>::type lw;
~~~

头文件`<type_traits>`中包含了进行类型转换的工具，C++14对其进行了改变
~~~cpp
// C++11
std::remove_const<T>::type 		// const T → T
std::remove_reference<T>::type 		// T&/T&& → T
std::add_lvalue_reference<T>::type 	// T → T&

// C++14
std::remove_const_t<T>
std::remove_reference_t<T>
std::add_lvalue_reference_t<T>
~~~

<a name='10'></a>
### 10. 比起unscoped enums更偏爱scoped nums
#### 1. `scoped enum`减少命名空间污染
~~~cpp
// unscoped enums
enum Color { black, white, red };
auto white = false;		// 错误，当前作用域已经声明了white

// scoped enums
enum class Color { black, white, red };
auto white = false;
Color c = white;		// wrong
Color c = Color::white;
auto c = Color::white;
~~~

#### 2. `scoped enum`枚举值无法被隐式转换为其他类型，需要`cast`来转换
#### 3. `scoped enum`可以前向声明(forward-declaration)，它可以不带值地声明枚举的名字；`unscoped enum`也可以前向声明，不过需要额外的工作。这是因为`scoped enum`的基础类型总是int，而对于`unscoped enum`需要指定它的类型。
~~~cpp
enum Color;		// wrong
enum class Color;	// right

enum class Status;			// 基础类型是int
enum class Status: std::uint32_t;	// 基础类型是uint32_t
~~~

<a name='11'></a>
### 11. 用deleted functions代替private undefined的做法
> 优势1：删除的函数在任何方式上都无法使用，所以在成员或者友元中尝试操作对象会无法通过编译，这样就不用到链接期间才诊断出不合法使用。

**被删除函数**声明为public，而不是private。当用户代码尝试调用一个成员函数时，C++会在检查它的删除状态位之前检查它的可获取性(即是否为public)
~~~cpp
template <class charT, class traits = char_traits<char T>>
class basic_ios : public ios_base {
public:
	...
private:
	basic_ios(const basic_ios&);
	basic_ios& operator=(const basic_ios&);
};

template <class charT, class traits = char_traits<char T>>
class basic_ios : public ios_base {
public:
	basic_ios(const basic_ios&) = delete;
	basic_ios& operator=(const basic_ios&) = delete;
};
~~~

> 优势2：删除的函数可以用于任何函数，而private只能用在成员函数中，这会影响**重载函数**

~~~cpp
bool isLucky(int number);
bool isLucky(char) = delete; 
~~~

> 优势3：删除的函数可以防止模板使用不想要的**类型实例化**。

~~~cpp
template <typename T> 
void processPointer(T *ptr);
template<>
void processPointer<void>(void *) = delete;
template<>
void processPointer<const void>(const void *) = delete;

class Widget {
public:
	template <typename T>
	void processPointer(T *ptr) {...}
private:
	template<>	// 错误，成员模板的特例化与主模板的访问权限不相同是不可能
	void processPointer<void>(void *);
};
template<>		// 正确
void Widget::processPointer<void>(void *) = delete;
~~~

<a name='12'></a>
### 12. 把重写函数(overriding function)声明为override

派生类经常需要重写基类的虚函数。重写函数需要遵循：
- 基类的函数必须是虚函数(virtual)
- 基类和派生类的函数名字必须相同(除了析构函数)
- 基类和派生类的函数参数必须相同
- 基类和派生类的const属性必须相同
- 基类和派生类的返回类型和异常规范(exception specifications)必须兼容
- **C++11新增**：函数的引用限定符(reference qualifiers)必须相同

~~~cpp
class Widget {
public:
	using DataType = std::vector<double>;
	DataType& data() & { return values; }
	DataType data() && { return std::move(values); }
private:
	DataType values;
};

Widget makeWidget();
Widget w;
auto vals1 = w.data();			// 调用左值引用
auto vals2 = makeWidget().data();	// 调用右值引用
~~~

声明为`override`可以强制编译器提醒错误

~~~cpp
class Derived: public Base {
public:
    virtual void f() const override;
};
~~~

<a name='13'></a>
### 13. 比起iterator更偏爱const_iterator
C++11只加了非成员函数版本的begin和end，而没有加入cbegin，cend，rbegin，rend，crbegin和crend。C++14修正了这问题。
~~~cpp
template <typename C, typename V>
void findAndInsert(C& container, const V& targetVal, const V& insertVal) {
    using std::cbegin;
    using std::cend;
    auto it = std::find(cbegin(container), cend(container), targetVal);
    container.insert(it, insertVal);
};
~~~

<a name='14'></a>
### 14. 把不发出异常的函数声明为noexcept
`noexcept`说明函数保证不会发出异常，是函数接口的一部分，它允许编译器生成更好的目标代码。

三个例子：
1. C++11中向`vector`添加元素，没有直接用移动代替拷贝（若在移动中抛出异常，原`vector`状态会改变），而是可以用移动的话就移动，不行就一定要用拷贝（move if you can, but copy if you must）。具体实现就是检查移动操作是否用noexcept声明
2. 标准库中的`swap`是否是noexcept取决于用户定义swap是否为noexcept
~~~cpp
template <class T, size_t N>
void swap(T (&a)[N], T (&b)[N]) noexcept(noexcept(swap(*a,*b)));
//
template <class T1, class T2>
struct pair {
	void swap(pair& p) noexcept(noexcept(swap(first, p.first)) &&
			noexcept(swap(second, p.second)));
};
~~~
3. 所有的**释放内存函数和析构函数**，不管是用户自定义还是编译器生成的，都是隐式noexcept的。

事实上大部分函数是**异常中立**的。这些函数自身没有抛任何异常，不过它们调用的函数可能发出异常，因此大部分函数普遍不适用noexcept。但是一些函数，尤其是移动赋值操作和swap，声明为noexcept有重大回报，这值得我们尽可能地把它们声明为noexcept。

<a name='15'></a>
### 15. 尽可能使用constexpr
在概念上，constexpr表明一个值不仅是常量，还是在编译期间可知。

#### 1. constexpr对象：

它们的值在编译期间就知道了，适用于数组大小的表示，整型模板参数（包括std::array对象的长度），枚举的值，对齐说明，等等。

> `const`并不提供与`constexpr`相同的保证，因为`const`对象在编译时不需要用已知的值**初始化**。

~~~cpp
constexpr auto arraySize2 =  10;
std::array<int, arraySize2> data2;

int sz;
const auto arraySize = sz;
std::array<int, arraySize> data;	// 错误，arraySize的值在编译期间不可知
~~~

#### 2. constexpr函数:
规则：
- constexpr函数可以用在**需求编译期间常量**的上下文。如果你传递参数的值在编译期间已知，那么函数的结果会在编译期间计算。如果任何一个参数的值在编译期间未知，代码将不能通过编译。
- 如果用一个或者多个在编译期间未知的值作为参数调用constexpr函数，函数的行为和普通的函数一样，在运行期间计算结果。这意味着你不需要用两个函数来表示这个操作——一个在编译期间和一个在运行期间。

在C++11，constexpr只能有一个return语句。C++14不限制。
~~~cpp
constexpr int pow(int base, int exp) noexcept{
	// C++11
	return (exp == 0 ? 1 : base * pow(base, exp - 1));
	// C++14
	auto result = 1;
	for (int i=0; i<exp; ++i) result *= base;
	return results;
}

constexpr auto numCouds = 5;
std::array<int, pow(3, numCouds)> results;
~~~

constexpr函数要求持有和返回的类型为**字面值类型**。在C++中，**除了void之外**的内置类型都是字面值类型，用户定义的类型也有可能是字面值类型。
~~~cpp
class Point {
public:
	// constexpr构造函数，表明constexpr对象
	constexpr Point(double xVal = 0, double yVal = 0) noexcept
	: x(xVal), y(yVal) {}

	constexpr double xValue() const noexcept { return xVal; }
	constexpr double yValue() const noexcept { return yVal; }

	void setX(double newX) noexcept { x = newX; }
	void setY(double newY) noexcept { y = newY; }

private:
	double x, y;
};

constexpr Point p2(28.8, 5.3);
constexpr Point midpoint(const Point &p1, const Point &p2) noexcept {
    return { (p1.xValue + p2.xValue)) / 2, (p1.yValue + p2.yValue)) / 2 };
}
constexpr auto mid = midpoint(p1, p2);
~~~

在C++11，有两个限制因素妨碍把Point的成员变量setX和setY声明为constexpr。第一，它们改变了它们操作的值，constexpr成员函数是隐式声明为const的。第二，它们的返回类型是void。但是这两个限制在C++14被解除了，所以在C++14，**设置函数也可以constexpr**

<a name='16'></a>
### 16. 使const成员函数成为线程安全函数
例子：多项式求根。const成员函数通常是线程安全的，除非含有mutable成员变量。
~~~cpp
class Polynomial {
public:
	using RootsType = std::vector<double>;
	RootsType roots() const{	// 通常不改变成员，设为const
		if (!rootsAreValid)  {
			...		// 计算并存储结果
			rootsAreVaild = true;
		}
		return rootVals;
	}
private:
	mutable bool rootAreValid{ false };	// 可能被改变，设为mutable
	mutable RootsType rootVals{};
};
~~~
解决办法是使用`mutex`。mutex是一个只可移动类型，使得多项式类也只能被移动。

~~~cpp
class Polynomial {
public:
	RootsType roots() const {
		std::lock_guard<std::mutex> g(m);	// 加锁
		if (!rootsAreValid) {
			...
			rootsAreValid = true;
		}
		return rootVals;
	}						// 解锁
private:
	mutable std::mutex m;
};
~~~

使用`std::atomic`变量可能比互斥锁提供更好的性能，不过它们只适用于**单一变量和单一存储单元**。

<a name='17'></a>
### 17. 理解特殊成员函数的生成

类中的两个**拷贝操作**是独立的：声明了其中一个不会阻止编译器生成另一个。类中的两个**移动操作**不是独立的，如果你声明了其中一个，那会阻止编译器生成另一个。这里的根据是：如果你声明了一个移动构造函数，暗示着你的移动构造函数实现与编译器产生的默认逐一移动实现不同，那么如果逐一移动的构造函数是有问题的，那么逐一移动的赋值运算可能也有问题。

显式声明拷贝操作的类不能生成移动操作。正当的理由是：声明了拷贝操作暗示着正常的拷贝对象的方法是不适合这个类的，然后编译器认为如果成员逐一拷贝不适合操作操作，成员逐一移动可能也不会适合移动操作。

三大法则规定：如果你声明了拷贝构造、拷贝复制、析构函数中的其中一个，你应该把这三个都声明。

~~~cpp
class Widget {
public:
	~Widget();
	Widget(const Widget&) = default;		// 使用默认拷贝构造
	Widget& operator=(const Widget&) = default;	// 使用默认拷贝复制操作
};
~~~

因此C++11管理特殊成员函数是这样的：
- 析构函数：本质上C++98的规则相同，唯一的区别就是析构函数默认声明为noexcept（看条款14）。C++98的规则是基类的析构函数的虚函数的话，生成的析构函数也是虚函数。
- 拷贝构造函数：如果类中声明了移动操作，拷贝构造会被删除；当类中存在用户声明的拷贝赋值操作符或析构函数时，不建议用生成的拷贝构造函数。
- 拷贝赋值运算符：如果类中声明了移动操作，拷贝赋值运算符会被删除；当类中存在用户声明的拷贝构造函数或析构函数时，不建议用生成的拷贝赋值运算符。
- 移动构造函数和移动赋值运算符：只有在类中没有用户声明的拷贝操作、移动操作、析构函数时才会自动生成。
- 没有规则说明成员函数模板会阻止编译器生成特殊成员函数

<a name='18'></a>
### 18. 用std::unique_ptr管理独占所有权的资源
`std::unique_ptr`表示独占所有权语义。一个非空的`std::unique_ptr`会一直拥有它指向的对象，只可移动，不能拷贝。

一个常见的例子是工厂函数，另一个是Pimpl Idiom技术，见[条款22](#22)
~~~cpp
class Investment { ... };
class Stock : public Investment { ... };
class Bond : public Investment { ... };
class RealEstate : public Investment { ... };

template <typename... Ts>
std::unique_ptr<Investment>
makeInvestment(Ts&&... params);

{	// 在局部作用域中生成指针
	auto pInvestment = makeInvestment(arguments);
} 
~~~

默认情况下，`std::unique_ptr`是借助delete来销毁管理的资源，但是，在构造期间，你可以指定使用自定义的删除器。如果删除器是函数指针，它通常会让std::unique_ptr的大小增加一到两个字节。如果删除器是函数对象，std::unique_ptr的大小改变取决于函数对象存储了多少状态。无状态的函数对象（不捕获变量lambda表达式）不会受到一丝代价。

~~~cpp
auto delInvmt = [](Investment *pInvestment) {
			makeLogEntry(pInvestment);	// 额外的删除工作
			delete pInvestment;
		};

template <typename... Ts>
std::unique_ptr<Investment, decltype(delInvmt)>
makeInvestment(Ts&&... params) {	// 定义的删除器作为第二个模板参数
	std::unique_ptr<Investment, decltype(delInvmt)> pInv(nullptr, delInvmt);
	if (...) {
		pInv.reset(new Stock(std::forward<Ts>(params)...));
	}
	else if (...) {
		pInv.reset(new Bond(std::forward<Ts>(params)...));
	}
	else if (...) {
		pInv.reset(new RealEstate(std::forward<Ts>(params)...));
	}
	return pInv;
}
~~~

`std::unique_ptr`可以直接作为**右值**转化为`std::shared_ptr`。
~~~cpp
// unique_ptr作为右值直接转换
std::shared_ptr<Investment> sp = makeInvestment(argument); 
~~~

<a name='19'></a>
### 19. 用std::shared_ptr管理共享所有权的资源
引用计数的工作方式：
- `std::shared_ptr`的大小是原生指针的两倍，因为它包含一个指向资源的原生指针和引用计数。
- 引用计数所用的内存一定是动态分配的。
- 增加和减少引用计数一定是原子操作，因为在不同的线程中会同时存在读和写。

引用计数是一个更大的数据结构的一部分，这个数据结构是control block（控制块）。每个shared_ptr管理的对象都有一个控制块，这个控制卡除了包含引用计数外，还有一份自定义删除器的拷贝，自定义分配器的拷贝。

<center>
<img src="{{ site.baseurl }}/assets/pic/shared_ptr.png" height="200px" >
</center>

控制块创建要服从以下规则：
- std::make_shared总是会创建控制块。
- 当std::shared_ptr由独占所有权指针(即std::unique_ptr或std::auto_ptr)构造时，控制块会被创建。
- 当以原生指针为参数调用std::shared_ptr的构造函数时，会创建控制块。如果你想从已有控制块的对象创建一个std::shared_ptr，你可能要传递一个std::shared_ptr或std::weak_ptr作为构造函数的参数。因此，通过指向动态分配的对象的原生指针创建std::shared_ptr是糟糕的。

`std::shared_ptr`也支持自定义删除器，但是它的类型不是智能指针的类型，因此也存在一个容器中存有不同删除器类型的shared_ptr。
~~~cpp
auto loggingDel = [](Widget *pw) {
			makeLogEntry(pw);
			delete pw;
		  };
std::unique_ptr<Widget, decltype(loggingDel)> upw(new Widget, loggingDel);
std::shared_ptr<Widget> spw(new Wiget, loggingDel); 
~~~

两点注意：
- 避免用原生指针构造std::shared_ptr，通常的选择是使用std::make_shared。
- 如果你一定要用原生指针构造std::shared_ptr，那么直接把new出来的结果传递过去，而不是传递原生指针变量
~~~cpp
std::shared_ptr<Widget> spw1(new Widget， loggingDel);
std::shared_ptr<Widget> spw2(spw1);	// 调用拷贝构造
~~~

如果类成员要处理this指针，可以继承`enable_shared_from_this`模板。为了保证成员函数在使用前已经用shared_ptr指向当前对象，会将构造函数声明为private，使用工厂创建对象。
~~~cpp
// 模板参数为它的派生类名字
class Widget : public std::enable_shared_from_this<Widget> {
public:
	template<typename... Ts>
	static std::shared_ptr<Widget> create(Ts&&... params);
	void process();
private:
	...	// 构造函数
};
void Widget::process() {
	// 使用this创建shared_ptr对象，并且不带重复的控制块
	processedWidgets.emplace_back(shared_from_this());
}
~~~

<a name='20'></a>
### 20. 把std::weak_ptr当作类似std::shared_ptr的、可空悬的指针使用
`std::weak_ptr`要和`std::shared_ptr`搭配使用，不能被解引用，也不能检测是否为空，不会影响对象的引用计数，能够追踪它什么时候对象被销毁。
~~~cpp
auto spw = std::make_shared<Widget>();
std::weak_ptr<Widget> wpw(spw);
~~~
通常需要一个原子操作检查std::shared_ptr是否过期，没有的话，给你它指向的对象。有两种形式。一种形式是std::weak_ptr::lock，它返回一个std::shared_ptr。如果std::weak_ptr过期了，std::shared_ptr就为空。
~~~cpp           `
auto spw1 = wpw.lock();
~~~
另一种形式是使用接受std::weak_ptr为参数的std::shared_ptr构造函数，这种情况呢，如果std::weak_ptr过期了，会抛出异常： 
~~~cpp
std::shared_ptr<Widget> spw2(wpw);
~~~

常见应用：
1. 应用1：带缓存的工厂函数
~~~cpp
std::shared_ptr<const Widget> fastLoadWidget(WidgetID id) {
    static std::unordered_map<WidgetID, std::weak_ptr<const Widget>> cache; 
    auto objPtr = cache[id].lock();
    if (!object) {
        objPtr = loadWidget(id);
        cache[id] = objPtr;
    }
    return object;
}
~~~
2. 应用2：观察者模式
每个主题持有一个元素为std::weak_ptr的容器，std::weak_ptr指向主题的每个观察者，因此主题在使用观察者之前可以查看指针是否空悬
3. 应用3：循环引用。B对A使用weak_ptr，避免循环引用。如果是树形结构，子结点指向父结点的指针可以用原生指针安全实现。
<center>
<img src="{{ site.baseurl }}/assets/pic/weak_ptr.png" height="70px" >
</center>

<a name='21'></a>
### 21. 比起直接使用new，更偏爱使用std::make_unique和std::make_shared
`make_shared`是C++11的一部分，`make_unique`在C++14才纳入标准库。可以写一个简单的`make_unique`的C++11版本
~~~cpp
template<typename T, typename... Ts>
std::unique_ptr<T> make_unique(Ts&&... params){
    return std::unique_ptr<T>(new T(std::forward<Ts>(params)...));
}
~~~

make函数的优点：
1. 避免代码重复。make函数内部是new函数，多次调用new导致代码重复。
2. 异常安全。make能立即获得对象的指针，并在发生异常时直接调用析构函数。
~~~cpp
processWidget(std::make_shared<Widget>(), computePriority())
processWidget(std::shared_ptr<Widget>(new Widget), computePriority());
~~~
3. 提高效率。make调用一次内存分配函数来同时持有对象和控制块。

make函数的缺点：
1. 不能指定自定义删除器
~~~cpp
auto widgetDeleter = [](Widget* pw) {...}
std::unique_ptr<Widget, decltype(widgetDeleter)> upw(new Widget, widgetDeleter);
std::shared_ptr<Widget> spw(new Widget, widgetDeleter);
~~~
2. 不适合大括号创建对象。[条款30]有另一种方法。
~~~cpp
auto upv = std::make_unique<std::vector<int>>(10, 20);
// 见条款30
auto initList = {10, 20};
auto spv = std::make_shared<std::vector<int>>(initList);
~~~
3. make_shared特有的：定义自己版本的operator new和operator delete的对象
4. make_shared特有的：只有std::shared_ptr和std::weak_ptr对象销毁，才能被回收。如果对象很大或持续时间长，不合适。

一种异常安全的不采用make的调用：即使new抛出异常，spw也能调用自定义的删除函数curDel
~~~cpp
std::shared_ptr<Widget> spw(new Widget, cusDel);
processWidget(std::move(spw), computeWidget); 	// computeWidget的异常无法干扰new
~~~

<a name='22'></a>
### 22. 当使用Pimpl Idiom时，在实现文件中定义特殊成员函数
Pimpl(“pointer to implementation”) Idiom：通过把类中的成员变量替换成指向一个实现类（的指针，成员变量被放进单独的实现类中，然后通过该指针间接获取原来的成员变量。

原来的类依赖于许多类型，在头文件中需要添加多个include，编译时间长。且头文件改变，就需要重新编译。通过Pimpl将include转移至cpp文件中。

~~~cpp
class Widget {			// 在头文件“widget.h”中
public:
	Widget();
private:
	std::string name;
	std::vector<double> data;
	Gadget g1, g2, g3; 	// 需要头文件gadget.h
};

// C++11改进版本
class Widget {			// 依然在头文件“widget.h”中
public:
    Widget();
private:
    struct Impl;		// 声明实现类
    Impl *pImpl;		// 声明指针指向实现类
};

#include "widget.h"		// 在实现文件“widget.cpp”
#include "gadget.h"
#include <string>
#include <vector>

struct Widget::Impl {		// 用原来对象的成员变量来定义实现类
    std::string name;
    std::vector<double> data;
    Gadget g1, g2, g3;
};
Widget::Widget() : pImpl(new Impl) {}

// C++14版本
class Widget {			// 在“widget.h”
public:
	Widget();
	~Widget();
private:
	struct Impl;
	std::unique_ptr<Impl> pImpl;
};

#include "widget.h" 		// 在“widget.cpp”
#include "gadget.h"
#include <string>
#include <vector>

struct Widget::Impl {
    std::string name;
    std::vector<double> data;
    Gadget g1, g2, g3;
};

Widget::Widget() : pImpl(std::make_unique<Impl>()) {}
Widget::~Widget() {}
~~~

注意：对于std::unique_ptr，删除器的类型是智能指针类型的一部分，这让编译器生成更小的运行时数据结构和更快的运行时代码成为可能。这高效导致的后果是当使用编译器生成的特殊成员函数时，指向的类型必须是完整类型。因此需要在头文件中声明特殊成员函数，但在实现文件中实现它们。std::shared_ptr相反。









<a name='23'></a>
### 23. 理解std::move和std::forward
`std::move`和`std::forward`仅仅是表现为**转换类型的函数**（实际上是模板函数），`std::move`无条件地把参数转换为右值，而`std::forward`在满足条件下才会执行`std::move`的转换。

> std::move接收一个对象的引用（准确地说，是**通用引用**），然后返回相同对象的**右值引用**。

近似的实现方式如下：
~~~cpp
// C++14
template <typename T>
decltype(auto) move(T&& param) {
	using ReturnType = remove_reference_t<T>&&;
	return static_cast<ReturnType>(param);
}
~~~
注意：想移动对象时，不要声明为`const`

> std::forward仅当参数是用右值初始化时，才会把它转换为右值。具体如何操作见[条款28](#28)。

通常`std::forward`可以替代`std::move`，但也有不同点：
- `std::move`通常会造成移动，而`std::forward`只是传递转发一个对象给另一个函数，而保持原来的左值性质或者右值性质
- `std::move`需要更少的类型，不用传递类型参数

~~~cpp
class Widget {
public:
	// 两种实现
	Widget(Widget&& rhs) : s(std::move(rhs.s)) { ++moveCtorCalls; }
	Widget(Widget&& rhs) : s(std::forward<std::string>(rhs.s)) { ++moveCtorCalls; }
private:
	static std::size_t moveCtorCalls;
	std::string s;
};
~~~

<a name='24'></a>
### 24. 区分通用引用和右值引用
`T&&`有两种含义：
- 右值引用：直接声明变量时
- 通用引用：包含模板推断（不含const）时，根据T的实际类型推断左值引用或右值引用

~~~cpp
// 右值引用
void f(Widget&& param);
Widget&& var1 = Widget();
template<typename T> void f(std::vector<T>&& param);

// 不是右值引用
auto&& var2 = var1;
template<typename T> void f(T&& param);
~~~


<a name='25'></a>
### 25. 对右值引用使用std::move，对通用引用使用std::forward
当把右值引用转发给其他函数时，右值引用应该**无条件**转换为右值（借助`std::move`），因为右值引用总是绑定右值。而当把通用引用转发给其他函数时，通用引用应该**有条**件地转换为右值（借助`std::forward`)，因为通用引用只是有时候会绑定右值。

1. 为确保这个对象不会被移动，在**最后一次使用**那个引用时，才用`std::move`或`std::forward`。
~~~cpp
template<typename T>
void setSignText(T&& text){ 
	sign.setText(text);	// 使用text，但不修改它
	auto now = std::chrono::system_clock::now();	// 获取当前时间
	signHistory.add(now, std::forward<T>(text));	// 有条件地把text转换为右值
}
~~~
2. 如果你有个**函数是通过值返回**，然后你函数内返回的是被右值引用或通用引用绑定的对象，那么你应该对你返回的对象使用`std::move`或`std::forward`。
~~~cpp
Matrix operator+(Matrix&& lhs, const Matrix& rhs) {
	lhs += rhs;
	return std::move(lhs);	// 移动到返回值
	return lhs;		// 拷贝到返回值
}
~~~
3. **RVO**（return value optimization）：在通过值返回的函数中，如果（1）一个局部变量的类型和返回值的类型相同，而且（2）这个局部变量是被返回的对象，那么编译器可能会省略局部变量的拷贝（或移动），此时不要对它们使用std::move或std::forward。

<a name='26'></a>
### 26. 避免对通用引用进行重载
接受通用引用作为参数的函数是C++最贪婪的函数，它们可以为几乎所有类型的参数实例化，从而创建的精确匹配。这就是为什么结合重载和通用引用几乎总是个糟糕的想法：通用引用重载吸收的参数类型远多于开发者的期望。
~~~cpp
template<typename T>
void logAndAdd(T&& name) {
	auto now = std::chrono::system_clock::now();
	log(now, "logAndAdd");
	names.emplace(std::forward<T>(name));
}

std::multiset<std::string> names;
std::string petName("Darla");
logAndAdd(petName);      		// 拷贝左值
logAndAdd(std::string("Persephone"));	// 移动右值
logAndAdd("Patty Dog");			// 在multisest内创建

std::string nameFromIdx(int idx);
void logAndAdd(int idx){
	auto now = std::chrono::system_clock::now();
	log(now, "logAndAdd");
	names.emplace(nameFromIdx(idx));
}
short nameIdx;
logAndAdd(nameIdx);   		// 匹配logAndAdd(T&& name)，错误
~~~

**完美转发构造函数**是特别有问题的，因为在接受非const左值作为参数时，它们通常比拷贝构造匹配度高，然后它们还能劫持派生类调用的基类的拷贝和移动构造。

如何解决这个问题，见[条款27](#27)

<a name='27'></a>
### 27. 熟悉替代重载通用引用的方法 
#### 1. **Tag dispatch**
在通用引用中加入额外的tag，如`std::false_type`和`std::true_type`。tag**没有命名**，因为它们在运行期间没有任何作用，只是希望编译器可以辨别出标签参数是不同寻常的，然后优化它们在运行期间的开销。它是模板元编程的基本构件。
~~~cpp
std::multiset<std::string> names;
// 原版本
template<typename T>
void logAndAdd(T&& name) {
	auto now = std::chrono::system_clock::now();
	log(now, "logAndAdd");
	names.emplace(std::forward<T>(name));
}
// 新版本
template<typename T>
void logAndAdd(T&& name) {
	logAndAddImpl(
		std::forward<T>(name),
		// 传入左值引用时有问题，需要移除引用
		std::is_integral<std::remove_reference<T>()
	);
}
template<typename T>
void logAndAddImpl(T&& name, std::false_type){
	auto now = std::chrono::system_clock::now();
	log(now, "logAndAdd");
	names.emplace(std::forward<T>(name));
}
void logAndAddImpl(int idx, std::true_type){
	logAndAdd(nameFromIdx(idx);
}
~~~

Tag dispatch不能解决完美转发构造函数的问题。当你想要调用编译器生成的拷贝构造函数时，它依然会跳过，而始终调用通用引用，重载依然不起作用。

#### 2. std::enable_if
默认地，所有模板都是enable（使能）的，不过模板使用`std::enable_if`后，只会在满足指定条件时才会被使能。在例子中，我们想要在传递给构造函数的参数类型不是Person时才使能完美转发构造函数，如果传递的参数类型是Person，我们打算disable完美转发构造函数，由类的拷贝或移动构造来处理这次调用。

~~~cpp
class Person {
public:
	// 原来代码
	template<typename T>
	explicit Person(T&& n) : name(std::forward<T>(n)) {}
	explicit Person(int idx);
	// 修改后，只有声明，定义相同
	template<
		typename T, 
		typename = typename std::enable_if<
			!std::is_same<Person, typename std::decay<T>::type>::value
		>::type
	>
	explicit Person(T&& n);

	...
};
~~~

`std::enable_if`中的表达式判断了T和Person的类型是否一致。`std::decay<T>`表示去除T的各种引用、const修饰符、volatile修饰符。

对于Person的派生类，它在调用基类的完美转发构造时进行判断，由于SpecialPerson与Person不同，依然会使用完美转发构造函数，问题依然存在。调用`std::is_base_of`代替`std::is_same`

~~~cpp
class Person {
public:
	template<
		typename T, 
		typename = typename std::enable_if<
			!std::is_base_of<Person, typename std::decay<T>::type>::value
		>::type
	>
	explicit Person(T&& n);
};
~~~

最终版本：
~~~cpp
class Person {
public:
	template<
		typename T, 
		typename = typename std::enable_if<
			!std::is_base_of<Person, typename std::decay<T>::type>::value
			&&
			!std::is_integral<std::remove_reference_t<T>>::value
		>::type
	>
	explicit Person(T&& n)
	: name(std::forward<T>(n)) {...}

	explicit Person(int idx)
	: name(nameFromIdx(idx)) {...}
};
~~~

#### 3. 权衡
完美转发更具效率，因为它为了保持声明参数时的类型，它避免创建临时对象。但是完美转发有缺点:
1. 某些类型不能被完美转发，尽管它们可以被传递到函数，见[条款30](#30)
2. 当用户传递无效参数时，错误信息难以理解：任何类型被通用引用绑定时不报错，只有进一步进入构造函数时才报错。可以加入`static_assert`确保某种类型。

~~~cpp
static_assert(
	// 可以从T构造出string变量
	std::is_constructible<std::string, T>::value,
	"Parameter n can't be used to construct a std::string"
);
~~~

<a name='28'></a>
### 28. 理解引用折叠
编译器禁止声明对引用的引用，但在特殊的上下文中可以产生，模板实例化就是其中之一。当编译器生成对引用的引用时，引用折叠指令就会随后执行。

> 引用折叠：如果两个引用中有一个是左值引用，那么折叠的结果是一个左值引用。否则（即两个都是右值引用），折叠的结果是一个右值引用。

通用引用不是一种新的引用类型，实际上它是右值引用，根据左值和右值来进行类型推断，发生引用折叠。

~~~cpp
template<typename T>
void f(T&& fParam){
	someFunc(std::forward<T>(fParam));
}

// forward工作方式
template<typename T>
T&& forward(typename remove_reference<T>::type& param){
    return static_cast<T&&>(param);
}
~~~

出现引用折叠的情况：
1. 模板实例化
2. auto
3. 使用typedef和类型别名声明
4. 使用decltype

<a name='29'></a>
### 29. 假设移动操作是不存在的、不廉价的、不能用的
移动操作在许多情况下并不比拷贝好，原因有三：
1. 没有移动操作
2. 移动的速度不快。许多容器类型的对象，在概念上，只持有一个指针（作为成员变量），指向存储容器内容的堆内存，这个指针的存在使得用常量时间移动一个容器的内容成为可能。`std::array`不同，因为它的数据直接存储在对象中。`std::string`提供常量时间的移动和线性时间的拷贝，实际上移动不比拷贝快。许多string的实现都使用了small string optimization(SSO)，通过SSO，small string（例如，那些容量不超过15字符的string）会被存储到`std::string`对象内的一个缓冲区中；不需要使用堆分配的策略。
3. 不能使用移动操作。标准库一些容器操作提供**异常安全保证**，只有当移动操作不抛异常时，才会把内部的拷贝当作替换成移动操作。结果就是：即使一个类提供移动操作，编译器可能仍然会使用拷贝操作，因为它对应的移动操作没有声明为`noexcept`。

<a name='30'></a>
### 30. 熟悉完美转发失败的情况

完美转发：不单单转发对象，我们还转发它们重要的特性：它们的类型，它们是右值还是左值，它们是否是const或者volation修饰的。

完美转发可变参数模板
~~~cpp
template<typename... Ts>
void fwd(Ts&& ...params){
	f(std::forward<Ts>(param)...);
}
// 下面两行意思一致，代表完美转发成功
f( expression );
fwd(expression);
~~~
#### 1. 大括号初始值

~~~cpp
void f(const std::vector<int>& v);
f({1, 2, 3});
fwd({1, 2, 3});		// error
~~~

f中支持{1,2,3}到`std::vector`的隐式转换，但fwd没有`std::initialist_list`类型的模板参数。由于`auto`变量在用大括号初始值初始化时会进行类型推断，因此可以用auto声明一个局部变量。

~~~cpp
auto il = {1, 2, 3};
fwd(il);
~~~
#### 2. 0和NULL作为空指针
用`nullptr`代替
#### 3. 只声明的static const成员变量
~~~cpp
class Widget {
public:
	static const std::size_t MinVals = 28;         // MinVals的声明
};
const std::size_t Widget::MinVals;			// 定义MinVals
~~~
`static const`变量只有声明，没有定义，可以编译，但不能链接。直接使用变量没问题，但通过引用使用，就会链接失败。

#### 4. 重载函数名字和模板名字
f有具体的声明，因此知道调用哪一个函数，但fwd是模板，编译器无法决定。
~~~cpp
void f(int (*pf)(int));
int processVal(int value);
int processVal(int value, int priority);

f(processVal);		// right
fwd(processVal);	// wrong

template<typename T>
T workOnVal(T param)	// 一个处理值的模板
{ ... }
fwd(workOnVal);		// wrong
~~~

像fwd这种进行完美转发的函数，想要接受一个**重载函数名字**或者**模板名字**的方法是：手动指定你想要转发的那个重载或者实例化。

~~~cpp
using ProcessFuncType = int (*)(int);
ProcessFuncType processValPtr = processVal;
fwd(processValue);

fwd(static_cast<ProcessFuncType>(workOnVal));
~~~

#### 5. 位域（Bitfields）
~~~cpp
struct IPv4Header {
	std::uint32_t   version : 4,
			IHL : 4,
			DSCP : 6,
			ECN : 2,
			totalLength : 16;
};

void f(std::size_t sz);
IPv4Header h;
f(h.totalLength);	// right
fwd(t.totalLength);	// wrong
~~~

C++标准规定不是常量引用不能绑定位域（A non-const reference shall not be bound to a bit-field）。原因很简答：位域可能是包括机器字的任意部分，但是没有方法直接获取它们的地址。

~~~cpp
// 转发拷贝
auto length = static_cast<std::uint16_t>(h.totalLength);
fwd(length); 
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
### 33. 对需要std::forward的auto&&参数使用decltype
C++14最令人兴奋的特性之一是泛型lambda——lambda可以在参数说明中使用auto。
~~~cpp
auto f = [](auto x) { return func(normalize(x)); };

// lambda类似于下面一个类
class SomeCompilerGeneratedClassName {
public:
	template<typename T>
	auto operator()(T x) const { return func(normalize(x)); }
};
~~~
如果normalized区别对待左值和右值，这个lambda这样写是不合适的。第一，x要改成通用引用[条款24](#24)，第二，借助std::forward[条款25](#25)把x转发到normalized。
~~~cpp
auto f = [](auto&& x) { return func(normalized(std::forward<decltype(params)>(params)...)); };
~~~

params是左值，`decltype(params)`得到左值，`std::forward`得到左值；params是右值，`decltype(params)`得到右值，尽管是`std::forward`的非常规使用，但`std::forward`依然得到右值；

<a name='34'></a>
### 34. 比起std::bind更偏向使用lambda

#### 1. `lambda`具有更好的可读性
在bind1中，实现的是错误的代码，参数先求值后绑定，表明在绑定函数后1小时触发，而非调用函数后1小时触发。bind2正确。
~~~cpp
using Time = std::chrono::steady_clock::time_point;
enum class Sound {Beep, Siren, Whistle };
using Duration = std::chrono::steady_cloak::duration;

void setAlarm(Time t, Sound s, Duration d);

// lambda
auto setSoundL = [](Sound s) {
	using namespace std::chrono;
	using namespace std::literals;     // C++14支持时间后缀
	setAlram(steady_clock::now() + 1h, s, 30s); 
};
// bind1
using namespace std::placeholders;
auto setSoundB = std::bind(setAlarm, steady_clock::now() + 1h, _1, 30s);
// bind2
auto setSoundB = std::bind(setAlarm, 
		// 在C++14，标准操作符模板的模板类型参数可以被省略
		std::bind(std::plus<>(), steady_clock::now(), 1h),, _1, 30s);
~~~

#### 2. 函数被重载时,bind会出错
当函数被重载时，`lambda`会自动选择该被调用的函数，而bind会编译错误，除非用**函数指针**指定调用的函数。使用函数指针会降低内联的可能性，导致效率降低。
~~~cpp
using SetAlarm3ParamType = void(*)(Time t, Sound s, Duration d);
// 指针的强制转换
auto setSoundB = std::bind(static_cast<SetAlarm3ParamType>(setAlarm), 
		std::bind(std::plus<>(), steady_clock::now(), 1h),, _1, 30s);
~~~

另一个例子
~~~cpp
auto betweenL = [lowVal, highVal] (const auto& val)
	{ return lowVal <= val && val <= highVal; };

using namespace std::placeholders;
auto betweenB = std::bind(std::logical_and<>(),
			  std::bind(std::less_equal<>(), lowVal, _1),
			  std::bind(std::less_equal<>(), _1, highVal));
~~~

#### 3. bind只能通过引用传递
`lambda`可以指定传值或传引用，`bind`的函数调用操作符使用了完美转发，只能引用传递。

在C++14，没有理由使用`std::bind`。而在C++11，`std::bind`可以使用在受限的两个场合：
1. 移动捕获。C++11的lambda没有提供移动捕获，但可以结合std::bind和lambda来效仿移动捕获，见[条款32](#32)
2. 多态函数对象。绑定对象的函数调用操作符会使用完美转发，它可以接受任何类型的实参。C++
14`lambda`已经支持auto形参

~~~cpp
class PolyWidget {
public:
	template<typename T>
	void operator() (const T& param);
};

PolyWidget pw;
auto boundPW = std::bind(pw, _1);
boundPW(1930);		// 传递各种类型

// C++14
auto boundPW = [pw](const auto& param){ pw(param); }
~~~

<a name='42'></a>
### 42. 考虑emplacement代替插入
`emplace`拿着构造函数进行对象的构造，然后插入到容器中，避免了临时对象的构造和析构开销。
~~~cpp
template<class T, class Allocator = allocator<T>>
class vector {
public:
	void push_back(const T& x);
	void push_back(T&& x);
};

template <class... Args>
void emplace_back (Args&&... args);
~~~

对于资源管理类，采用传统的插入，`emplace`可能导致内存泄露。
