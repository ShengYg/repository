---
layout: post
title:  "C++ design patterns"
date:   2017-10-13 14:00:00 +0800
toc: true
categories: [cpp]
tags: [note]
description: C++ design patterns
---

## Table of Contents
- 创建型模式
  - [工厂模式](#factory)
  - [建造者模式](#builder)
  - [原型模式](#prototype)
  - [单例模式](#Singleton)
- 结构型模式
  - [适配器模式](#adapter)
  - [桥接模式](#bridge)
  - [组成模式](#composite)
  - [装饰模式](#decorator)
  - [外观模式](#facade)
  - [享元模式](#flyweight)
  - [代理模式](#proxy)
- 行为模式
  - [职责链](#chain)
  - [命令]
  - [解释器]
  - [迭代器]
  - [中介者](#mediator)
  - [备忘录](#memento)
  - [观察者](#observer)
  - [状态](#state)
  - [策略](#strategy)
  - [模板方法]
  - [访问者]


----
<a name='factory'></a>
## 抽象工厂模式 Abstract Factory
用处：创建不同的产品的不同系列，每个产品有相应的继承类（不同系列）。

缺点：难以添加新的产品种类

实现抽象工厂的有用技术：
1. 将具体工厂作为单例
2. 具体工厂生产具体产品时，为每个产品定义一个工厂方法（虚函数）
3. 抽象工厂不是抽象类，它只是所有抽象工厂方法的一个集合

<center>
<img src="{{ site.baseurl }}/assets/pic/factory.png" height="400px" >
</center>

~~~cpp
class SingleCore {
public:
    virtual void Show() = 0;
};
class SingleCoreA: public SingleCore {
public:
    void Show() { cout<<"SingleCore A"<<endl; }
};
class SingleCoreB: public SingleCore {
public:
    void Show() { cout<<"SingleCore B"<<endl; }
};

// MultiCore ...
class CoreFactory {
public:
    virtual SingleCore* CreateSingleCore();
    virtual MultiCore* CreateMultiCore();
};

class FactoryA :public CoreFactory {
public:
    SingleCore* CreateSingleCore() { return new SingleCoreA(); }
    MultiCore* CreateMultiCore() { return new MultiCoreA(); }
};

class FactoryB : public CoreFactory {
public:
    SingleCore* CreateSingleCore() { return new SingleCoreB(); }
    MultiCore* CreateMultiCore() { return new MultiCoreB(); }
};
~~~

<a name='builder'></a>
## 建造者模式
<center>
<img src="{{ site.baseurl }}/assets/pic/builder.png" height="400px" >
</center>
在导向者的控制下一步一步构造不同的产品。
与抽象工厂类似，不同点在于：
1. 抽象工厂中每个方法生成一个产品，具体调用时只需调用具体一个方法。建造者模式中每个方法是生产产品的**某一步骤**，生产产品时需要调用多个方法。
2. 抽象工厂的不同派生类表示同一产品的不同系列，建造者模式的派生类表示生产步骤互不相同的不同产品。

因此，建造者模式的特点在于它隐藏了建造某物品的**内部结构与步骤**。

> 建造者模式基类**不能是抽象基类**，因为其派生类可能只会利用其中一部分方法（生产产品时不需要所有步骤）。

~~~cpp
class Builder {
public:
    virtual void BuildHead() {}
    virtual void BuildBody() {}
}; 
class ThinBuilder : public Builder{
public:
    void BuildHead() { cout<<"build thin body"<<endl; }
    void BuildBody() { cout<<"build thin head"<<endl; }
};
 
class FatBuilder : public Builder {
public:
    void BuildHead() { cout<<"build fat body"<<endl; }
    void BuildBody() { cout<<"build fat head"<<endl; }
};
 
class Director {
private:
    Builder *m_pBuilder;
public:
    Director(Builder *builder) { m_pBuilder = builder; }
    void Create(){
        m_pBuilder->BuildHead();
        m_pBuilder->BuildBody();
    }
};
~~~

<a name='prototype'></a>
## 原型模式
当系统需要大量子类时，可以通过**拷贝**其中的一个实例来创建新的子类，称其为原型。原型需要有对应的`clone()`方法。

<center>
<img src="{{ site.baseurl }}/assets/pic/prototype.png" height="200px" >
</center>

~~~cpp
class Resume {
protected:
    char *name;
public:
    virtual Resume* Clone() { return NULL; }
};

class ResumeA : public Resume
{
public:
    ResumeA* Clone(){
        return new ResumeA(*this);
    }
};
~~~

<a name='Singleton'></a>
## 单例模式 Singleton
### 局部静态变量
线程安全
~~~cpp
class Singleton {
public:
    static Singleton& Instance() {
        static Singleton* instance_;
        if (instance_ == NULL) {
            instance_ = new Singleton();
        }
        return *instance_;
    }
private:
    Singleton(){}
    ~Singleton();
    Singleton(const Singleton&);
    Singleton& operator=(const Singleton&);
};

Singleton::Instance().dosth();
~~~
### Lazy Singleton
直到`Instance()`被访问，才会生成实例，这种特性被称为**延迟初始化**。不是线程安全的。
~~~cpp
class Singleton {
public:
    static Singleton& Instance() {
        if (instance_ == NULL) {
            instance_ = new Singleton;
        }
        return *instance_;
    }
private:
    Singleton();
    ~Singleton();
    Singleton(const Singleton&);
    Singleton& operator=(const Singleton&);
private:
    static Singleton* instance_;
};
Singleton* Singleton::instance_ = nullptr;	// 需要先初始化
Singleton::Instance();
~~~

### Eager Singleton
这种实现在程序开始前的时候就完成了实例的创建。线程安全。
~~~cpp
class Singleton {
public:
    static Singleton& Instance() {
        return *instance_;
    }
private:
    Singleton();
    ~Singleton();
    Singleton(const Singleton&);
    Singleton& operator=(const Singleton&);
private:
    static Singleton* instance_;
};
Singleton* Singleton::instance_ = new Singleton();	// 需要先初始化
Singleton::Instance();
~~~

### 双检测锁模式 Double-Checked Locking Pattern
Lazy Singleton的一种线程安全改造是在`Instance()`中每次判断是否为`nullptr`前加锁
~~~cpp
class Singleton {
public:
    static Singleton& Instance() {
        if(instance_ == nullptr){		// 当需要新建实例时，加锁
            lock_guard<mutex> lock(m_mutex);	// 加锁
            if(instance_ == nullptr) {
                instance_ = new Singleton();
            }
        }
        return *instance_;
    }
private:
    Singleton(){}
    ~Singleton();
    Singleton(const Singleton&);
    Singleton& operator=(const Singleton&);
    static Singleton* instance_;
    static mutex m_mutex;
};

Singleton* Singleton::instance_ = nullptr;
mutex Singleton::m_mutex;
~~~

<a name='adapter'></a>
## 适配器模式 Adapter
适用场景：想要用一个已知类，但它的接口和当前的类不匹配。举个例子，STL实现了一种数据结构，称为双端队列，支持前后两段的插入与删除。STL实现栈和队列时，没有从头开始定义它们，而是直接使用双端队列实现的。这里双端队列就扮演了适配器的角色。

分类：
- 类适配器：adapter公有继承target，私有继承adaptee
- 对象适配器：adapter公有继承target，并维护一个指向adaptee的成员

不同点：
- 类适配器：adapter可以重定义adaptee的某些方法
- 对象适配器：一个adapter可以适配多个adaptee（比如adaptee及其子类）

<center>
<img src="{{ site.baseurl }}/assets/pic/adapter.png" height="400px" >
</center>

~~~cpp
class Deque {
public:
    void push_back(int x) { cout<<"Deque push_back"<<endl; }
    void push_front(int x) { cout<<"Deque push_front"<<endl; }
    void pop_back() { cout<<"Deque pop_back"<<endl; }
    void pop_front() { cout<<"Deque pop_front"<<endl; }
};
 
class Sequence {
public:
    virtual void push(int x) = 0;
    virtual void pop() = 0;
};
 
class Stack: public Sequence {
public:
    void push(int x) { deque.push_back(x); }
    void pop() { deque.pop_back(); }
private:
    Deque deque; 
};
  
class Queue: public Sequence {
public:
    void push(int x) { deque.push_back(x); }
    void pop() { deque.pop_front(); }
private:
    Deque deque;
};
~~~

<a name='bridge'></a>
## 桥接模式 Bridge
抽象类定义了一系列接口，并保存实现类的指针，其派生类可以扩展接口；实现类包含具体的实现方法。

抽象类需要获得正确的实现类引用（基类还是派生类），可以通过一个单例的工厂模式实现。

考虑装操作系统，有多种配置的计算机，同样也有多款操作系统。可以将操作系统和计算机分别抽象出来，让它们各自发展，减少它们的耦合度。
<center>
<img src="{{ site.baseurl }}/assets/pic/bridge.png" height="200px" >
</center>

~~~cpp
class OS {		// 实现类
public:
    virtual void InstallOS_Imp() {}
};
class WindowOS: public OS {
public:
    void InstallOS_Imp() { cout<<"安装Window操作系统"<<endl; }
};
class LinuxOS: public OS {
public:
    void InstallOS_Imp() { cout<<"安装Linux操作系统"<<endl; }
};
class UnixOS: public OS {
public:
    void InstallOS_Imp() { cout<<"安装Unix操作系统"<<endl; }
};

class Computer {	// 抽象类
public:
    virtual void InstallOS(OS *os) {}
};
class DellComputer: public Computer {
public:
    void InstallOS(OS *os) { os->InstallOS_Imp(); }
};
class AppleComputer: public Computer {
public:
    void InstallOS(OS *os) { os->InstallOS_Imp(); }
};
class HPComputer: public Computer {
public:
    void InstallOS(OS *os) { os->InstallOS_Imp(); }
};
~~~

<a name='composite'></a>
## 组成模式 Composite
将对象组合成**树形**结构以表示“部分-整体”的层次结构。组合使得用户对单个对象和组合对象的使用具有一致性。这种树形结构在现实生活中随处可见，比如一个集团公司，它有一个母公司，下设很多家子公司。不管是母公司还是子公司，都有各自直属的财务部、人力资源部、销售部等。对于母公司来说，不论是子公司，还是直属的财务部、人力资源部，都是它的部门。整个公司的部门拓扑图就是一个树形结构。
<center>
<img src="{{ site.baseurl }}/assets/pic/composite.png" height="300px" >
</center>

~~~cpp
class Company {
public:
    Company(string name) { m_name = name; }
    virtual ~Company(){}
    virtual void Add(Company *pCom){}
    virtual void Show(int depth) {}
protected:
    string m_name;
};

class ConcreteCompany : public Company {
public:
    ConcreteCompany(string name): Company(name) {}
    virtual ~ConcreteCompany() {}
    void Add(Company *pCom) { m_listCompany.push_back(pCom); }
    void Show(int depth) {
        for(int i = 0;i < depth; i++)
            cout<<"-";
        cout<<m_name<<endl;
        vector<Company *>::iterator iter=m_listCompany.begin();
        for(; iter != m_listCompany.end(); iter++)
            (*iter)->Show(depth + 2);
    }
private:
    vector<Company *> m_listCompany;
};

class FinanceDepartment : public Company {
public:
    FinanceDepartment(string name):Company(name){}
    virtual ~FinanceDepartment() {}
    virtual void Show(int depth)
    {
        for(int i = 0; i < depth; i++)
            cout<<"-";
        cout<<m_name<<endl;
    }
};

class HRDepartment :public Company {
public:
    HRDepartment(string name):Company(name){}
    virtual ~HRDepartment() {}
    virtual void Show(int depth)
    {
        for(int i = 0; i < depth; i++)
            cout<<"-";
        cout<<m_name<<endl;
    }
};
~~~

<a name='decorator'></a>
## 装饰模式 Decorator
动态地给一个对象添加一些**额外的职责**。就增加功能来说，装饰模式相比生成子类更为灵活。有时我们**希望给某个对象而不是整个类添加一些功能**。比如有一个手机，允许你为手机添加特性，比如增加挂件、屏幕贴膜等。一种灵活的设计方式是，将手机嵌入到另一对象中，由这个对象完成特性的添加，我们称这个嵌入的对象为装饰。这个装饰与它所装饰的**组件接口一致**，因此它对使用该组件的客户透明。

构成：
- component定义了相关操作
- decorator继承自component，并保留了一个指向component的指针。
- decorator的方法对具体的某个component进行操作。

注意：
- 没有必要设计抽象的decorator类
- 当component很大时，采用decorator代价高，可以改成策略模式。装饰器对某个对象进行包装，策略模式进行内核上的改变

<center>
<img src="{{ site.baseurl }}/assets/pic/decorator.png" height="400px" >
</center>

~~~cpp
class Phone {
public:
    Phone() {}
    virtual ~Phone() {}
    virtual void ShowDecorate() {}
};
class iPhone : public Phone {
private:
    string m_name;
public:
    iPhone(string name): m_name(name){}
    ~iPhone() {}
    void ShowDecorate() { cout<<m_name<<"的装饰"<<endl;}
};
class NokiaPhone : public Phone {
private:
    string m_name;
public:
    NokiaPhone(string name): m_name(name){}
    ~NokiaPhone() {}
    void ShowDecorate() { cout<<m_name<<"的装饰"<<endl;}
};

class DecoratorPhone : public Phone {
private:
    Phone *m_phone;
public:
    DecoratorPhone(Phone *phone): m_phone(phone) {}
    virtual void ShowDecorate() { m_phone->ShowDecorate(); }
};
class DecoratorPhoneA : public DecoratorPhone {
public:
    DecoratorPhoneA(Phone *phone) : DecoratorPhone(phone) {}
    void ShowDecorate() { DecoratorPhone::ShowDecorate(); AddDecorate(); }
private:
    void AddDecorate() { cout<<"增加挂件"<<endl; }
};
class DecoratorPhoneB : public DecoratorPhone {
public:
    DecoratorPhoneB(Phone *phone) : DecoratorPhone(phone) {}
    void ShowDecorate() { DecoratorPhone::ShowDecorate(); AddDecorate(); }
private:
    void AddDecorate() { cout<<"屏幕贴膜"<<endl; }
};
~~~

<a name='facade'></a>
## 外观模式 Facade
系统提供给客户的是一个**简单的对外接口**，而把里面复杂的结构都封装了起来。客户只需使用这些简单接口就能使用这个系统，而不需要关注内部复杂的结构。通常为**单例模式**。
<center>
<img src="{{ site.baseurl }}/assets/pic/facade.png" height="200px" >
</center>

~~~cpp
class Scanner {
public:
    void Scan() { cout<<"词法分析"<<endl; }
};
class Parser {
public:
    void Parse() { cout<<"语法分析"<<endl; }
};
class GenMidCode {
public:
    void GenCode() { cout<<"产生中间代码"<<endl; }
};
class GenMachineCode {
public:
    void GenCode() { cout<<"产生机器码"<<endl;}
};

class Compiler {
public:
    void Run()
    {
        Scanner scanner;
        Parser parser;
        GenMidCode genMidCode;
        GenMachineCode genMacCode;
        scanner.Scan();
        parser.Parse();
        genMidCode.GenCode();
        genMacCode.GenCode();
    }
};
~~~

<a name='flyweight'></a>
## 享元模式 Flyweight
运用共享技术有效地支持大量细粒度的对象。在围棋中，棋子就是大量细粒度的对象。其属性有内在的，比如颜色、形状等，也有外在的，比如在棋盘上的位置。内在的属性是可以共享的，区分在于外在属性。因此，可以这样设计，只需定义两个棋子的对象，一颗黑棋和一颗白棋，这两个对象含棋子的内在属性；棋子的外在属性，即在棋盘上的位置可以提取出来，存放在单独的容器中。相比之前的方案，现在容器中仅仅存放了位置属性，而原来则是棋子对象。显然，现在的方案大大减少了对于空间的需求。

<center>
<img src="{{ site.baseurl }}/assets/pic/flyweight.png" height="300px" >
</center>

~~~cpp
// 棋子定义
enum PieceColor {BLACK, WHITE};
struct PiecePos {
    int x;
    int y;
    PiecePos(int a, int b): x(a), y(b) {}
};
class Piece {
protected:
    PieceColor m_color;
public:
    Piece(PieceColor color): m_color(color) {}
    ~Piece() {}
    virtual void Draw() {}
};
class BlackPiece: public Piece {
public:
    BlackPiece(PieceColor color): Piece(color) {}
    ~BlackPiece() {}
    void Draw() { cout<<"绘制一颗黑棋\n"; }
};
class WhitePiece: public Piece {
public:
    WhitePiece(PieceColor color): Piece(color) {}
    ~WhitePiece() {}
    void Draw() { cout<<"绘制一颗白棋\n";}
};

// 棋盘定义
class PieceBoard {
private:
    vector<PiecePos> m_vecPos; //存放棋子的位置
    Piece *m_blackPiece;
    Piece *m_whitePiece;
    string m_blackName;
    string m_whiteName;
public:
    PieceBoard(string black, string white): m_blackName(black), m_whiteName(white) {
        m_blackPiece = NULL;
        m_whitePiece = NULL;
    }
    ~PieceBoard() { delete m_blackPiece; delete m_whitePiece;}
    void SetPiece(PieceColor color, PiecePos pos) {
        if(color == BLACK) {
            if(m_blackPiece == NULL)
                m_blackPiece = new BlackPiece(color);
            cout<<m_blackName<<"在位置("<<pos.x<<','<<pos.y<<")";
            m_blackPiece->Draw();
        }
        else {
            if(m_whitePiece == NULL)
                m_whitePiece = new WhitePiece(color);
            cout<<m_whiteName<<"在位置("<<pos.x<<','<<pos.y<<")";
            m_whitePiece->Draw();
        }
        m_vecPos.push_back(pos);
    }
};
~~~

<a name='proxy'></a>
## 代理模式 Proxy
为其他对象提供一种代理以**控制访问对象**，原因是有的对象使用开销大，只有在确实需要时才对它进行访问。

有四种常用的情况：
1. 远程代理：为同一个对象在不同地址空间提供局部代表
2. 虚代理：根据需要创建开销很大的对象
3. 保护代理：处理有不同访问权限的对象
4. 智能引用

例子： 考虑一个可以在文档中嵌入图形对象的文档编辑器。有些图形对象的创建开销很大。这里就可以运用代理模式，在打开文档时，并不打开图形对象，而是打开图形对象的代理以替代真实的图形。待到真正需要打开图形时，仍由代理负责打开。

注意：
- proxy与subject**接口相同**（可以让它们同时继承一个基类），且保存一个指向subject的指针。
- 重载->操作符，使得proxy能够直接调用subject的方法，让proxy表现的像一个指针
- proxy可以通过接口处理所有subject类，而不需为每个subject定义一个对应的proxy，除非想用proxy实例化subject。

<center>
<img src="{{ site.baseurl }}/assets/pic/proxy.png" height="300px" >
</center>

~~~cpp
class Image {
public:
    Image(string name): m_imageName(name) {}
    virtual ~Image() {}
    virtual void Show() {}
protected:
    string m_imageName;
};
class BigImage: public Image {
public:
    BigImage(string name):Image(name) {}
    ~BigImage() {}
    void Show() { cout<<"Show big image : "<<m_imageName<<endl; }
};
class BigImageProxy: public Image {
private:
    BigImage *m_bigImage;
public:
    BigImageProxy(string name):Image(name),m_bigImage(0) {}
    ~BigImageProxy() { delete m_bigImage; }
    void Show() {
        if(m_bigImage == NULL)
            m_bigImage = new BigImage(m_imageName);
        m_bigImage->Show();
    }
};
~~~

<a name='chain'></a>
## 职责链模式 chain of responsibility
适用场景：向多个对象发出请求，并且不知道该哪个对象处理时，将这些对象**连成一条链**，并沿着这条链传递该请求，直到有一个对象处理它为止。例子：考虑员工要求加薪，公司的管理者一共有三级，总经理、总监、经理，如果一个员工要求加薪，应该向主管的经理申请，如果加薪的数量在经理的职权内，那么经理可以直接批准，否则将申请上交给总监。总监的处理方式也一样，总经理可以处理所有请求。这就是典型的职责链模式，请求的处理形成了一条链，直到有一个对象处理请求。

<center>
<img src="{{ site.baseurl }}/assets/pic/chain.png" height="300px" >
</center>

~~~cpp
class Manager {
protected:
    Manager *m_manager;
    string m_name;
public:
    Manager(Manager *manager, string name):m_manager(manager), m_name(name){}
    virtual void DealWithRequest(string name, int num)  {}
};

class CommonManager: public Manager {
public:
    CommonManager(Manager *manager, string name):Manager(manager,name) {}
    void DealWithRequest(string name, int num) {
        if(num < 500){
            cout<<"经理"<<m_name<<"批准"<<name<<"加薪"<<num<<"元"<<endl<<endl;
        }else {
            cout<<"经理"<<m_name<<"无法处理，交由总监处理"<<endl;
            m_manager->DealWithRequest(name, num);
        }
    }
};
class Majordomo: public Manager {
public:
    Majordomo(Manager *manager, string name):Manager(manager,name) {}
    void DealWithRequest(string name, int num) {
        if(num < 1000){
            cout<<"总监"<<m_name<<"批准"<<name<<"加薪"<<num<<"元"<<endl<<endl;
        }else{
            cout<<"总监"<<m_name<<"无法处理，交由总经理处理"<<endl;
            m_manager->DealWithRequest(name, num);
        }
    }
};
class GeneralManager: public Manager {
public:
    GeneralManager(Manager *manager, string name):Manager(manager,name) {}
    void DealWithRequest(string name, int num){
        cout<<"总经理"<<m_name<<"批准"<<name<<"加薪"<<num<<"元"<<endl<<endl;
    }
};

int main(){
    Manager *general = new GeneralManager(NULL, "A");
    Manager *majordomo = new Majordomo(general, "B");
    Manager *common = new CommonManager(majordomo, "C");
    common->DealWithRequest("D",300);			//员工D要求加薪
    common->DealWithRequest("E", 600);
    common->DealWithRequest("F", 1000);
    delete common; delete majordomo; delete general;
    return 0;
}
~~~

<a name='mediator'></a>
## 中介者模式 mediator
用一个中介对象来封装一系列的对象交互。中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。例子：房东和房客保留了中介的指针，当他们需要交互时，向中介发送请求。中介保留了房东和房客的指针，处理他们之间的信息交互。

实现：
- 忽略抽象中介类
- 将中介实现成**观察者模式**，一旦subject状态发生变化就通知中介

<center>
<img src="{{ site.baseurl }}/assets/pic/mediator.png" height="300px" >
</center>

~~~cpp
class Mediator;

class Person {
protected:
    Mediator *m_mediator; //中介
public:
    virtual void SetMediator(Mediator *mediator){} //设置中介
    virtual void SendMessage(string message) {}    //向中介发送信息
    virtual void GetMessage(string message) {}     //从中介获取信息
};
class Renter: public Person {
public:
    void SetMediator(Mediator *mediator) { m_mediator = mediator; }
    void SendMessage(string message) { m_mediator->Send(message, this); }
    void GetMessage(string message) { cout<<"租房者收到信息"<<message; }
};
class Landlord: public Person {
public:
    void SetMediator(Mediator *mediator) { m_mediator = mediator; }
    void SendMessage(string message) { m_mediator->Send(message, this); }
    void GetMessage(string message) { cout<<"房东收到信息："<<message; }
};

class Mediator {
public:
    virtual void Send(string message, Person *person) {}
    virtual void SetA(Person *A) {}
    virtual void SetB(Person *B) {}
};
class HouseMediator : public Mediator {
private:
    Person *m_A; //租房者
    Person *m_B; //房东
public:
    HouseMediator(): m_A(0), m_B(0) {}
    void SetA(Person *A) { m_A = A; }
    void SetB(Person *B) { m_B = B; }
    void Send(string message, Person *person) {
        if(person == m_A) //租房者给房东发信息
            m_B->GetMessage(message); //房东收到信息
        else
            m_A->GetMessage(message);
    }
};
int main() {
    Mediator *mediator = new HouseMediator();
    Person *person1 = new Renter();
    Person *person2 = new Landlord();
    mediator->SetA(person1);
    mediator->SetB(person2);
    person1->SetMediator(mediator);
    person2->SetMediator(mediator);
    person1->SendMessage("我想在南京路附近租套房子，价格800元一个月\n");
    person2->SendMessage("出租房子：南京路100号，70平米，1000元一个月\n");
    delete person1; delete person2; delete mediator;
    return 0;
} 
~~~

<a name='memento'></a>
## 备忘录模式 memento
在不破坏封装性的前提下，捕获一个对象的**内部状态**，并在该对象之外保存这个状态。这样以后就可将该对象恢复到原先保存的状态。例如，游戏中的保存进度。

<center>
<img src="{{ site.baseurl }}/assets/pic/memento.png" height="200px" >
</center>

~~~cpp
class Memento {
public:
    int m_vitality; //生命值
    int m_attack;   //进攻值
    int m_defense;  //防守值
public:
    Memento(int vitality, int attack, int defense):
            m_vitality(vitality),m_attack(attack),m_defense(defense){}
    Memento& operator=(const Memento &memento) {
        m_vitality = memento.m_vitality;
        m_attack = memento.m_attack;
        m_defense = memento.m_defense;
        return *this;
    }
};

class GameRole {
private:
    int m_vitality;
    int m_attack;
    int m_defense;
public:
    GameRole(): m_vitality(100),m_attack(100),m_defense(100) {}
    Memento Save(){
        Memento memento(m_vitality, m_attack, m_defense);
        return memento;
    }
    void Load(Memento memento){
        m_vitality = memento.m_vitality;
        m_attack = memento.m_attack;
        m_defense = memento.m_defense;
    }
    void Show() { cout<<"vitality : "<< m_vitality<<", attack : "<< m_attack<<", defense : "<< m_defense<<endl; }
    void Attack() { m_vitality -= 10; m_attack -= 10;  m_defense -= 10; }
};

class Caretake {
public:
    Caretake() {}
    void Save(Memento menento) { m_vecMemento.push_back(menento); }
    Memento Load(int state) { return m_vecMemento[state]; }
private:
    vector<Memento> m_vecMemento;
};
~~~

<a name='observer'></a>
## 观察者模式 observer
定义对象间的一种**一对多的依赖关系**，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。它还有两个别名，依赖(Dependents)，发布-订阅(Publish-Subsrcibe)。举个博客订阅的例子，当博主发表新文章的时候，即博主状态发生了改变，那些订阅的读者就会收到通知，然后进行相应的动作，比如去看文章，或者收藏起来。可以建一个`Blog`类，里面存储`Observer`的链表。

<center>
<img src="{{ site.baseurl }}/assets/pic/observer.png" height="400px" >
</center>

~~~cpp
class Observer {
public:
    Observer() {}
    virtual ~Observer() {}
    virtual void Update() {}
};

class Blog {
public:
    Blog() {}
    virtual ~Blog() {}
    void Attach(Observer *observer) { m_observers.push_back(observer); }	//添加观察者
    void Remove(Observer *observer) { m_observers.pop_back(); } 		//移除观察者
    void Notify(){  	//通知观察者
        vector<Observer*>::iterator iter = m_observers.begin();
        for(; iter != m_observers.end(); iter++)
            (*iter)->Update();
    }
    virtual void SetStatus(string s) { m_status = s; } 	//设置状态
    virtual string GetStatus() { return m_status; }    	//获得状态
private:
    vector<Observer* > m_observers;	//观察者链表
protected:
    string m_status;
};


class BlogCSDN : public Blog {
private:
    string m_name;	//博主名称
public:
    BlogCSDN(string name): m_name(name) {}
    ~BlogCSDN() {}
    void SetStatus(string s) { m_status = "CSDN通知 : " + m_name + s; } //具体设置状态信息
    string GetStatus() { return m_status; }
};

class ObserverBlog : public Observer {
private:
    string m_name;	//观察者名称
    Blog *m_blog;	//观察的博客，当然以链表形式更好，就可以观察多个博客
public:
    ObserverBlog(string name,Blog *blog): m_name(name), m_blog(blog) {}
    ~ObserverBlog() {}
    void Update() {	//获得更新状态
        string status = m_blog->GetStatus();
        cout<<m_name<<"-------"<<status<<endl;
    }
};
~~~

<a name='state'></a>
## 状态模式 state
允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。它有两种使用情况：（1）一个对象的行为取决于它的状态, 并且它必须在运行时刻根据状态改变它的行为。（2）一个操作中含有庞大的多分支的条件语句，且这些分支依赖于该对象的状态。以战争为例，假设一场战争需经历四个阶段：前期、中期、后期、结束。当战争处于不同的阶段，战争的行为是不一样的，也就说战争的行为取决于所处的阶段，而且随着时间的推进是动态变化的。

<center>
<img src="{{ site.baseurl }}/assets/pic/state.png" height="300px" >
</center>

~~~cpp
class War;
class State {
public:
    virtual void Prophase() {}
    virtual void Metaphase() {}
    virtual void Anaphase() {}
    virtual void End() {}
    virtual void CurrentState(War *war) {}
};

class War {
private:
    State *m_state;  	//目前状态
    int m_days;      	//战争持续时间
public:
    War(State *state): m_state(state), m_days(0) {}
    ~War() { delete m_state; }
    int GetDays() { return m_days; }
    void SetDays(int days) { m_days = days; }
    void SetState(State *state) { delete m_state; m_state = state; }
    void GetState() { m_state->CurrentState(this); }
};

class EndState: public State {
public:
    void End(War *war) {
        cout<<"战争结束"<<endl;
    }
    void CurrentState(War *war) { End(war); }
};
class AnaphaseState: public State {
public:
    void Anaphase(War *war) {
        if(war->GetDays() < 30)
            cout<<"第"<<war->GetDays()<<"天：战争后期，双方拼死一搏"<<endl;
        else {
            war->SetState(new EndState());
            war->GetState();
        }
    }
    void CurrentState(War *war) { Anaphase(war); }
};
class MetaphaseState: public State {
public:
    void Metaphase(War *war) {
        if(war->GetDays() < 20)
            cout<<"第"<<war->GetDays()<<"天：战争中期，进入相持阶段，双发各有损耗"<<endl;
        else {
            war->SetState(new AnaphaseState());
            war->GetState();
        }
    }
    void CurrentState(War *war) { Metaphase(war); }
};
class ProphaseState: public State {
public:
    void Prophase(War *war){
        if(war->GetDays() < 10)
            cout<<"第"<<war->GetDays()<<"天：战争初期，双方你来我往，互相试探对方"<<endl;
        else {
            war->SetState(new MetaphaseState());
            war->GetState();
        }
    }
    void CurrentState(War *war) { Prophase(war); }
};
~~~

<a name='strategy'></a>
## 策略模式 strategy
策略模式是指定义一系列的算法，把它们一个个封装起来，并且使它们可相互替换。本模式使得算法可独立于使用它的客户而变化。也就是说这些算法所完成的功能一样，对外的接口一样，只是各自实现上存在差异。用策略模式来封装算法，效果比较好。例子：高速缓存。

<center>
<img src="{{ site.baseurl }}/assets/pic/strategy.png" height="200px" >
</center>

~~~cpp
class ReplaceAlgorithm {
public:
    virtual void Replace() = 0;
};
class LRU_ReplaceAlgorithm : public ReplaceAlgorithm {
public:
    void Replace() { cout<<"Least Recently Used replace algorithm"<<endl; }
};
class FIFO_ReplaceAlgorithm : public ReplaceAlgorithm {
public:
    void Replace() { cout<<"First in First out replace algorithm"<<endl; }
};
class Random_ReplaceAlgorithm: public ReplaceAlgorithm {
public:
    void Replace() { cout<<"Random replace algorithm"<<endl; }
};

// 传入一个特定算法的指针
class Cache {
private:
    ReplaceAlgorithm *m_ra;
public:
    Cache(ReplaceAlgorithm *ra) { m_ra = ra; }
    ~Cache() { delete m_ra; }
    void Replace() { m_ra->Replace(); }
};

// 传入一个标签
enum RA {LRU, FIFO, RANDOM};
class Cache {
private:
    ReplaceAlgorithm *m_ra;
public:
    Cache(enum RA ra)
    {
        if(ra == LRU)
            m_ra = new LRU_ReplaceAlgorithm();
        else if(ra == FIFO)
            m_ra = new FIFO_ReplaceAlgorithm();
        else if(ra == RANDOM)
            m_ra = new Random_ReplaceAlgorithm();
        else
            m_ra = NULL;
    }
    ~Cache() { delete m_ra; }
    void Replace() { m_ra->Replace(); }
};

// 传入一个模板实参
template <class RA>
class Cache {
private:
    RA m_ra;
public:
    Cache() { }
    ~Cache() { }
    void Replace() { m_ra.Replace(); }
};
~~~









