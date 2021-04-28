---
title: "如何在两周内学会C++并构建优质的项目"
tags: ["code"]
date: 2020-09-20T21:14:57+08:00
draft: false
---

## 简介

最近因为科研需要，捡起了好几年前大学水平的C++（不忍直视），毫无意外地忘记地一干二净。于是两周后就有了这篇文章，期望能够帮助所有拥有一定编程基础（至少写过一个完整的项目的那种）的同学入门并掌握这一门编程语言。通过阅读这篇文章，你能够学到：

1. 用优质的资源，快速学习C++的语法构成
2. 指针和防止内存泄漏等，较难的需要实战（编码）的内容
3. C++标准的现代化目录结构
4. 利用CMake来进行跨平台开发/编译，把开发的软件安装到系统中
5. 利用Catch2来编写单元测试，集成到CMake中去
6. Doxygen自动化生成API文档
7. Shell脚本与CMake联动，进一步实现自动化（龟速更新中）
8. 跨系统交叉编译，做成Docker镜像（未来用到了再更）

我的开发平台是"macOS Catalina"，选择的IDE是"vscode"，编译器采用了"g++"，在考虑兼容性和新特性之后，我选择了"c++17"版本进行开发。文中许多的资源，是需要梯子才能够访问的，这点请注意。

## 快速学习C++语法结构

### 写出你的 "Hello, world!"（VSCODE配置）

首先，让我们从简单难度开始：在你的本地代码仓库中建立一个文件夹，名字随意，创建一个 hello.cc 文件并保存，内容如下（现在不需要去关心这些内容代表什么）：

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

当然，我更推荐你看完这份资料 [The Pitchfork Layout (PFL)](https://api.csswg.org/bikeshed/?force=1&url=https://raw.githubusercontent.com/vector-of-bool/pitchfork/develop/data/spec.bs)，我是参考这份资料同时结合了现代化的 cmake 项目设计，做了些许改动然后定下项目结构的。

作为参考，我的玩具代码是拥有了一个主入口的（可以作为CLI程序来用），并且同时项目有做成库的需求，所以我选用了 separate header placement，但需要注意这种模式会给你的编译和测试带来很大麻烦，所以后续的 cmake 配置也会较为复杂。举个玩具例子，我想写一个绘图程序，能够画出一些简单的图形，在绘制的同时有一些声效，我构建的项目结构如下：

```
toy-application
│   README.md
│   LICENSE
│   CMakeLists.txt	// 第一个 cmake 的配置，仅存放一些全局的变量、版本号、编译器的配置等
│   Doxyfile // 自动化生成API文档的配置
│   Dockerfile // 打包成 docker 镜像的配置
│
└───build	// cmake 本地编译的缓存文件夹
└───data	// 用来存放依赖的静态文件，例如图片、声音等等
└───docs	// doxygen 生成的文档文件夹
└───extern	// 依赖的外部第三方库，都放在这里
│      CMakeLists.txt	// 第二个 cmake 的配置，主要解决依赖第三方库
│
└───apps	// CLI程序的源代码
│      CMakeLists.txt	// 第三个 cmake 的配置，是对CLI程序（调用我们自己写的）的配置
│      graph.cpp		// CLI主程序入口
│
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
│   │   CMakeLists.txt	// 第四个 cmake 的配置，用来配置要编译的我们自己写的绘图库
│   │   graph.config.hpp.in	// 用于将 cmake 参数传入主程序
│   │   
│   └───shape	// 用来存放绘图程序中的图形们，一种图形是一个 translation unit
│   │      circle.area.cpp	// 比较复杂的函数，单独一个文件来实现
│   │      circle.circumference.cpp	// 比较复杂的函数，单独一个文件来实现
│   │      circle.cpp	// 较为简单的函数们都在这里
│   │
│   └───sound // 用来存放绘图过程中，对声效的实现
│
└───tests		// 存放所有单元测试的代码
│   │   CMakeLists.txt	// 第五个 cmake 的配置，引入第三方单元测试库，编译所有的单元测试
│   │   main.test.cpp	// 要把单元测试编译成可执行程序的外壳
│   │   
│   └───shape
│   │      circle.test.cpp	// 对圆的测试文件
│   │
│   └───sound // 用来存放绘图过程中，对声效的实现
│
└───tools // 主要用来放一些脚本文件，例如安装/自动化测试的脚本
    │   docker_build.sh	// 打包成 docker 镜像
    │	native_build.sh	// 在本机上编译并安装我们写的绘图库
    │	uninstall.sh	// 卸载删除安装的绘图库
```

这里的 include 与 src 文件夹的内部目录结构要一致，include 仅仅存放那些对外公开的 headers，src 则是保存源代码对其具体实现的地方。未来在进行编译时，我们会直接把 include/graph 拷贝到 /usr/local/include/graph 中去，这样就变成了可以被别人引用的库。apps 里则是CLI可中可运行的代码，也单独分离出去了。总的来说，选择 seperate header 还是 merged header，还是要看项目的大小，以及是否有被别人引用的需求。

这样的项目结构设计已经很复杂并且足够强大了（我没能在网上能找到这么复杂的例子），我把一个 physical unit 分成了三份（header, implementation, test）放在不同的地方，这样做的优缺点是：

1. 最后把绘图库打包发布后，库的使用者在编译时并不会引入单元测试（这部分不需要编译），这样能够加快下载/编译速度。
2. 分开放置的头文件，能够降低打包/安装难度，同时也能够拥有内部隐藏的头文件，方便内部开发和外部使用者的调用（理想一些的话，能做到使用者不用看你的源码，只需要看公开的头文件们+文档注释，就能知道函数/类的用法）
3. 单独放置的单元测试，方便独立的管理，不会和主代码混在一起（主要是CMakeLists.txt配置的分离）
4. 缺点是 cmake 配置编译的确有些复杂，较为适用于中型/大型项目的开发，相对于 merged header 来说比较笨重（光5个CMakeLists.txt就很劝退了）

## CMake配置和编译

不同于Win平台拥有的"宇宙第一IDE"（Visual Studio），在类 Unix 操作系统上写C++的项目会更加费劲一些，主要的麻烦来源就是：编译。我选择了 cmake 来帮助我配置和自动化掉编译的过程，它与VSCODE的兼容性良好，你需要在VSCODE中安装两个额外的扩展：CMake, CMake Tools。当然你也可以不使用它，通过自己写几个 shell script 来实现（对于那些小型CLI程序来说，这样做完全没有问题！）

安装完两个扩展以后，可以在VSCODE中，按 cmd+shift+p 呼出以下界面，进行 configure/build/install：

![cmake](../cmake.png)

这里我推荐你按以下的顺序看完两份资料，分别是：

1. [How to CMake Good](https://www.youtube.com/playlist?list=PLK6MXr8gasrGmIiSuVQXpfFuE1uPT615s)
2. [CMake Tutorial](https://cmake.org/cmake/help/v3.16/guide/tutorial/index.html)

两份资料看完以后，你就可以安装你的项目需求，尝试构建一个标准的目录结构，再塞入一些玩具代码，把 cmake 的配置文件写好，然后用VSCODE跑起来。你需要尝试并确保以下几个问题（如果下面这些术语看不明白的话，请先阅读上一小节的资料）：

1. 创建一个 main 入口，创建两个 physical component，尝试将它们连结在一起
2. 无论你选择了 separate header 还是 merged header，你要保证它在 main 中的 include path，是具有 logical/physical 一致性的
3. 检查 builds/compile_commands.json 文件，确保生成的编译命令行是正确的（尤其是 include path）
4. 能够对每一个 physical component 进行 debug，能够加设断点，打中断点
5. 尝试把这个玩具程序安装到你的系统中（而不是在代码库里运行）

另外还有一份额外的资料，也希望你能够阅读：[An Introduction to Modern CMake](https://cliutils.gitlab.io/modern-cmake/)，它会回答你心中很多关于 cmake 的疑问，例如：增加依赖库、编译时运行单元测试、对 cmake script 进行调试、Do's and Don'ts 等等。

关于 seperate header 的 cmake 配置，你可以参考 [How to set include directories in CMake](https://github.com/vector-of-bool/pitchfork/issues/18)，下面是对我之前玩具代码的 cmake 配置，总共有五个，分别是：

* 第一个 cmake 的配置，仅存放一些全局的变量、版本号、编译器的配置等

```cmake
# configures required and project version/name
cmake_minimum_required(VERSION 3.14)
project(
    CustomGraph VERSION 0.1
    DESCRIPTION "My custom graph library and cli-app."
    LANGUAGES CXX
)

set(CMAKE_CXX_STANDARD 17) # switch to c++17
set(CMAKE_CXX_STANDARD_REQUIRED True) # prevent fall back behaviour
# modify flat to -std=c++17 rather than -std=gnu++17
set(CMAKE_CXX_EXTENSIONS OFF) 

# custom variables
set(CMAKE_INCLUDE_DIR "${CMAKE_SOURCE_DIR}/include")
set(PROJECT_INCLUDE_DIR "${CMAKE_SOURCE_DIR}/include/graph")

add_subdirectory(extern)
add_subdirectory(src)
add_subdirectory(apps)
add_subdirectory(tests)
```

* 第二个 cmake 的配置，主要解决依赖第三方库，你可以增加更多的库

```cmake
# dependencies install path
message(STATUS "Dependencies are installed at: " ${FETCHCONTENT_BASE_DIR})

include(FetchContent)
set(FETCHCONTENT_QUIET off)

# formatting library
FetchContent_Declare(
    fmtlib
    GIT_REPOSITORY  https://github.com/fmtlib/fmt.git
    GIT_TAG 5.3.0
)

# make the libraries availible
FetchContent_MakeAvailable(fmtlib)
```

* 第三个 cmake 的配置，是对CLI程序（调用我们自己写的）的配置

```cmake
# configure excutable
add_executable(MyGraph graph.cpp)
target_link_libraries(MyGraph PRIVATE graph)
target_include_directories(MyGraph PUBLIC ${PROJECT_INCLUDE_DIR})

# install executable to system
install(TARGETS MyGraph DESTINATION bin)
```

* 第四个 cmake 的配置，用来配置要编译的我们自己写的绘图库

```cmake
# set variables
message(STATUS "CMake project include dir: " ${PROJECT_INCLUDE_DIR})

# configures the libraries going to build
add_library(
    graph
    shape/circle.cpp
    shape/circle.area.cpp
    shape/circle.circumference.cpp
)
list(APPEND EXTRA_LIBS graph)
target_include_directories(graph PUBLIC ${PROJECT_INCLUDE_DIR})

# configure a header file to pass some of the CMake settings to the source code
configure_file(graph.config.hpp.in ${PROJECT_INCLUDE_DIR}/graph.config.hpp)

# print default install path
message(STATUS "Default install prefix path is: " ${CMAKE_INSTALL_PREFIX})
# install all libraries to system
install(TARGETS ${EXTRA_LIBS} DESTINATION lib)
install(DIRECTORY ${PROJECT_INCLUDE_DIR} DESTINATION include)
```

* 第五个 cmake 的配置，引入第三方单元测试库（Catch2），编译所有的单元测试

```cmake
# add testing library
FetchContent_Declare(
  catch2
  GIT_REPOSITORY https://github.com/catchorg/Catch2.git
  GIT_TAG        v2.5.0
)
# make catch2 availible
FetchContent_MakeAvailable(catch2)

add_executable(test_main 
    main.test.cpp
    shape/circle.test.cpp
)
# linking
target_link_libraries(test_main PRIVATE Catch2::Catch2 graph)
```

在这样的配置下，你执行 cmake --install 的时候，总共会有创建以下的文件：

1. include/graph -> /usr/local/include/graph
2. libshape.a -> /usr/local/lib/libshape.a
3. MyGraph -> /usr/local/bin/MyGraph

一切都符合Linux的FHS标准（[Filesystem Hierarchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)），的确C++的编译流程是很复杂的，cmake 本身的语法也可以算作是一种编程语言（do and don't 里也提到了：要把编写CMakeLists.txt本身作为写一份代码来看待）。总而言之，了解目录结构和 cmake 的配置，然后尝试直到最后到完全编译、安装成功，大概会花费你三天以上的时间。

## Catch2单元测试

单元测试大概是不陌生的，我很喜欢TDD开发，尤其是在写一些会给别人调用的库的时候（[Test-driven development](https://en.wikipedia.org/wiki/Test-driven_development)），单元测试的编写相对来说就不太难了，这里也就不再介绍，具体的你可以看一下 [Catch2-Tutorial](https://github.com/catchorg/Catch2/blob/master/docs/tutorial.md#top)，以及 [Catch2-List-of-Examples](https://github.com/catchorg/Catch2/blob/master/docs/list-of-examples.md)，与 cmake 的联动则已经在上一小节配置好了，采用的是较为复杂的多文件单元测试（还是那句话，玩玩可以，真实项目要看情况选择复杂度）。

更为传统一些的话，可以选择 [google-test](https://github.com/google/googletest)，它在制作 mock 数据方面更加方便，功能也更为齐全。另一方面，Catch2 的优势则是它仅仅只有一个头文件，拖到哪都能用，比较轻量。

这里再多科普几句TDD，“测试驱动开发”：代码未动，测试先行。在真正的逻辑代码编写之前，我们就先编写所有的测试（所有可能导致BUG的情况全部考虑进去），调用的API也是假的。随后编写业务逻辑代码的时候，要满足这些所有的测试。所以总的来说，是用单元测试来驱动整个代码开发。这样做的优缺点也是显而易见的：

1. 最大化保证了你的业务代码没有BUG，这对编写供他人使用的库的时候很关键
2. 任意一个时刻，递交出去的版本总是可以使用的（因为涵盖了所有可能的测试）
3. 测试即文档，别人可以通过阅读你的单元测试代码，知道函数的使用细节
4. 因为单元测试的约束和理清逻辑，所以其实能够保证你业务代码/设计的精简
4. 缺点则是测试的代码量很大（甚至高达业务代码的两倍），并且一旦有很大的需求更改，单元测试要先进行改动，一定程度上增加了复杂度

![test](../test.png)

## Doxygen自动生成文档

Doxygen这部分也不难，也是一门开发人员的必修课吧（必修的原因比较恶心，纯粹是规章制度），Doxygen不止可以作用于C++，还可以是其他高级编程语言（Java、C#、Python、Fortran、etc），所以如何使用的细节便不再提了，随便在Youtube上找个视频看个半小时你就会了。

主要是我不喜欢遍地写注释和文档，原因是文档这东西就是忽悠人的，为啥？当项目比较小，或者只有几个人一起协同工作时，根本不需要文档，面对面沟通来的更快更有效，并且代码量也不大复杂度也不高，看源码完全可以明白。另一方面当项目变得庞大，开发人员变多的时候，你如果作为项目的负责人，经常出现的一件事就是文档和真实代码之间存在错误，有极大的可能是某个人更改了代码，但是没动注释和文档，这种错误会让内部的沟通变得低效，并且维护这些注释和文档本身也是需要花费很大精力的（再吐槽一句，那些不设主程和架构师的软件公司，千万不要去！）

“一定要写类和函数的头注释？”，这本身就是意见很荒谬的事情。文档就是写给那些不懂代码的人看的，文档的正确写法应该是描述清楚整个软件的构架、优势、存在的问题、责任划分、如何使用等等。

一个好的构架 + 许多的单元测试 = 一份足够优秀的开发文档。

## Shell脚本实现一些自动化

## 交叉编译与Docker镜像

## 更多的内容

写不动了，放我一条生路，也许下次吧（-，-!）

