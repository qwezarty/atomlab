---
title: "Notes of python interface to pythia, the event generator, on macOS"
tags: ["physics"]
date: 2020-05-08T23:19:12+08:00
lastmod: 2020-05-09T20:10:12+08:00
author: "qwezarty"
draft: false
---

## 简介

[Pythia](http://home.thep.lu.se/Pythia/)是基于蒙特卡罗方法制作的Event Generator，最近的一个课题会需要使用到它。它是采用C++编写的，但却通过[SWIG](http://www.swig.org)支持了Python的调用。其官方的指引写的略微模糊，下面我会以macOS系统+VSCODE编辑器为例，简单讲一下如何实现Python调用。

## 配置编译选项 

我们需要在编译pythia的时候加入额外的选项，以获取python的支持，我们需要获得两个信息：

### python include path

```bash
python-config --includes
```

返回值例如：/System/Library/Frameworks/Python.framework/Versions/2.7/include/python2.7

### python path

```bash
echo $(dirname $(which python))/
```

返回值例如：/usr/bin/

注意，这里最后的"/"是必须的，不然你等等在编译的时候会出错（作者用的直接是+号拼接字符串）

## compile

最后，在pythia的根目录，我们重新配置再进行编译，将以下命令行里的两个path替换为你上面自己获取到的两个path，最后编译完了记得查看以下log，是否存在error，例如：

```bash
# 确保你在本地pythia的根目录
./configure --with-python-include=/System/Library/Frameworks/Python.framework/Versions/2.7/include/python2.7 --with-python-bin=/usr/bin/
# 编译它！
make
```

注：以上方案也适用于Linux，注意下如果你电脑里同时存在了python(2.7)和python3(3.7)，例如在macOS里就是这样，则要选择其中一个版本进行编译。但要注意，作者默认是采用了"python"作为调用指令的（而不是python3，所以很不友好），若想使用python3，则要先将python3制作一个软链接为python，然后将--with-python-bin指向到这个软连接的位置。（我更建议大家直接使用python，无论是版本2.7还是版本3.7都是可以的）

## 智能补全与DEBUG

### auto-completion

在VSCODE里，我们是通过pylint来进行代码补全的。我们需要在~/.bashrc里增加两个环境变量，使用你喜欢的编辑器打开它，在最后增加以下内容（注：将"your_pythia_lib_path"替换为你的pythia目录下，lib文件夹的绝对路径，例如: $HOME/Code/clang/pythia8244/lib）：

```
# added by pythia, the event generator
PREFIX_LIB="your_pythia_lib_path"
export PYTHONPATH=$PYTHONPATH:$PREFIX_LIB
export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:$PREFIX_LIB
```

最后在终端里重新加载~/.bashrc即可（所有开着的终端都需要，或者把所有终端重启），最后使用vscode打开pythia根目录，在example目录下存在文件：main01.py，尝试运行它并且测试以下智能补全是否可用。

```bash
source ~/.bashrc
```

一般而言到这里就可以了，如果依然不能够使用pylint的智能补全，不妨在~/.pylintrc里增加以下内容：
```
[MASTER]
init-hook='import sys; sys.path.append("your_pythia_lib_path")'
```

### debug

由于本身作者提供的python interface不是原生的python代码，所以给debug和变量监视带来了一些困难。在打好断点，进入debug以后，我们可以在debug console里使用dir(object)方法来查看所有object里提供的attribute和method，例如dir(pythia.event[0])等等。

