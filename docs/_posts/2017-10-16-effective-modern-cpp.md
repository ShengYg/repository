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












<a name='23'></a>
### 23. 理解std::move和std::forward
`std::move`不会移动任何东西，`std::forward`不会转发任何东西，在运行期间，它们什么事情都不会做。`std::move`和`std::forward`仅仅是表现为**转换类型的函数**（实际上是模板函数），`std::move`无条件地把参数转换为右值，而`std::forward`在满足条件下才会执行`std::move`的转换。

> `std::move`接收一个对象的引用（准确地说，是**通用引用**），然后返回相同对象的**右值引用**。

近似的实现方式如下：
~~~cpp
// C++11
template <typename T>
typename remove_reference<T>::type&& move(T&& param) {
	using ReturnType = typename remove_reference<T>::type&&;
	return static_cast<ReturnType>(param);
}

// C++14
template <typename T>
decltype(auto) move(T&& param) {
	using ReturnType = remove_reference_t<T>&&;
	return static_cast<ReturnType>(param);
}
~~~
注意：想移动对象时，不要声明为`const`

> `std::forward`仅当参数是用右值初始化时，才会把它转换为右值。具体如何操作见[条款28](#28)。

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
`T&&`由两种含义：
- 右值引用
- 通用引用：既可以绑定左值，也可以绑定右值

~~~cpp
// 右值引用
void f(Widget&& param);
Widget&& var1 = Widget();
template<typename T> void f(std::vector<T>&& param);

// 不是右值引用
auto&& var2 = var1;
template<typename T> void f(T&& param);
~~~

通用引用通常出现在含有类型推断的地方，常见的有**`auto&&`和模板**。如果初始值是个左值，通用引用相当于左值引用，如果初始值是个右值，通用引用相当于右值引用。

#### 1. 模板

> 注意：通用引用必须精确地定义为`T&&`且含有类型推断

~~~cpp
template<typename T>
void f(std::vector<T>&& param);	// param是vector&&类型，右值引用

template<typename T>
void f(const T&& param);    	// param是const类型，右值引用

template<class T, class Allocator = alloctor<T>>
class vector {
public:
    void push_back(T&& x);	// push_back是实例化vector的一部分，没有推断，右值引用
};

template<class T, class Allocator = allocator<T>>
class vector {
public:
    template <class... Args>
    void emplace_back(Args&&... args);	// 通用引用
};
~~~

#### 2. auto&&

`auto&&`的变量都是通用引用，在C++14中出现较多。

例子：`func`是个通用引用，可以绑定任何的可执行对象，而`params`是0个或多个通用引用，可以绑定任何数目个任意类型对象。最终结果是，auto通用引用使得可以记录**几乎所有**（见[条款30](#30)）的函数执行的所需时间。
~~~cpp
auto timeFuncInvocation = [](auto&& func, auto&&... params){
	start timer;
	std::forward<decltype(func)>(func)(
		std::forward<decltype(params)>(params);
	);
	// stop timer and record elapsed time;
};
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





