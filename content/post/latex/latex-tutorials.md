---
title: "献给科研工作者的latex新手指南"
tags: ["latex"]
date: 2020-01-06T19:45:12+08:00
lastmod: 2020-01-06T19:45:12+08:00
author: "qwezarty"
draft: true
---

## 简介

在日常文本编辑中，我最常用的两种就是markdown与latex。前者以轻量、简洁、无需编译，纯文本的特性具有高度兼容性，基本上若是没有太高强度的公式、交叉引用和严厉的格式要求的话，我都会选用markdown。剩下的所有情况，latex是我唯一的选择（没错，就是写paper专用），其实对于科研工作者而言，也仅能够使用它。

这里不再阐述Latex的优点了，这里我们会以*Physics Review*期刊提供的revtex包为例直接进入主题，我会假定你拥有基础的计算机操作知识，主要介绍一些语法规则以及推荐采用的编辑策略和工具。（另外虽然我是以macos系统讲解的，不过文中大部分内容都能够很好地适用于linux/win）

## 工具

相比较TexLive这样动不动就4G这样庞大的安装包，如果你没有编辑中文的要求并且能够随时上网，我会更推崇使用[MiKTex](https://miktex.org/download)。它足够小巧并且支持全平台，在macos下甚至仅仅只有50mb，当编译时若检测到没有安装相应的包时会自动下载，而且不需要担心下载速度因为支持镜像加速。MiKTex的操作比较傻瓜，打开后你稍微倒腾下就会明白了，不再展开讲，如果有遇到没法解决的问题，请写邮件给我。

其次是编辑器，这里我推荐两个，第一个是推荐大多数人使用的[vscode](https://code.visualstudio.com)。它是我经常会在调试时使用的编辑器，也足够的简单并且有中文化支持，安装完成后你需要打开vscode安装一个扩展：LaTeX Workshop。安装完成以后重启vscode，随便打开一个tex文件你会看到侧边栏多出了一个TEX，我们需要增加一个编译食谱并设为默认编译项：xelatex->bibtex->xelatex->xelatex（标准的四次编译）。现在，你拥有了一个能够补全、智能提示和快捷编译（macos下是command+alt+b）的编辑器，开始起飞吧！

第二个编辑器就是vim了（我的最爱），同样它也有plugin能支持高亮、补全、提示和快捷编译，这里就不再展开了（会用vim相信你也能自己配置好了）。

## 结构

latex是一种需要编译的标记语言，比较简单的结构如下。

```latex
\documentclass[a4paper,12pt]{article} % document layout

\usepackage{amsmath} % core math
\usepackage{amssymb} % math symbols
\usepackage{mathtools} % beautify amsmath

\title{A First Look to Latex}
\author{qwezarty}

\begin{document} % main content inside
\maketitle % required, making title

\section{Part. 1}
\section{Part. 2}

% only required when \cite{} being used
% it demands a file.bib stays at same directory
% of your file.bib file
% \bibliography{file} 

\end{document}
```

## 其他

### 拼写检查

博主使用的拼写检查工具是[aspell](https://en.wikipedia.org/wiki/GNU_Aspell)，它的开发初衷是为了完全替代ispell，相较后者而言，aspell能够支持UTF-8的编码格式。另外可选用拼写检查工具请的参看[Language checking](https://wiki.archlinux.org/index.php/Language_checking). 

使用也非常简单，就不展开来讲了，这里具体参数的意思你可以使用"man aspell"来查看更多的帮助。之后你就能看到一个交互式的shell，使用相应的数字和字母来替换/更改即可，非常容易。

```bash
aspell -t -c file.tex
```

