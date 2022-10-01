---
title: "Jupyter安装与基础入门"
tags: ["code"]
date: 2022-09-30T23:26:35+08:00
lastmod: 2022-10-01T12:26:35+08:00
draft: false
---

这篇教程面向的是从未接触过终端（命令行）的同学（通常是Windows用户），所以我会以Windows平台介绍如何安装Jupyter Notebook（但macOS与Linux也绝对通用啦），并简单介绍它的理念和基础使用。

题外话：为什么要使用Jupyter？

其实这个答案有很多，我认为则是它提供了交互式的（所见即所得）的编程方式，就像曾经的 Microsoft Word 一样。对的，它极大地降低了编程的难度，让更多的人能够学习并享受程序的乐趣。（同时它也真的超级适合科研领域）

~~题外话的题外话：对于大型项目使用Jupyter是一场灾难。~~

## 我使用Windows系统

前往[Python.org](https://www.python.org/downloads/windows/)选择 *Latest Python 3 Release* 拉到最底下选择 *Windows Installer* 下载并安装，这里建议安装到默认的路径即可。

![python-install](../python-install.gif)

**注意！一定要勾选*Add Python to PATH*选项！！！它会把python和pip添加到PATH中，供你在CMD/PowerShell中使用命令行。**

安装完成以后，你可以按下 *WIN+R* 键，输入 *powershell* 打开终端，检查python与pip是否正确安装。

![python-check](../python-check.gif)

## 如果是macOS或者Linux

安装几乎一样，区别在于终端不再是powershell，而是使用Terminal（相信你们能打得开它）来检查安装是否成功喔！

## 使用pip来安装Jupyter以及依赖库

检验完成后（现在你已经会使用powershell了），我们再次打开powershell，复制以下命令开始安装Jupyter以及常用的数据分析依赖库，过程会比较慢（取决于网络和你的电脑运行速度）通常会花费2-10分钟不等。

**注意！最后要耐心等待安装完成，直到能继续输入命令为止。**

```bash
pip install jupyter numpy pandas scipy matplotlib seaborn
```
![pip-install](../pip-install.gif)

## Jupyter Notebook

安装完所有依赖以后，在powershell里输入 *jupyter notebook* 会自动跳转到浏览器。你可以尝试新建一个 *ipynb* 文件，并写一些简单的python代码（shift+enter运行单元格）。当你想要关闭的时候，只需要关掉浏览器，同时关闭powershell即可。

![jupyter-notebook](../jupyter-notebook.gif)

在Jupyter中最重要的概念就是单元格（Cell），它可以是 *python code* 也可以是 *markdown* （什么是markdown？看这里：[Markdown 是什么？](https://keatonlao.gitee.io/a-study-note-for-markdown/introduction/)），单元格被编辑后需要使用shift+enter运行以后才会真正执行这个片段，所以单元格不再是像以前的传统程序一样按顺序执行了，而是按照你手动shift+enter的顺序来执行。（每次修改代码都别忘记运行这个单元格后面所有的单元格们哦！）。

我们可以选中一个单元格，按m键来让它转变成一个markdown单元，按y键来让它变回一个code单元。在markdown单元下，我们甚至可以输入Latex的数学公式。

![jupyter-markdown](../jupyter-markdown.gif)

[*Markdown 语法・简明版*](https://keatonlao.gitee.io/a-study-note-for-markdown/syntax/) 有你需要的所有基础语法和演示。[*Short Math Guide for LATEX*](http://mirrors.sjtug.sjtu.edu.cn/ctan/info/short-math-guide/short-math-guide.pdf) 则是一份很好的快速入门Latex的资料，由美国数学协会编写免费开放给公众（我也收集到了它的中译本，[*一份简短的LATEX数学指南*](../short-math-guide-cn.pdf) 翻译如有侵权请告知我会删去）。

