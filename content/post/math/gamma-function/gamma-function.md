---
title: 从广义二项式定理到Gamma函数
tags: ["math"]
date: 2019-01-14T18:20:42+08:00
lastmod: 2019-01-14T18:20:42+08:00
draft: true
---

## 引言

追溯到高中，那是我们便接触了阶乘和简单的二项式定理。随后直到大学，接触到了微积分。那时的我一直在思考：微积分同二项式定理是否存在关联？

现在，我能够自己回答这个问题。当然，答案是有的。

## 文章内容

* 简单介绍广义二项式定理
* 将其与微积分联系到一起，引入Gamma函数
* 总结Gamma函数的基本特性

## 二项式定理

### 更为自然的表达形式

我们熟知的二项式定理写作：$ \displaystyle (x+y)^n = \sum_{k=0}^n{C_n^k\,x^k\,y^{n-k}} $ ，其中 $\displaystyle C_n^k$ 为组合数。

为了更通用地描述广义二项式定理，我们将组合数换一种新的形式写作：$\displaystyle C_n^k = {n \choose k}$，读作：n choose k。

此时，二项式定理变为：

$$\begin{equation} \displaystyle (x+y)^n = \sum_{k=0}^n {n \choose k} \,x^k\,y^{n-k} = \sum_k^n {n \choose k} \,x^{n-k}\,y^k \end{equation}$$

在物理学中更常见的形式：

$$\begin{equation} \displaystyle (1+x)^n = \sum_{k=0}^n {n \choose k} \,x^k \end{equation}$$

自然的，当指数n为自然数（0和所有正整数），我们知道：

$$\label{coefficient} \begin{equation} \displaystyle {n \choose k} = \frac{n!}{k!(n-k)!} \end{equation}$$

### 当指数为非整数时

当指数n为分数时，我们就面临了几个问题：

1. 二项式定理是否还能继续成立？
2. 分数阶乘的数学定义是什么？
3. 此时应该怎么怎么做计算？

牛顿（Isaac Newton）在1665年将二项式定理推广到了实数域。此时在求和时，有限项变成了无穷级数。事实上，即使在复数域，二项式定理依然成立。

$$\label{generalized} \begin{equation} \displaystyle (1+x)^n = \sum_{k=0}^\infty 1 + nx + \frac{n(n-1)}{2!}\,x^2 + \frac{n(n-1)(n-2)}{3!}\,x^3 + \cdots  \end{equation}$$

进一步考察此式：当n为自然数时，所有 $k>n$ 项的系数均为零。此时无穷级数截止于 $k=n$ 项（包括此项），二项无穷级数退化成有限项，即普通的二项式定理。

广义二项式的无穷级数敛散性会随着x与n的变化而变化。对于式 $(\ref{generalized})$ 的敛散性这里提两种情况：

* $\lvert x \rvert < 1$，对于任何实数和复数n，无穷级数均收敛。
* $\lvert x \rvert > 1$，除非n是一个非负整数，不然无穷级数均发散。

对于 $\lvert x \rvert = 1$ 的情况，详细可以看Wikipedia：[Bionomial series convergence](https://en.wikipedia.org/wiki/Binomial_series#Convergence)。

## Gamma函数

### 阶乘的定义

当我们使用式 $(\ref{coefficient})$ 计算系数时，可以使用分子分母约去相同项来计算。

另一方面，当我们刨根问底想要直接计算时，却产生了这个问题：$\displaystyle \frac{1}{2}! = \,?$ 

不妨从代数方面重新思考阶乘：$\displaystyle n! = n\cdot(n-1)\cdot(n-2)\cdots2\cdot1$，但是它只能适用于n是正整数的情况，这些点都是离散的。然而我们想要找到一个连续函数 $\displaystyle f(x)$ ，使得：

* $$\label{condition-1} \begin{equation} \displaystyle f(1) = 1 \end{equation}$$
* $$\label{condition-2} \begin{equation} \displaystyle f(n) = n \cdot f(n-1) \end{equation}$$

这样，既能使其满足递推关系，又能满足边界条件。

![factorial](../images/factorial.svg)
<center>_(图片参照wikipedia相关条目绘制，如有侵权请及时告知。)_</center>
<center>_(This image was inspired by wikipedia articles, please notify me if any infringement.)_</center>

### Pi函数与阶乘

当我们希望通过组合基础函数得到光滑的解析解，以拟合正整数阶乘的时候，结果却均以失败告终。

19世纪，高斯（Gaussian）给出了一个积分形式，将其命名为Pi函数，能够同时满足式 $(\ref{condition-1})(\ref{condition-2})$ ：

$$\begin{equation} \displaystyle f(x) = \Pi(x) = \int_0^\infty{t^{x}\,e^{-t}\,dt} \end{equation}$$

Pi函数其实也是Gamma函数的一种，它更适合用来表示阶乘，或过度到连续连续函数。需要注意的是，此处引入了一个Dummy Variable: t，积分结果是不显含t的，仅为x的函数。

首先考察式 $(\ref{condition-1})$，可以使用分部积分法（integral by parts）轻松得到结果。其中 $\displaystyle \lim_{x \to \infty}te^{-t} = 0$ 可以通过洛必达法则得到（L'Hopital Rule）：

$$\displaystyle f(1) = \Pi(1) = \int_0^\infty{te^{-t}\,dt} = -te^{-t}\rvert_0^\infty + \int_0^\infty{e^{-t}}\,dt = 1 $$

其次考察式 $(\ref{condition-2})$，同样使用分部积分法，其中的极限可以反复使用洛必达法则依然得到结果零：

$$\displaystyle f(n) = \Pi(n) = -t^ne^{-t}\rvert_0^\infty + \int_0^\infty{e^{-t}}\,d(t^n) = n\int_0^\infty{t^{n-1}e^{-t}}\,dt = nf(n-1) $$

可见，Pi函数能够很好地满足正整数时阶乘的定义。事实上，即便是分数阶乘，也能给出精确解。由此可见，Pi函数能够很自然地代表阶乘：

$$\begin{equation} \displaystyle \Pi(x) = x! = \int_0^\infty{t^{x}\,e^{-t}\,dt} \end{equation}$$

那么，利用Pi函数和高斯积分我们能够回答上面提出的问题：

$$\displaystyle \frac{1}{2}! = \int_0^\infty{\sqrt{t}e^{-t}\,dt} = 2\int_0^\infty{y^2e^{-y^2}\,dy} = \frac{\sqrt{\pi}}{2}$$

### Pi函数与Gamma函数

上一小节中曾提到，Pi函数是Gamma函数的一种（an alternative notation），那么它们之间的关系是什么？

$$\label{gamma} \begin{equation} \displaystyle \Gamma(x) = \Pi(x-1) = \int_0^\infty{t^{x-1}e^{-t}\,dt} \end{equation}$$

### 应用与特性

将在未来写一篇新的文章补充此部分，已经加入待办清单。

* Gamma函数的欧拉反射公式
* Gamma导数及其形式
* 与黎曼Zeta函数的联系
* Gamma函数的傅立叶展开

## 结束语

虽然将广义二项式定理与Gamma函数联系到一起显得有些牵强，但是这也是我在学习量子物理的过程中，真真切切地对自己的无知发出的思考。

面对未知，我们应当去问为什么。这也是我学习的方法论：选择看似困难一些的道路，别被经验所束缚。

希望这篇文章能够帮助到同样拥有好奇心的你。最后，共勉。
