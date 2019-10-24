---
layout: post
title:  "template metaprogramming"
date:   2019-08-04 14:00:00 +0800
toc: true
categories: [cpp]
tags: [note]
description: template metaprogramming
---

- 目录
{:toc #markdown-toc}

## 高阶元函数
例子：创建`f(f(x))`
~~~cpp
// method 1
template <class F, class X>
struct twice{
    typedef typename F::template apply<X>::type once;
    typedef typename F::template apply<once>::type once;
};

// method 2: 元函数转发
template <class F, class X>
struct twice : F::template apply<typename F::template apply<X>::type>
{ };

// method 3: 分解元函数转发
template <class UnaryMeatFunctionClass, class Arg>
struct apply1: UnaryMeatFunctionClass::template apply<Arg>
{};

template <class F, class X>
struct twice: apply1<F, typename apply1<F,X>::type>
{};
~~~

## Polymorphism
### static Polymorphism
~~~cpp
// dynamic
void mydraw(GeoObj const& obj){ //GeoObj is abstract class
    obj.draw();
}

// static
template<typename GeoObj>
void mydraw(GeoObj const& obj){ //GeoObj is template parameter
    obj.draw();
}
~~~

**Constraint**: all types must be determined at compile time.

### Concept
"**Concept**" will be part of the next standard after C++17. It denotes **a set of constraits** that template arguments have to fulfill to successfully instantiate a template.

~~~cpp
template<typename T>
concept GeoObj = requires(T x){
    {x.draw()} -> void;
    {x.center_of_gravity()} -> Coord;
}

template<typename T>
require GeoObj<T>
void mydraw(GeoObj const& obj){
    obj.draw();
}
~~~

## Overloading on Type Properties
We want to overload function templates based on types, code below is a wrong way:

~~~cpp
template<typename Number> void f(Number);
template<typename Container> void f(Container);
~~~

Function templates that differ **only based on their template parameter names** are not overloadable.

### Algorithm Specialization
It is useful in many cases.

~~~cpp
template<typename T>
void swap(T& x, T& y);

template<typename T>
void swap(Array<T>& x, Array<T>& y);
~~~

### Tag Dispatching
It works well when there is a natural hierarchical structure and an existing set of traits. For example, Iterator.

~~~cpp
// definition of different tags
struct input_iterator_tag { };
struct forward_iterator_tag : public input_iterator_tag { };

// overload 2 functions based on "tag"
template<typename Iterator, typename Distance>
void advanceIterImpl(Iterator& x, Distance n, std::input_iterator_tag);
template<typename Iterator, typename Distance>
void advanceIterImpl(Iterator& x, Distance n, std::random_access_iterator_tag)

// use traits to access "tag" info
template<typename Iterator, typename Distance>
void advanceIter(Iterator& x, Distance n) {
    advanceIterImpl(x, n, typename std::iterator_traits<Iterator>::iterator_category())
}
~~~

### Enable/Disable Function Template
**EnableIf**(alias to `std::enable_if`): When the condition is true, return template T. Otherwise, **no valid type** will be produced.

~~~cpp
template<bool, typename T = void>
struct EnableIfT { };

template<typename T>
struct EnableIfT<true, T> {
    using Type = T;
};

template<bool Cond, typename T = void>
using EnableIf = typename EnableIfT<Cond, T>::Type;
~~~

We use EnableIf to implement advanceIter. It will return void when the argument is a random access iterator, and will be removed from consideration otherwise.

~~~cpp
template<typename Iterator>
constexpr bool IsRandomAccessIterator = IsConvertible<
    typename std::iterator_traits<Iterator>::iterator_category,
    std::random_access_iterator_tag>;

// enable it    
template<typename Iterator, typename Distance>
EnableIf<IsRandomAccessIterator<Iterator>>
advanceIter(Iterator& x, Distance n){
    x += n; // constant time
}

// disable it
template<typename Iterator, typename Distance>
EnableIf<!IsRandomAccessIterator<Iterator>>
advanceIter(Iterator& x, Distance n) {
    while (n > 0) {//linear time
        ++x;
        --n;
    }
}
~~~

#### In Constructor Templates
EnableIf is typically used in the **return type of the function template**, it does not work for **constructor templates** or **conversion function templates**, because neither has a specified return type.

~~~cpp
template<typename T>
class Container {
public:
    template<typename Iterator, 
            typename = EnableIf<IsInputIterator<Iterator>>>
    Container(Iterator first, Iterator last);
    template<typename Iterator, 
            typename = EnableIf<IsRandomAccessIterator<Iterator>>, 
            typename = int> // extra dummy parameter is used to overload
    Container(Iterator first, Iterator last);

convertible:
    template<typename U, typename = EnableIf<IsConvertible<T, U>>>
    operator Container<U>() const;
};
~~~

#### if-constexpr in C++17
`if-else` can be used in compiler time in C++17 with `if constexpr()`, so it can be used in template.

~~~cpp
template<typename Iterator, typename Distance>
void advanceIter(Iterator& x, Distance n) {
    if constexpr(IsRandomAccessIterator<Iterator>) {
        x += n;
    }else if constexpr(IsBidirectionalIterator<Iterator>) {
        ....
    }
}
~~~

**Downsides**: It doesn’t work well with SFINAE. Invalid f<T>() is not removed from the candidates list and therefore may inhibit another overload resolution outcome. Use `enable_if` in such case.

#### Concepts
`requires` clause describes the requirements of the template. If any of the requirements are not satisfied, the template is not considered a candidate. 

~~~cpp
template<typename T>
class Container {
public:
    template<typename Iterator>
    requires IsInputIterator<Iterator>
    Container(Iterator first, Iterator last);
    
    template<typename Iterator>
    requires IsRandomAccessIterator<Iterator>
    Container(Iterator first, Iterator last);
};
~~~

### Class Specialization
#### Enable/Disable
Different class templates can be implemented by using enabled/disabled partial specializations of class templates. We use an unnamed parameter as an anchor for EnableIf.

~~~cpp
template<typename Key, typename Value, typename = void> // unname parameter
class Dictionary
{
    //all vector implementation
};

template<typename Key, typename Value>
class Dictionary<Key, Value, EnableIf<HasLess<Key>>>  // specialized template
~~~

#### Tag Dispatching
It is similar to tag dispatching for functions. MatchOverloads(emm...diffucult to understand) template uses recursive inheritance to declare a match() function with each type.

~~~cpp
template<typename Iterator,
        typename Tag =
            BestMatchInSet<
                typename std::iterator_traits<Iterator>::iterator_category,
                std::input_iterator_tag,
                std::bidirectional_iterator_tag>>
class Advance;

template<typename Iterator>
class Advance<Iterator, std::input_iterator_tag>
template<typename Iterator>
class Advance<Iterator, std::bidirectional_iterator_tag>

template<typename... Types>
struct MatchOverloads;
template<>
struct MatchOverloads<> {
    static void match(...);
};
template<typename T1, typename... Rest>
struct MatchOverloads<T1, Rest...> : public MatchOverloads<Rest...> {
    static T1 match(T1); // introduce overload for T1
    using MatchOverloads<Rest...>::match;
};

template<typename T, typename... Types>
struct BestMatchInSetT {
using Type = decltype(MatchOverloads<Types...>::match(declval<T>()));
};
template<typename T, typename... Types>
using BestMatchInSet = typename BestMatchInSetT<T, Types...>::Type;
~~~

### Instantiation-Safe Templates

~~~cpp
// define LessResult here

template<typename T>
EnableIf<IsConvertible<LessResult<T const&, T const&>, bool>, T const&>
T const& min(T const& x, T const& y)
{
    if (y < x) {    // it will check operator<
        return y;
    }
    return x;
}
~~~

## Templates and Inheritance
### The Empty Base Class Optimization (EBCO)

~~~cpp
class Empty { };                    // sizeof(Empty)=1
class EmptyToo : public Empty { };  // sizeof(EmptyToo)=1
class EmptyThree : public Empty, 
                   public EmptyToo { };  // sizeof(EmptyToo)=2
~~~

#### Members as Base Classes
- A template parameter is known to be substituted **by class types only**.
- Another member of the class template is available.

~~~cpp
// original definition
template<typename CustomClass>
class Optimizable {
private:
    CustomClass info; // might be empty
    void* storage;
};

// replace member by base class
template<typename CustomClass>
class Optimizable {
private:
    BaseMemberPair<CustomClass, void*> info_and_storage;
};

// defined in basememberpair.hpp
template<typename Base, typename Member>
class BaseMemberPair : private Base {
private:
    Member mem;
public:
    BaseMemberPair (Base const & b, Member const & m) : Base(b), mem(m) { }
    Base const& base() const {
        return static_cast<Base const&>(*this);
    }
~~~

### The Curiously Recurring Template Pattern (CRTP)

~~~cpp
template<typename Derived>
class CuriousBase { };

class Curious : public CuriousBase<Curious> { };

template<typename T>
class CuriousTemplate : public CuriousBase<CuriousTemplate<T>> { };
~~~

Example: object counter
~~~cpp
template<typename CountedType>
class ObjectCounter {
private:
    // in C++17, we can use inline initialization
    inline static std::size_t count = 0;
protected:
    ObjectCounter() {++count;} 
    ObjectCounter (ObjectCounter<CountedType> const&) {
        ++count;
    }
    ~ObjectCounter() { --count; }
public:
    static std::size_t live() {
        return count;
    }
};
~~~

#### Operator Implementations
It is useful when factoring behavior into a base class while retaining the identity of the eventual derived class.
~~~cpp
template<typename Derived>
class EqualityComparable
{
public:
    friend bool operator!= (Derived const& x1, Derived const& x2)
    {
        return !(x1 == x2);
    }
};

class X : public EqualityComparable<X>
{
public:
    friend bool operator== (X const& x1, X const& x2) {
        // XXXX
    }
};
~~~

#### Facades
CRTP base class defines most of the public interface.

~~~cpp
template<typename Derived, typename Value, typename Category,
         typename Reference = Value&, typename Distance = std::ptrdiff_t>
class IteratorFacade
{
public:
    // a list of alias names
    using value_type = typename std::remove_const<Value>::type;

    // access derived class
    Derived& asDerived() { return *static_cast<Derived*>(this); }
    
    Derived& operator++() {
        asDerived().increment();
        return asDerived();
    }
}

template<typename T>
class ListNodeIterator : public IteratorFacade<ListNodeIterator<T>, 
                                T, std::forward_iterator_tag>
{
    ListNode<T>* current = nullptr;
public:
    // ....
};
~~~

### Mixins
#### Common Mixins
We have an original design of Point and Polygon, users can easily inherite from Point. 
~~~cpp
class Point {};
class Point1: Point {};

template<typename P>
class Polygon{
private:
    vector<P> points;
}
~~~

It has shortcomings. Users must provide the same interface as Point. In Mixins, new classes are base classes as a template.

~~~cpp
template<typename... Mixins>    // let new classes be the base classes
class Point : public Mixins...
{
public:
    double x, y;
    Point() : Mixins()..., x(0.0), y(0.0) { }
    Point(double x, double y) : Mixins()..., x(x), y(y) { }
};

class Color { };
using MyPoint = Point<Color>;

template<typename... Mixins>
class Polygon
{
private:
    std::vector<Point<Mixins...>> points;
};
~~~

















