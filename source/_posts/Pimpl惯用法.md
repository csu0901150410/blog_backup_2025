---
title: Pimpl惯用法
date: 2025-11-30 22:34:17
tags: [C/C++, C++ API设计]
categories: [学习笔记]
---

> *Pimpl即“Pointer to Implementation”（指向实现的指针）的缩写，这是一种受限于C++特定限制的惯用法，主要用来避免在头文件中暴露内部的实现细节。*

### 1、使用Pimpl

考虑一个Timer类，该类的对象在构造时记录创建的时间戳，在析构时报告对象的生存时间，以下是一个简单的实现。

```cpp
// Timer.h
#pragma once
#include <string>

class Timer
{
public:
    explicit Timer(const std::string& name);
    ~Timer();

    void restart();

private:
    double get_elapsed() const;

    std::string m_timerName;
    double m_startTime;
};
```

如果将这个类提供给其他人使用，其他人获得头文件Timer.h之后，除了公有的接口restart能够调用之外，私有的接口get_elapsed和成员变量都无法调用和访问。所以对于Timer API的使用者来说，头文件中暴露的私有成员只会影响API的简洁性，而对于API开发者来说，后续对内部实现细节的调整都会直接反映到头文件中。基于此，我们可以将内部的实现细节隐藏在源文件中，而头文件仅将必要的方法暴露为公有（即构造函数、析构函数和restart）。以下是使用Pimpl重构之后的Timer类。

```cpp
// Timer.h
#pragma once
#include <string>

class Timer
{
public:
    explicit Timer(const std::string& name);
    ~Timer();

    void restart();

private:
    class PrivImpl;
    PrivImpl* m_impl;
};

// Timer.cpp
#include "Timer.h"
#include <iostream>

class Timer::PrivImpl
{
public:
    double get_elapsed() const
    {
        // 假设当前时间是10
        return 10 - m_startTime;
    }

    std::string m_timerName;
    double m_startTime;
};

Timer::Timer(const std::string& name)
    : m_impl(new Timer::PrivImpl())
{
    m_impl->m_timerName = name;
    // 假设初始时间是0
    m_impl->m_startTime = 0;
}

Timer::~Timer()
{
    std::cout << m_impl->m_timerName << ":took " 
        << m_impl->get_elapsed() << std::endl;
    delete m_impl;
    m_impl = nullptr;
}

void Timer::restart()
{
    m_impl = 0;
}

// main.cpp
#include "Timer.h"

int main()
{
    Timer timer("main");
    timer.restart();
}
```

重构之后的Timer类头文件中包含了暴露的公有成员和一个私有的指向Timer::PrivImpl类型变量的指针，并在Timer的构造函数和析构函数中分配和销毁该指针指向的对象，而原本Timer的私有成员被放到了Timer::PrivImpl类中，Timer类的内部实现细节需要通过Timer::PrivImpl指针去访问相关的变量。这样，其他人拿到Timer的头文件也无法了解和Timer实现相关的私有变量，而对Timer实现细节的更改也不会体现在头文件中。

---

### 2、复制语义

如果没有为类显式定义复制构造函数和赋值操作符，C++编译器会为类创建默认的复制构造函数和赋值操作符，但是这种默认的构造函数只能执行对象的浅复制，即简单地拷贝成员变量的值。对于Timer类的对象，这意味着如果用户复制了Timer对象，则两个对象中的m_impl是一样的，即指向了同一个Timer::PrivImpl对象。

而在Timer的析构函数中，会释放m_impl指向的对象，那么当复制前后的两个Timer对象析构时，就会对同一个m_impl指向的对象进行两次释放，这会导致崩溃，如下所示。

```cpp
Timer t0("main");
Timer t1(t0);
// Timer t1 = t0;
```

处理这个问题有以下两种可选方案。
- 禁止复制类。如果不打算让用户创建对象的副本，那么可以将对象声明为不可复制的（[参考SFML::NonCopyable](https://www.sfml-dev.org/documentation/2.6.1/NonCopyable_8hpp_source.php)）。
- 显式定义复制语义。如果希望用户能够复制采用Pimpl的类的对象，就应该生命并定义自己的复制构造函数和赋值操作符。它们可以执行对象的深复制，即创建Timer::PrivImpl对象的副本，而不是仅复制指针。

关于第一种禁止复制类的解决方案，有两种实现方法。一种是将构造函数和赋值操作符声明为私有成员，这样用户的复制代码将会在编译时报错，因为无法访问私有的复制语义成员。另一种是将构造函数和赋值操作符声明为公有成员但不提供实现，这样用户的复制代码将会在链接时报错，因为无法找到复制语义成员的实现。其实还有第三种方案，从C++11起，可以将构造函数声明为delete的，具体可以参考[Cppreference|Default constructors (4)](https://en.cppreference.com/w/cpp/language/default_constructor.html)。

以下是两种禁止复制类的实现方法。

```cpp
// 声明为私有成员
private:
	Timer(const Timer& other);
	const Timer& operator =(const Timer& other);

// 声明为公有成员但不提供实现
public:
	Timer(const Timer& other);
	const Timer& operator =(const Timer& other);
```
