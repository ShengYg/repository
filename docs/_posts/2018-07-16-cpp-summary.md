---
layout: post
title:  "cpp python summary"
date:   2018-07-15 14:00:00 +0800
categories: [cpp]
tags: []
description: cpp和python总结，各种类型
---

- 目录
{:toc #markdown-toc}

## C++
### 基础
#### 引用与指针
引用：1、不是对象（变量别名，绑定其他对象）；2、需要初始化；3、绑定后不可改

#### const & static
- `const int* const p = 10;`，数据，指针
- `const int& f(const& int) const`，不能修改成员变量
- 静态局部变量：整个程序（与全局变量的区别：只对当前函数有效）
- 全局变量：整个程序，外部文件同`extern`访问
- 静态全局变量：当前cpp文件
- 静态成员变量：类内声明，类外定义
- 初始化：
    - static：类的外部，不能用构造函数
    - const：构造函数初始化列表
    - const static：类内直接初始化或类外初始化

#### 关键词
- `explicit`：避免隐式类型转换
- `volatile`：禁止编译器优化（编译器优化后，可能直接从cpu寄存器取值，声明`volatile`表示必须从内存中读取，避免操作系统修改值导致出错）
- `mutable`：1、允许修改const；2、lambda值捕获时不允许改变值，除非加`mutable`
- `final`：禁止类继承、虚函数重载
- `override`：强制虚函数重载
- `extern`：1、extern "C" 表示按C的规则编译函数；2、声明其他模块中的全局变量或函数
- `inline`：编译期将对函数的调用展开为函数本体。1、避免调用开销，可能使代码膨胀；2、在头文件内，修改后要重新编译；3、复杂函数不能用（构造、虚函数）

#### 强制类型转换
- const_cast：去除指针、引用的常量性
- static_cast：1、基本数据类型的转换，安全；2、类层次转换，没有动态类型检查，向下不安全；
- dynamic_cast：运行期类型检查，提供安全的向下转换，如果基类指针确实指向派生类，转换成功，否则返回空指针或异常`bad_cast`。基于RTTI实现，所以类需要有虚函数。
- reinterpret_cast：通常低层次指针类型的转化，如`void* -> int*`

### C++11相关
#### decltype
提供**返回值类型后置**的语法，用来声明**返回值类型依赖于参数类型的**函数模板。

#### constexpr
- 修饰对象：可以看做是const，唯一区别：const对象不一定是常量（可以是变量），不一定需要在编译期知道，而constexpr一定。
~~~cpp
int a = 10;
const int b = a;    // const变量
const int c = 10;   // const常量，等同于constexpr
~~~
- 修饰函数：如果所有参数的值在编译期可知，结果会在编译期得到，否则在运行期得到。可以看做两个函数，一个为编译期常量服务，一个为所有值服务。

#### emplace_back
- 实现：含通用引用类型的模板函数，能够判断类型是左值还是右值。如果是右值，直接在容器内部构造
- 好处：在容器中添加字符串常量时，1、调用`push_back`会构造临时变量，然后复制到容器中

#### lambda
- 包含**值捕获**和**引用捕获**，引用捕获可能导致引用悬挂的问题，因此尽量使用值捕获
- 只能捕获非静态局部变量（注意类成员的捕获），静态变量的值捕获类似引用捕获的特性
- 移动捕获（C++14）。`=`左边指定lambda类成员，`=`右边表示类成员初始化方法。
~~~cpp
auto pw = std::make_unique<Widget>();
auto func = [pw = std::move(pw)]{ return pw->isValid(); };  // C++14
auto func = std::bind(                                      // C++11 
    [](const std::unique_ptr<Widget>& pw){ return pw->isValid(); },
    std::make_unique<Widget>()
);
~~~

### 继承
#### Overload、Override、Overwrite
- overload：同一个类，函数名相同，参数不同，virtual可有可无
- override：基类与派生类，函数名相同，参数相同，virtual必须有
- overwrite：基类与派生类
    - 函数名相同，参数不同，virtual可有可无
    - 函数名相同，参数相同，virtual没有


#### 构造函数与虚函数
- 构造函数不能是虚函数：虚函数的类型在运行期确定，因此构造对象时无法确定其具体类型。
- 构造函数不能调用虚函数：语法上可以；但是，构造函数会调用基类的构造函数，进而始终调用基类的虚函数，达不到效果。
- 解决：工厂模式，为每种要创建的类型单独建一个factory，在factory中建立新对象。

#### 虚继承
- 无论虚基类在继承体系中出现多少次，派生类只包含一个虚基类子对象。
- 构造顺序：先构造虚基类，剩下的依次进行。
- 构造原则：虚基类由**最底层派生类**初始化。

### GCC编译链接
- 预处理：将源代码中的预编译指令进行替换，如`#include`，`#define`，`#ifdef`。
- 编译：进行语法检查，生成汇编代码
- 汇编：把汇编代码翻译成机器码，obj文件
- 链接：将目标文件、系统库文件链接，生成可执行文件
    - 静态链接：将代码从静态库中拷贝到可执行文件
    - 动态链接：将代码存到动态链接库中

### 内存
- 分类：堆、栈、自由存储区、全局/静态存储区、常量存储区
- 堆栈区别：大小、碎片、动/静、控制权
- malloc/new：异常、自由存储区、初始化、指定大小
- `new`过程：1、调用`operator new`申请内存。申请内存未能得到满足时，`operator new`会不断调用指定的错误处理函数称为`new-handle`（这个函数通过`std::set_new_handle`设置），直到找到足够的内存；内存依然不够，则会抛出异常`bad_alloc`；2、调用类的构造函数。如果构造失败，系统自动调用对应的`operator delete`将之前申请的内存释放。
- `placement new`：带有额外参数的new，通常用于在已开辟的内存上进行构造，参数为内存指针。注意：只有在`placement new`触发的构造函数导致异常时才会调用`placement delete`，其他时候只会调用正常的`operator delete`。
- 注意：1、构造函数抛出异常，指针没有赋值，无法调用析构函数，会造成内存泄漏，解决办法是类内指针使用智能指针；2、new的构造函数抛出异常，会调用`operator delete`释放申请的内存（只释放类，不会释放类内的指针指向的空间）。
- 限制对象内存
    - 只在堆：`protected`析构函数（方便继承），且额外需要一个伪析构函数。
    - 只在栈：`private void* operator new(...)`

### 智能指针（C++11）
- `shared_ptr`：多个指针可以同时指向一个对象，当最后一个`shared_ptr`离开作用域时，内存才会自动释放。
    - 构造函数是`explicit`，不支持隐式转换，需要直接初始化`shared_ptr<int> p(new int(1024));`
    - 使用`make_shared`
    - 循环引用：循环中有一个改成`weak_ptr`即可
    - 结构：指向数据的指针+指向控制块（引用计数，删除器等）的指针
    - 控制块相关问题
        - 控制块创建：`make_shared`，由`unique_ptr`构造，由原生指针构造
        - 问题：一个指针可能对应多个控制块，其中一个控制块引用计数为零就会产生析构，导致另一个指针未定义现象。
        - 解决：`class A: public std::enable_shared_from_this<A>`，`shared_from_this`方法检查当前智能指针的控制块并使用，如果没有则抛出异常。
- `unique_ptr`：
    - 独占语义，如果离开作用域或被重写，之前的资源将被释放。
    - 和原生指针大小相同，除非自定义删除器。
- `weak_ptr`：
    - 指向`shared_ptr`，但不改变引用计数
    - 不能直接访问对象，需要通过`lock()`返回`shared_ptr`，判断对象是否依然存在，并进一步访问。
    - `expired`判断对象是否被析构

### type_traits
概念：定义一些结构体或类，并提供模板类特化和偏特化版本，使用时会引发C++的函数重载机制或模板类匹配，优先匹配特化版本，实现同一种操作因类型不同而异的效果。
~~~cpp
template<typename _Tp> struct remove_const              // 泛化
{typedef _Tp   type;};
template<typename _Tp> struct remove_const<_Tp const>   // 特化
{typedef _Tp   type;};

//常量包装类型
template <class T, T v>
struct integral_constant
{
    static const T     value = v;
    typedef T          value_type;
    typedef integral_constant<T, v> type;
};

//定义true和false两个类型
typedef integral_constant<bool, true>  true_type;
typedef integral_constant<bool, false> false_type;

template<typename> //泛化
struct __is_void_helper : public false_type{};

template<>  //特化
struct __is_void_helper<void> : public true_type{};

//is_void
template<typename _Tp>
struct is_void : public __is_void_helper<typename remove_cv<_Tp>::type>::type{};
~~~




## python
### 装饰器
~~~python
def log(func):
    def wrapper(*args, **kw):
        print 'call %s():' % func.__name__
        return func(*args, **kw)
    return wrapper

@log
def g():    ## g(x) => log(g)(x)
	return 1
~~~

### 闭包
- 产生条件
    - 包含嵌套函数
    - 嵌套函数引用封闭函数中的值，即自由变量（代码块使用了一个变量，但该变量在代码块中没有被定义）
    - 封闭函数返回嵌套函数
~~~python
def outer(x):       ## 封闭函数
    def inner():    ## 嵌套函数，引用自由变量
        return x
    return inner
f = outer("hello")
del outer
f() # => "hello"    当outer生命周期结束，自由变量x依然存在
~~~
- 原理：每个函数对象有`__closure__`属性，如果函数是闭包，里面会存储自由变量。
- 作用：
    - 代替全局变量，提供某种形式的数据隐藏（代替简单的类）
    - 提供一致的函数签名

### 生成器
- 定义：生成器是一种迭代器，并且提供了**延迟操作**。生成器函数在每次暂停执行时，函数体内的所有变量都将被封存在生成器中，并将在恢复执行时还原。
- 生成器函数：由yield生成，不加return
~~~python
def gensquares(N):
    for i in range(N):
        yield i ** 2
f = gensquares(10)
f.next()
~~~
- 生成器表达式
~~~python
squares = (x**2 for x in range(5))
next(squares)
~~~
- 注意：**生成器只能遍历一次**


### 编码问题
- 三种编码
    - Unicode：一个符号集，包含任意字符，但它只规定了符号的二进制代码，却没有规定这个二进制代码应该如何存储
    - UTF-8：一种针对Unicode的可变长度字符编码（定长码），优先选用UTF-8编码
    - ascii
- 设置默认编码（python默认ascii）
~~~python
# coding:utf-8
~~~
- 编码转换
    - encode()：将unicode转为str
    - decode()：将str转为unicode
    - str.encode() => str.decode(sys.defaultencoding).encode()，sys.defaultencoding一般是ascii
- 注意：
    - str用引号直接定义，unicode用`u'...'`定义
    - 任何两种字符编码之间如果想完成转化，必须要通过unicode这个桥梁
    - unicode对象直接进行输出，往往会出现乱码，需要编码成str对象


### 执行过程
- 当python程序运行时，编译，结果保存在位于内存中的PyCodeObject中
- 当Python程序运行结束时，Python解释器则将PyCodeObject写回到pyc文件中
- 当python程序第二次运行时，首先程序会在硬盘中寻找pyc文件，如果找到，则直接载入，否则就重复上面的过程

可以认为pyc文件其实是PyCodeObject的一种持久化保存方式。

### 垃圾回收
**引用计数**为主，**标记-清除**和**分代回收**为辅的策略。
#### 引用计数
- 原理：python里每个对象都表示成一个结构体PyObject，PyObject中ob_refcnt就是做为引用计数。根据对象的引用或删除，ob_refcnt会增加减少。引用计数为0时，对象生命结束。
- 优点：简单；实时性，没有引用，内存实时释放
- 缺点：循环引用；维护计数消耗资源；释放一个大数据结构时，需要递归地减少。

#### 标记-清除
- 原理：1、标记，由对象之间的引用关系构成一个有向图。2、清除，由根对象出发，清除不可达对象。
- 优点：回收循环引用对象
- 缺点：扫描所有对象

#### 分代回收
- 原理：Python将内存对象根据存活时间分为3代，对应3个链表，越年轻垃圾回收频率越高。
- 步骤：
    - 创建对象时，Python会将其加入**零代链表**。
    - 当分配对象的计数值与释放对象的计数值超过阈值时，开始回收。
    - 循环遍历**零代列表**上的每个对象，检查列表中每个互相引用的对象，根据规则减掉其引用计数。
    - 清除有效计数为零的对象，剩下的活跃对象被移动到**一代链表**。
    - Python处理零代最为频繁，其次是一代然后才是二代。

### GIL（Global Interpreter Lock）
#### 概念
GIL是CPython中引入的一个概念。因为CPython的内存管理并非线程安全，为了避免多个操作系统原生线程一次性执行Python字节码，引入了GIL。当操作系统原生的线程在解释器解释执行任何Python代码时，都需要先获得这把锁才行，在遇到 I/O 操作时会释放这把锁。

#### 缺陷：基于pcode数量的调度方式
- 一种设计方法：Python的线程就是C语言的一个pthread，并通过操作系统调度算法进行调度。为了让各个线程能够平均利用CPU时间，python会计算当前已执行的微代码数量，达到一定阈值后就强制释放GIL，这时会触发一次操作系统的线程调度。
- 分析
    - 单CPU：没问题。因为只有释放了GIL才会引发线程调度，而任何一个线程被唤起时都能成功获得到GIL。
    - 多CPU：出现问题。主线程释放GIL引发线程调度，有一个线程会立刻获得GIL，而其他线程被唤醒后只能在此等待，浪费了时间。
- 结论：Python的多线程在多核CPU上，只对于IO密集型计算产生正面效果；而当至少有一个CPU密集型线程存在时，多线程效率会大幅下降。

#### 解决方案
- 使用multiprocessing创建进程池
- 使用其他C语言拓展模块，如numpy等等
- 使用C拓展编程，将计算密集型任务转移给C代码，并在C中释放GIL

### 线程与协程
- 定义：协程在子程序内部可中断，然后转而执行别的子程序，在适当的时候再返回来接着执行（如生成器yield）
- 调度：进程和线程完全由操作系统负责调度；协程则是在程序中自己负责调度。
- 运行效率：
    - 进程是重量级别的程序，创建和销毁开销大。
    - 线程是轻量级别的程序，相比进程下创建和销毁开销小，切换速度较快。
    - 协程则是单线程的异步编程模型。没有线程切换、保存上下文的开销的协程，相比下运行效率则更高；不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁。
- CPU利用
    - 进程可以充分利用多核CPU
    - 线程和协程由于CPython中全局解释器锁GIL的问题，只能使用到单核CPU的计算资源
