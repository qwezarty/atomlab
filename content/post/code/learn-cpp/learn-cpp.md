---
title: "如何在两周内学会C++并构建优质的项目"
tags: ["code"]
date: 2020-09-20T21:14:57+08:00
draft: true
author: "qwezarty"
---

## 简介

最近因为科研需要，捡起了好几年前大学水水的C++（土木工程专业的课程，太水了可以说基本等于没学过），毫无意外地忘记地一干二净。于是两周后就有了这篇文章，期望能够帮助所有拥有一定编程基础（至少写过一个完整的项目的那种）的同学入门并掌握这一门编程语言。通过阅读这篇文章，你能够学到：

1. 用优质的资源，快速学习C++的语法构成（耗时一周）
2. 指针和防止内存泄漏等，较难的需要实战（编码）的内容
3. C++标准的现代化目录结构
4. 利用CMake来进行跨平台开发/编译
5. Doxygen自动化生成API文档
6. 日志、调试、测试、持续集成等进阶内容（缓慢更新中）

我的开发平台是"macOS Catalina"，选择的IDE是"vscode"，编译器采用了"g++"，在考虑兼容性和新特性之后，我选择了"c++17"版本进行开发。文中许多的资源，是需要梯子才能够访问的，这点请注意。

## 快速学习C++语法结构

### 写出你的 "Hello, world!"（VSCODE配置）

首先，在你的本地代码仓库中建立一个文件夹，名字随意，创建一个 hello.cc 文件并保存，内容如下（现在不需要去关心这些内容代表什么）：

```c++
#include <iostream>

int main() {
    std::cout << "hello, world!" << std::endl;

    return 0;
}
```

打开 terminal 以后，你就可以进行手动编译了：

```bash
# 编译
g++ hello.cc -o hello.o
# 运行
./hello.o
```

当然，每次进行这样的手动编译，实在繁琐并且无法进行断点调试。在VSCODE中编写C++项目，首先你要安装一个"C/C++"的扩展，然后打开刚才存放 hello.cc 的目录，双击打开 hello.cc 以后，按住 cmd+shift+p 呼出vscode的命令面板，输入 tasks: 索引到 "Tasks: Configure Task"（如图所示）选择编译器，这里我选择了g++作为编译器，之后你能在.vscode目录下发现自动创建出的 "tasks.json"（当然我们要进行修改）

![task](../task.png)

之后选择左侧的调试按钮，创建一个 "launch.json"，附上我的两个配置文件如下：

* .vscode/task.json
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "shell",
      "label": "C/C++: g++ build active file",
      "command": "/usr/bin/g++",
      "args": [
        "-std=c++17",
        "-g",
        "${file}",
        "-o",
        "${fileDirname}/caches/${fileBasenameNoExtension}.o"
      ],
      "options": {
        "cwd": "${workspaceFolder}"
      },
      "problemMatcher": ["$gcc"],
      "group": {
        "kind": "build",
        "isDefault": true
      }
    }
  ]
}
```

* .vscode/launch.json
```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "g++ - Build and debug active file",
      "type": "cppdbg",
      "request": "launch",
      "program": "${fileDirname}/caches/${fileBasenameNoExtension}.o",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${workspaceFolder}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "lldb",
      "preLaunchTask": "C/C++: g++ build active file"
    }
  ]
}
```

接下来你需要尝试在 hello.cc 中加个断点，进行断点调试等等操作，确保一切没有问题以后开始真正地学习C++的语法结构！额外再提一句，我们这里的配置仅仅作为学习时使用，它仅仅只能够编译单个文件，所以当你引用自己写的库的时候可能会产生 linker error （不知道什么意思也没关系，后面你会学到）别担心，等你学完了所有的语法结构，我会提及如何构建一个真正项目级别的编译配置。

### 需要弄懂的知识点

你要做的事，就是在一周内刷完这个列表（油管，需要梯子）：[C++ by The Cherno](https://www.youtube.com/playlist?list=PLlrATfBNZ98dudnM48yfGUldqGD0S4FFb)，任务很艰巨，偷懒一点的话但至少也请刷掉前70个视频（每个10-20分钟左右），加油吧！少年！

这个过程至少会花费你20个有效时（全神贯注，效率极高的那种）当然现实中，我们会松懈、会偷懒、还要偶尔摸鱼划水，所以刷视频真正要花费的时间，估计要超过30小时甚至更多。其次，某一些视频看完疑惑的点，你还要自己去编码弄懂，调试看看到底是怎么回事，这也额外需要时间。在这个阶段也是最容易放弃的，请坚持坚持再坚持！它总共会占用你大概50小时，意味着一周6天里每天你要花费8小时在这上面，也请做好心理准备。（科研民工就别想着双休了哈！）

以下是你刷教学视频/或刷完以后，要知道的知识点：

* 编译总共会经历哪几个过程？它们分别做了什么？
* translation unit 是怎么划分的？试着用代码在同一个项目中模拟两个 translation unit 并编译
* 为什么我们需要 linking stage？如何解决一个简单的 linker error？
* header file 存在的意义是什么？（至少能说出三点以上吧）
* C++中基础变量的种类，变量/指针变量的区别
* std::string 的本质是什么？如何正确地把一个 string 传入给某个函数？
* stack memory 和 pile memory 的特征和区别，在C++中如何分配一块内存？
* 编写函数时，pass by value 和 pass by refrence 哪里不一样？该如何选择？
* class 和 struct 有什么不一样？在什么时候我们要使用 struct？
* constructor/destructor 如何将外部传入参赋值给内部隐藏变量？若传入的是一个数组又该如何处理？
* inheritance, interface/abstract-class, visibility(private/protect/public)
* 简单实现一个多态的例子，接收一个父类型，却能表现出不同子类的特征
* 在哪些情况下，我们会使用 const 关键字？说出 int* const 与 const int* 的区别
* 三种不同 smart pointer 的区别，该如何正确使用它？
* 如何覆盖原有的 operator，顺便去看看内建的 operator 有哪些吧
* 浅拷贝、深拷贝的区别，实现一个函数利用 memcpy 来深拷贝一个大型的对象
* 如何写 lambda 表达式？不同的 lambda capture 区别在哪？利用标准C++库"algorithm"写几个例子
* 多线程编程、线程锁、等待和同步
* 更多其他的知识：template、预编译加速等等

最后附上我很喜欢的一个语法/概念参考网站：[cpp refrence](https://en.cppreference.com/w/)，书籍方面我也还没来得及看，也推荐两本吧（大家看自己的需求和定位来选择读哪一本）：

* [A Tour of C++](https://www.amazon.com/gp/product/0134997832), 256 pages
* [C++ Programming Language](https://www.amazon.com/Programming-Language-hardcover-4th/dp/0321958322)，1376 pages

## 指针和防止内存泄漏

C++成也指针，败也指针。如果你会许多种高级的编程语言（比如Python/Java/Go等等）但没接触过偏向底层的语言，那么你应该花更多的时间在内存管理这块知识上。这部分作为入门你应当去看一下 Bjarne Stroustrup 给出的报告（2014 at the University of Edinburgh）[The Essence of C++](https://www.youtube.com/watch?v=86xWVb4XIyE&t=4678s)，Bjarne 在报告中举了这样一个令人深思的问题：

> 我对着商店的镜子挥手，我拿到了镜子上的形状（我的手）的指针。谁来对这个形状的销毁负责？是镜子？还是我？然后我把这个指针又传递给了另一个人，现在是他的工作（指销毁这个形状）还是我的？

在真正的编码过程中，尤其是操纵大量的数据在函数之间传递，就会出现上述的情况。Bjarne 在报告中分享了如何安全正确地使用指针（based on C++14）以及什么时候改使用指针。我强烈建议你看完这个报告，它会让你对C++的指针理解更加深刻。

另一个资料是国内的，文中给的例子很多，能够帮助你在初期杜绝很多指针用法的错误：[C++内存管理](https://www.cnblogs.com/findumars/p/5929831.html?utm_source=itdadao&utm_medium=referral)

C++的编写思路会和高级编程语言不太一样（可以说，你越精通就会发现它们越不一样），更多的在C++中你要关心的是：这个资源到底属于谁？这个类，这个函数，能不能正确地在这一小块内存中安全地被创建销毁？异常处理会不会导致内存泄漏？在这里我要不要用并行来加速运行效率？诸如此类，然后我们再来说说那些拥有 Garbage Collection 的高级编程语言，你需要考虑的东西，更多的是要不要新增一个实体类？这里能不能用设计模式抽象一下？等等。

在编写C++的时候，很有必要的一种认知就是：我写的代码，是需要沟通机器的，我得考虑电脑的行为和感受，其次再是我想要实现的内容，想要用代码创造出的世界（实体类、逻辑联系），牢记这一点很有必要。

## 标准的目录结构

这部分简单一些的话，你可以参考以下两个 stack-overflow 的回答，分别是：

* [C++ project organisation](https://stackoverflow.com/questions/13521618/c-project-organisation-with-gtest-cmake-and-doxygen)
* [Directory structure for a C++ library](https://stackoverflow.com/questions/1398445/directory-structure-for-a-c-library)

当然，我更推荐你看完这份资料 [The Pitchfork Layout (PFL)](https://api.csswg.org/bikeshed/?force=1&url=https://raw.githubusercontent.com/vector-of-bool/pitchfork/develop/data/spec.bs)，我是按照这份资料来设计自己的项目结构的。

作为参考，我的玩具代码是拥有了一个主入口的（可以作为CLI程序来用），并且同时项目有做成库的需求，所以我选用了 separate header placement，但需要注意这种模式会给你的编译带来麻烦，所以后续的 cmake 配置也会较为复杂。举个玩具例子，我想写一个绘图程序，能够画出一些简单的图形，在绘制的同时有一些声效，我构建的项目结构如下：

```
toy-application
│   README.md
│   LICENSE
│
└───build	// cmake 本地编译的缓存文件夹
└───data	// 用来存放依赖的静态文件，例如图片、声音等等
└───docs	// doxygen 生成的文档文件夹
└───external	// 依赖的外部库，都放在这里
└───include	// 对外公开的 headers，但用来存放公开的 headers
│   └───graph	// 目录结构需要与 src 中一致
│       │   graph.config.hpp	// 用于将 cmake 参数传入主程序
│       │   
│       └───shape	// 用来存放绘图程序中的图形们，一种图形是一个 translation unit
│       │      circle.hpp	// 圆的头文件
│       │
│       └───sound // 用来存放绘图过程中，对声效的实现
│
└───src		// 存放源代码的目录
│   │   CMakeLists.txt	// 第一个 cmake 的配置，仅存放一些全局的变量、版本号、编译器的配置等
│   │   Doxyfile.in
│   │
│   └───graph // 你真正的项目名称，比如我想写个绘图程序
│       │   CMakeLists.txt	// 第二个 cmake 的配置，用来配置要编译的 libraries，还有依赖
│       │   graph.config.hpp.in	// 用于将 cmake 参数传入主程序
│       │   graph.cpp		// CLI主程序入口
│       │   graph.test.cpp	// 主程序入口的测试文件
│       │   
│       └───shape	// 用来存放绘图程序中的图形们，一种图形是一个 translation unit
│       │      circle.area.cpp	// 比较复杂的函数，单独一个文件来实现
│       │      circle.circumference.cpp	// 比较复杂的函数，单独一个文件来实现
│       │      circle.cpp	// 较为简单的函数们都在这里
│       │      circle.test.cpp	// 测试文件
│       │
│       └───sound // 用来存放绘图过程中，对声效的实现
│
└───tools // 主要用来放一些脚本文件，例如安装/自动化测试的脚本
```

这里的 include 与 src 文件夹的内部目录结构要一致，include 仅仅存放那些对外公开的 headers，src 则是保存源代码对其具体实现的地方。未来在进行编译时，我们会直接把 include/graph 拷贝到 /usr/local/include/graph 中去，这样就变成了可以被别人引用的库。总的来说，选择 seperate header 还是 merged header，还是要看项目的大小，以及是否有被别人引用的需求。另一方面，即便是 merged header，也是能够通过配置 cmake 做成库给别人引用的。

## CMake配置和编译

不同于Win平台拥有的"宇宙第一IDE"（Visual Studio），在类 Unix 操作系统上写C++的项目会更加费劲一些，主要的麻烦来源就是：编译。我选择了 cmake 来帮助我配置和自动化掉编译的过程，它与VSCODE的兼容性良好，你需要在VSCODE中安装两个额外的扩展：CMake, CMake Tools。当然你也可以不使用它，通过自己写几个 shell script 来实现（对于那些小型CLI程序来说，这样做完全没有问题！）

这里我推荐你按以下的顺序看完两份资料，分别是：

1. [How to CMake Good](https://www.youtube.com/playlist?list=PLK6MXr8gasrGmIiSuVQXpfFuE1uPT615s)
2. [CMake Tutorial](https://cmake.org/cmake/help/v3.16/guide/tutorial/index.html)

两份资料看完以后，你就可以安装你的项目需求，尝试构建一个标准的目录结构，再塞入一些玩具代码，把 cmake 的配置文件写好，然后用VSCODE跑起来。你需要尝试并确保以下几个问题（如果下面这些术语看不明白的话，请先阅读上一小节的资料）：

1. 创建一个 main 入口，创建两个 physical component，尝试将它们连结在一起
2. 无论你选择了 separate header 还是 merged header，你要保证它在 main 中的 include path，是具有 logical/physical 一致性的
3. 检查 builds/compile_commands.json 文件，确保生成的编译命令行是正确的（尤其是 include path）
4. 能够对每一个 physical component 进行 debug，能够加设断点，打中断点
5. 尝试把这个玩具程序安装到你的系统中（而不是在代码库里运行）

关于 seperate header 的 cmake 配置，你可以参考 [How to set include directories in CMake](https://github.com/vector-of-bool/pitchfork/issues/18)，下面是对我之前玩具代码的 cmake 配置，总共有两个，分别是：

* 第一个 cmake 的配置，仅存放一些全局的变量、版本号、编译器的配置等

```cmake
# configures required and project version/name
cmake_minimum_required(VERSION 3.18)
project(MyGraph VERSION 1.0)

set(CMAKE_CXX_STANDARD 17) # switch to c++17
set(CMAKE_CXX_STANDARD_REQUIRED True) # prevent fall back behaviour
# modify flat to -std=c++17 rather than -std=gnu++17
set(CMAKE_CXX_EXTENSIONS OFF) 

add_subdirectory(graph)
```

* 第二个 cmake 的配置，用来配置要编译的 libraries，还有依赖

```cmake
# set variables
set(PROJECT_INCLUDE_DIR "${PROJECT_SOURCE_DIR}/../include/graph")

# configures the libraries going to build
add_library(
    shape
    shape/circle.cpp
    shape/circle.area.cpp
    shape/circle.circumference.cpp
)
list(APPEND EXTRA_LIBS shape)
target_include_directories(shape PUBLIC ${PROJECT_INCLUDE_DIR}/shape)

# configure a header file to pass some of the CMake settings to the source code
configure_file(graph.config.hpp.in ${PROJECT_INCLUDE_DIR}/graph.config.hpp)

# configure excutable
add_executable(MyGraph graph.cpp)
target_link_libraries(MyGraph PUBLIC ${EXTRA_LIBS})
target_include_directories(MyGraph PUBLIC ${PROJECT_INCLUDE_DIR})

# print default install path
message(STATUS "Default install prefix path is:")
message(STATUS ${CMAKE_INSTALL_PREFIX})
# install all libraries to system
install(TARGETS ${EXTRA_LIBS} DESTINATION lib)
install(DIRECTORY ${PROJECT_INCLUDE_DIR} DESTINATION include)
# install executable to system
install(TARGETS MyGraph DESTINATION bin)
```

在这样的配置下，你执行 cmake --install 的时候，总共会有创建以下的文件：

1. include/graph -> /usr/local/include/graph
2. libshape.a -> /usr/local/lib/libshape.a
3. MyGraph -> /usr/local/bin/MyGraph

一切都符合Linux的FHS标准（[Filesystem Hierarchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)），了解目录结构和 cmake 的配置，然后尝试直到最后到完全编译、安装成功，大概会花费你2天以上的时间。

## Doxygen自动生成文档

## CTest单元测试

## Bonus：进阶内容！（持续更新）

