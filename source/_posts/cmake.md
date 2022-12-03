---
title: cmake日常
date: 2022-11-04 21:40:17
categories: 
  - [tech]
  - [cpp]
tags: [tech,cpp]
category_bar: true
---

# ubuntu20.04 cmake 升级

阿里云自带的是3.16的，

**有坑！**

不要升级内核

`sudo apt install build-essential libssl-dev`不推荐执行，升级内核有可能造成服务器异常

`apt-get autoremove cmake`不推荐执行,会把之前安装的一些库包都删除了，以后如果要卸载什么包，最好用命令：`sudo apt remove package_name`，`atutoremove`命令不要轻易用

```shell
sudo apt remove cmake
wget https://cmake.org/files/v3.22/cmake-3.22.1-Linux-x86_64.tar.gz
tar zxvf  cmake-3.22.1-linux-x86_64.tar.gz
#建立软连接
sudo ln -s /root/cmake-3.22.1-linux-x86_64/bin/cmake /usr/bin/cmake
cmake --version
```

# 常用命令

```shell
cmake -B build 
#在build目录建立makefile等文件
cmake --build build
#在build目录开始编译
#或者直接并行执行makefile的编译
make -j$(nproc)
```



# 基础

最基本的是将源码构建成可执行文件，最简单的就是一个三行的`CMakelist.txt`分别定义了cmake的最低版本号，项目名，可执行文件

```cmake
cmake_minimum_required(VERSION 3.20.1)
project(hello)
add_executable(hello hello.cpp)
```

## 生成 & 构建 & 运行命令：

```shell
cmake -B build      # 生成构建目录，-B 指定生成的构建系统代码放在 build 目录
cmake --build build # 执行构建
./build/hello      # 运行 hello 程序
```

## 将文件看成library

例如许多文件都需要answer.cpp依赖我们就可以将其定义为library

```cmake
add_library(libanswer STATIC answer.cpp)
#当需要使用时，使用target_link_libraries就可以
add_executable(answer main.cpp)
target_link_libraries(answer libanswer)
```

## 将文件放到单独子目录

当项目不断变大时，多级目录是必不可少的，这时需要将每个文件整理的更加结构化，功能独立的模块可以放到一个子目录中
├── answer
│  ├── answer.cpp
│  ├── CMakeLists.txt
│  └── include
│     └── answer
│        └── answer.hpp
├── CMakeLists.txt
└── main.cpp

```cmake
# CMakeLists.txt
add_subdirectory(answer)
add_executable(answer_app main.cpp)
target_link_libraries(answer_app libanswer) # libanswer 在 answer 子目录中定义

# answer/CMakeLists.txt
add_library(libanswer STATIC answer.cpp)
target_include_directories(libanswer PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
```

`${CMAKE_CURRENT_SOURCE_DIR}`是内置变量，代表当前`CMakeLists.txt`所在的目录名

`PUBLIC`代表这个包含目录是libraries公开的，在其他`CMakeList.txt`使用libanswer可以直接#include该目录的文件

`PRIVATE`代表这个是libraries私有的，只有libraries能看到，对于使用libraries的文件不可见

## Cache变量

私密的 App ID、API Key 等不应该直接放在代码里，应该做成可配置的项，从外部传入。除此之外还可通过可配置的变量来控制程序的特性、行为等。在 CMake 中，通过 cache 变量实现：

`set(WOLFRAM_APPID "" CACHE STRING "WolframAlpha APPID")`

`set` 第一个参数是变量名，第二个参数是默认值，第三个参数 `CACHE` 表示是 cache 变量，第四个参数是变量类型，第五个参数是变量描述。

`BOOL` 类型的 cache 变量还有另一种写法：

```cmake
set(ENABLE_CACHE OFF CACHE BOOL "Enable request cache")
option(ENABLE_CACHE "Enable request cache" OFF) # 和上面基本等价
```

Cache 变量的值可在命令行调用 `cmake` 时通过 `-D` 传入：

```shell
cmake -B build -DWOLFRAM_APPID=xxx
```

也可用 `ccmake` 在 TUI 中修改

要让 C++ 代码能够拿到 CMake 中的变量，可添加编译时宏定义：

```cmake
target_compile_definitions(libanswer PRIVATE WOLFRAM_APPID="${WOLFRAM_APPID}")
```

这会给 C++ 代码提供一个 `WOLFRAM_APPID` 宏。`${WOLFRAM_APPID}`是CMakeLists.txt文件中cache变量的值

## header only

也就是.h文件

Header-only 的库可以添加为 `INTERFACE` 类型的 library：

```
add_library(libanswer INTERFACE)
target_include_directories(libanswer INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_compile_definitions(libanswer INTERFACE WOLFRAM_APPID="${WOLFRAM_APPID}")
target_link_libraries(libanswer INTERFACE wolfram)
```

通过 `target_xxx` 给 `INTERFACE` library 添加属性都要用 `INTERFACE`。

## target_complie_features

可以针对 target 要求编译 feature（即指定要使用 C/C++ 的什么特性）：

```
target_compile_features(libanswer INTERFACE cxx_std_20)
```

和直接设置 `CMAKE_CXX_STANDARD` 的区别：

1. `CMAKE_CXX_STANDARD` 会应用于所有能看到这个变量的 target，而 `target_compile_features` 只应用于单个 target
2. `target_compile_features` 可以指定更细粒度的 C++ 特性，例如 `cxx_auto_type`、`cxx_lambda` 等。

# 随笔

## find_package()

两种模式 module和config cmake默认采取Module模式，如果Module模式未找到库，才会采取Config模式。也可以显式的使用Config模式指定

`find_package(protobuf CONFIG REQUIRED)`

不推荐使用指定模式，除非很了解这个包。

 Module模式（模块模式）下是要查找到名为**FindXXX.cmake**的文件。这个文件负责找到库所在的路径，为我们的项目引入头文件路径和库文件路径。cmake搜索这个文件的路径有两个，一个是cmake安装目录下的Modules目录（如root/cmake-3.22.1-linux-x86_64/share/cmake-3.22/Modules），另一个使我们指定的CMAKE_MODULE_PATH的所在目录。

每一个FindXXX.cmake模块都会定义以下几个变量：

- \<LibaryName>_FOUND：用来判断FindXXX.cmake模块是否被找到；
- \<LibaryName>\_INCLUDE_DIR or \<LibaryName>_INCLUDES；
- \<LibaryName>\_LIBRARY or \<LibaryName>\_LIBRARIES；

## list()

```cmake
	list(LENGTH <list><output variable>)
    list(GET <list> <elementindex> [<element index> ...]<output variable>)
    list(APPEND <list><element> [<element> ...])
    list(FIND <list> <value><output variable>)
    list(INSERT <list><element_index> <element> [<element> ...])
    list(REMOVE_ITEM <list> <value>[<value> ...])
    list(REMOVE_AT <list><index> [<index> ...])
    list(REMOVE_DUPLICATES <list>)
    list(REVERSE <list>)
    list(SORT <list>)
```

LENGTH　　　　  　　　　 返回list的长度

GET　　　　　　   　　　返回list中index的element到value中

APPEND　　　　   　　　添加新element到list中

FIND　　　　　　  　　　　返回list中element的index，没有找到返回-1

INSERT 　　　　　　　　　 将新element插入到list中index的位置

REMOVE_ITEM　　　　　　从list中删除某个element

REMOVE_AT　　　　　　　从list中删除指定index的element

REMOVE_DUPLICATES    从list中删除重复的element

REVERSE 　　　　　　　　将list的内容反转

SORT 　　　　　　　　　　将list按字母顺序排序

## file()

***\*file(GLOB variable [RELATIVE path] [globbingexpressions]...)\****

GLOB 会产生一个由所有匹配globbing表达式的文件组成的列表，并将其保存到变量中。Globbing 表达式与正则表达式类似，但更简单。如果指定了RELATIVE 标记，返回的结果将是与指定的路径相对的路径构成的列表。 (通常不推荐使用GLOB命令来从源码树中收集源文件列表。原因是：如果CMakeLists.txt文件没有改变，即便在该源码树中添加或删除文件，产生的构建系统也不会知道何时该要求CMake重新产生构建文件。globbing 表达式包括：

  *.cxx   - match all files with extension cxx
  *.vt?   - match all files with extension vta,...,vtz
  f[3-5].txt - match files f3.txt,f4.txt, f5.txt
