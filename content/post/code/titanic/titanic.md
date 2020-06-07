---
title: "从泰坦尼克号谈到浅层神经网络"
tags: ["code", "machine-learning"]
date: 2020-06-04T15:39:13+08:00
lastmod: 2020-06-07T16:13:11+08:00
author: "qwezarty"
draft: false
---

## 前言

最近因为课题需要，捡起了很多年前学了但是没怎么应用的机器学习。作为一个经典的算法，逻辑回归目前被作为神经网络的基础，有了广泛的应用。单层单节点的逻辑回归在这个问题上表现较为良好（~78%准确度）。本文的在线示例是用大约600条历史上泰坦尼克号上的搭乘者->生还/死亡，喂给一个3层的神经网络训练而成。对于以下细节，你或许会感兴趣：

- 数据集：训练样本627条，测试样本264条
- 准确度：训练集88%，测试集83%，尚未进行超参数优化（来不及做了）
- 模型状况：三层的浅层神经网络，节点数分别为：128(ReLU), 64(ReLU), 1(Sigmoid)
- 源码：[qwezarty/machine-learning-examples](https://github.com/qwezarty/machine-learning-examples)

注：在线演示的参数验证并不严格，还请大家手下留情不要用表单工具提交一些奇怪的值（要是把AI玩坏了，她会来找你算账的），另外为了简化输入，我使用下拉选项替代了许多的文本框。好了，现在大家就可以由观看《泰坦尼克号》的经验，有序上船，会不会翻全看造化啦！

## 在线示例

{{< rawhtml >}}

<form id="titanicForm">
	<label for="sex">性别（只接受男/女噢）：</label>
	<select id="sex" name="sex">
	  <option value="0">男</option>
	  <option value="1">女</option>
	</select> <br> <br>

	<label for="age">年龄（5-80之间都很合理）：</label>
	<select id="age" name="age">
	  <option value="5">5</option>
	  <option value="10">10</option>
	  <option value="15">15</option>
	  <option value="20">20</option>
	  <option value="21">21</option>
	  <option value="22">22</option>
	  <option value="23">23</option>
	  <option value="24">24</option>
	  <option value="25">25</option>
	  <option value="26">26</option>
	  <option value="27">27</option>
	  <option value="28">28</option>
	  <option value="29">29</option>
	  <option value="30">30</option>
	  <option value="35">35</option>
	  <option value="40">40</option>
	  <option value="45">45</option>
	  <option value="50">50</option>
	  <option value="60">60</option>
	  <option value="70">70</option>
	  <option value="80">80</option>
	</select> <br> <br>

	<label for="class">船票等级：</label>
	<select id="class" name="class">
	  <option value="0">三等舱</option>
	  <option value="1">一等舱</option>
	  <option value="2">二等舱</option>
	</select> <br> <br>

	<label for="fare">船票费用：</label>
	<select id="fare" name="fare">
	  <option value="0">0（偷渡的或工作人员）</option>
	  <option value="5">5</option>
	  <option value="10">10</option>
	  <option value="15">15</option>
	  <option value="30">30</option>
	  <option value="50">50</option>
	  <option value="100">100</option>
	  <option value="200">200</option>
	</select> <br> <br>

	<label for="deck">船舱位置：</label>
	<select id="deck" name="deck">
	  <option value="0">我也不清楚</option>
	  <option value="1">C</option>
	  <option value="2">G</option>
	  <option value="3">A</option>
	  <option value="4">B</option>
	  <option value="5">D</option>
	  <option value="6">F</option>
	  <option value="7">E</option>
	</select> <br> <br>

	<label for="embark_town">登船港口：</label>
	<select id="embark_town" name="embark_town">
	  <option value="0">南安普敦, Southampton</option>
	  <option value="1">瑟堡, Cherbourg</option>
	  <option value="2">皇后镇，Queenstown</option>
	  <option value="3">我也不清楚</option>
	</select> <br> <br>

	<label for="alone">是否单独一人：</label>
	<select id="alone" name="alone">
	  <option value="0">否</option>
	  <option value="1">是</option>
	</select> <br> <br>

	<label for="n_siblings_spouses">登船的配偶及亲兄弟/姐妹人数：</label>
	<select id="n_siblings_spouses" name="n_siblings_spouses">
	  <option value="1">1</option>
	  <option value="2">2</option>
	  <option value="3">3</option>
	  <option value="4">4</option>
	  <option value="0">0</option>
	</select> <br> <br>

	<label for="parch">登船的父母/子女人数：</label>
	<select id="parch" name="parch">
	  <option value="0">0</option>
	  <option value="1">1</option>
	  <option value="2">2</option>
	  <option value="3">3</option>
	  <option value="5">5</option>
	</select> <br> <br>

	<button>来吧，决定命运的时候到了！</button>
</form>

<div id="result" style="display: none;">
	<p>本AI认为你大约有<label id="propability"></label>%的几率生还。
</div>

{{< /rawhtml >}}

<script>
document.forms[0].onsubmit = async(e) => {
	e.preventDefault(); // prevent reload page
	const params = new FormData(document.getElementById('titanicForm'));
	fetch("https://atomlab.org/forms/titanic", { method:"POST", mode:"cors", body:params })
		.then(res =>  res.json() ).then(data => {
			document.getElementById('propability').innerHTML = data["propability"];
			document.getElementById('result').style.display = 'block';
		});
}
</script>

## 这个模型有什么用？

这是一个典型的“疾病与风险控制”示例，想象一下把数据源替换成“良性/恶性肿瘤与各特征的关系”，例如肿瘤的三维尺寸、发病时长、患者的性别/年龄/遗传疾病史等等，如果你有几百条这样的数据，在良好的调优的前提下，你甚至能达到90%以上的准确度（甚至超过了那些经验丰富的医师，所以有言论说是AI的出现会使医生失业，并不是空穴来风）。

此外，Andrew Ng 也曾经分享了他为某家航空公司，预测飞机引擎在本次飞行中出现的故障率，用的也是纯粹的神经网络逻辑回归模型，结合关键位置的传感器信息，能够降低空难/减少检修次数等等。

由此可见，在机器学习中数据与模型同样重要。与单层单节点的逻辑回归（代价函数为凸函数，仅有一个最优解）不同的是，神经网络能够随着样本数量的提升不断有效地提高准确度，所以在更多的样本的情况下，模型的准确度也能进一步提高。

*P.S. 如果你对“为什么”不感兴趣，那本文到这里就已经结束了。*

## 数据集概况与模型训练

![feature distributions](../images/distributions.png)

以上四张图分别为训练集中年龄、船费、性别和船票等级的整体数据分布。在拿到一个机器学习任务时，首先对数据的整体感觉是十分重要的。图中我们不难看出登船的大部分都是年轻人（20-35），并且男性比女性多了一倍的数量，大部分人的船费仅在0-40刀之间。

![% of survive, vs, features](../images/survivors.png)

存活率方面，我只绘出了四张图，它们依次是：性别、是否单独一人登船、船票等级和登船的配偶/兄弟姐妹数量。可以看到，女性的存活率比男性高出了四倍，是首要的影响因素（历史上船长爱德华·约翰·史密斯在最后的时刻下命令，先让妇女和儿童上救生艇。而电影中，Rose最后也是在救生艇上。）此外，一等舱的存活率比三等舱高出了近三倍，可见富裕的资源却是能让你更好地活下来（贫穷人流下了悔恨的泪水。）

关于泰坦尼克号的数据分析，网上其实有很多（做数据分析的人）有兴趣的同学可以进一步看下这篇文章：[泰坦尼克号乘客数据分析](https://zhuanlan.zhihu.com/p/26440212)。看过人工对真实数据的分析以后，你可以再次调戏一下AI，结合两者不难发现AI的准确度还是相当的高。

![cost, vs, # of iterations](../images/costs.png)

这是代价函数随着迭代次数的改变，可以明显看到相比于普通的逻辑回归（代价函数为凸函数），浅层的神经网络已经有了持续优化的能力：在10k次迭代以后依然能够持续地提高训练集的准确度。然而，这会造成 [over-fitting problem](https://en.wikipedia.org/wiki/Overfitting) 所以我们这里仅仅选取了12k次迭代训练。

![accuracy](../images/accuracy.png)

最后，这是训练完毕以后的准确度，训练集～89%，测试集～83%，符合我们文章一开始的期待值。另外我们还可以在对训练数据进行清洗，如扩充样本、缺失值处理等等，以及在设计浅层神经网络时进行超参数调优，以进一步提高准确度。（偷懒，这个工作以后看情况补上）

## 神经网络到底在做什么？

（瞎扯淡预警）

回忆起最一开始的平面线性拟合问题，比如有10个数据点基本散布在某条直线附近，你想要去找到这条平面直线去最大程度上的拟合这些点。那么最容易想到的就是假想现在有某个斜率+截距的直线，计算所有这些点到直线距离的平方和，你要做的就是找到那条令这个平方和最小的直线（至此已经变成了一个纯数学问题。）

现在，我们把这个问题进一步扩展一下：考虑三维空间的点，现在有了100个点分布在空间中，你要找到的是某个三维曲面尽量更多地包围住那些在泰坦尼克号上生还的人（survive=1的点），可以想见如果你的自由参数越多（神经网络的层数/节点数），那么你这个曲面能够变化的形状就越奇怪，越能够拟合这些数据点。

实际上在我们的泰坦尼克号问题中，总共有九个特征维度，这些点分布在九维空间里，每个点都有数值（0代表死亡，1代表生还）。那么这些生还的数据点，肯定有某种内在的分布规律（比如集中在某一小块区域附近），你的任务就是在九维空间里找到一个曲面，以更大程度上地包围住这些生还者的点。

当我思考到这些的时，确实让我有点震惊：**人类在用有限的知识和能力，去理解和感知突破肉体极限的东西。**

## 写在最后

其实近些年来，“神经网络”和“人工智能”已经被资本市场炒烂了，如果你完完整整看完了通篇文章，那么不难看出其实“神经网络”和人类大脑的神经元没有太大关系（现有的科学甚至都没法理解神经元是如何工作的），“人工智能”一点儿也不智能：**我们只是在给定的条件下，求解一个数学问题的最优解。**

