---
title: CMake笔记
date: 2026-05-14 21:49:13
tags: [C/C++, 构建系统, CMake]
categories: [学习笔记]
---

> *刚毕业第一份工作做嵌入式单片机应用开发用的是Keil，后来用C++进行桌面开发用的是Visual Studio，最近想尝试一下用CMake构建项目，本文记录使用CMake的过程用作备忘。*

### CMake最小示例

在目录下分别创建`main.cpp`和`CMakeLists.txt`，后者包含CMake语言中的项目描述。

```cmake
cmake_minimum_required(VERSION 3.23)

project(App LANGUAGES CXX)

message("PROJECT_NAME : ${PROJECT_NAME}")

add_executable(${PROJECT_NAME})

target_sources(${PROJECT_NAME}
    PRIVATE
        main.cpp
)
```

`cmake_minimum_required(VERSION 3.23)`用来指定CMake的最小版本，必须指定一个版本，不同的版本有不同的特性，一般只有对某些特性的版本差异有需求时才会关注这个设置。

`project(App LANGUAGES CXX)`用来指定项目名称为App，指定项目语言为CXX，代表C++语言，必须指定项目名称。

`message("PROJECT_NAME : ${PROJECT_NAME}")`用来在构建过程中输出字符串信息，可用来辅助调试定位问题，`${PROJECT_NAME}`是获取CMake变量值的写法，`PROJECT_NAME`是存储`project()`语句中指定的项目名称的变量，`${}`用来取变量的值。

`add_executable(${PROJECT_NAME})`用来指定一个可执行程序，可执行程序名称设置为项目名称，或者说，指定了一个CMake中的目标（Target），目标的类型是可执行程序。

`target_sources(${PROJECT_NAME} PRIVATE main.cpp)`用来描述目标的源文件，第一个参数`${PROJECT_NAME}`表示目标的名字，`PRIVATE`表示依赖关系的权限。

核心是最后两行，可以这么理解，CMake中存在不同类型的目标，比如可执行程序、库，我们需要使用CMake去描述这些目标的属性，比如目标的名称、目标需要哪些文件来构建、目标之间的依赖关系。在使用诸如`target_sources()`来描述目标时需要提供目标名称，其中的PRIVATE表示权限，具体来说，目标依赖于`main.cpp`，但是依赖这个目标的其他目标，不关心也不需要依赖`main.cpp`，PRIVATE限制了App目标对`main.cpp`的依赖关系的传递。

接下来是构建过程。

1、在CMakeLists.txt所在目录执行命令`cmake -B build`，这将在相对路径`./build`中生成构建文件，我的环境是Windows+Visual Studio，CMake会自动检测工具链，并在指定的build目录下生成解决方案文件（App.sln文件），这时候打开解决方案文件可以自行编译生成可执行程序。可以看到，这个命令作用是生成指定工具链的工程文件。

2、在CMakeLists.txt所在目录执行命令`cmake --build build`，这将根据相对路径`./build`中生成的构建文件构建出定义的目标，比如可执行程序。

当然，我还是习惯使用Visual Studio进行调试，所以CMake目前对我而言就是一个能够替代手动在VS中创建工程的工具，就像是使用markdown语法定义文档格式而不是使用Word等软件格式化文档一样。

**参考链接**

[mastering-cmake](https://cmake.org/cmake/help/book/mastering-cmake/index.html)
