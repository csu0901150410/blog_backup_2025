---
title: C++代码调用C代码的方法
date: 2025-04-20 22:46:05
tags:
categories: C++
---

在混合编程环境中，C++代码调用C代码是一种常见的需求，extern "C"可以实现这种需求。有很多库是用C语言编写的，为了能够在C++中使用这些库，需要使用一个特殊的语法。

假设现在有一个C库，头文件是module_c.h，实现在module_c.c中，如下所示。

```c
// file module_c.h
#ifndef __MODULE_C_H__
#define __MODULE_C_H__

int func_in_c(int val);

#endif // __MODULE_C_H__

// file module_c.c
#include "module_c.h"
int func_in_c(int val) { return val; }
```

如果需要在C++源文件main.cpp中包含module_c.h并调用func_in_c函数，visual studio在链接阶段会报错LNK2019无法解析的外部符号，代码如下所示。

```cpp
#include "module_c.h"
int main() {
    func_in_c(0);
}
```

由于C、C++的编译器对函数的编译处理是不完全相同的，尤其是对于C++来说其支持函数的重载，编译后的函数一般根据函数名和形参类型对函数进行重命名，而C则没有这个机制，这就会导致，在C++源文件中，函数func_in_c编译之后名称可能变成了func_in_c_@xxx的形式，从而导致链接时无法根据文件名找到对应的实现，所以需要在C++代码中调用C函数时，需要一个机制明确指明所调用的函数是一个C风格的函数，即使用extern "C"修饰。代码如下所示。

```c
// file module_c.h
#ifndef __MODULE_C_H__
#define __MODULE_C_H__

extern "C" {
    int func_in_c(int val);
}

#endif // __MODULE_C_H__
```

所作的改动仅仅是把C风格的函数声明在头文件的extern "C"块中，这样C++编译器就知道这部分代码应该按照C语言的编译方式进行编译，就不会对函数名进行重命名了。
