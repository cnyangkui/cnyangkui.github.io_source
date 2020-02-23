---
title: '论文笔记 - RadCloud: Visualizing Multiple Texts with Merged Word Clouds'
date: 2020-02-23 18:45:17
tags:
  - word cloud
  - text visualization
categories: Visualization
mathjax: true
---

Burch M, Lohmann S, Beck F, et al. RadCloud: Visualizing multiple texts with merged word clouds[C]//2014 18th International Conference on Information Visualisation. IEEE, 2014: 108-113.

RadCloud能够同时比较多个文本的内容，采用径向布局将所有词汇放在一个圆圈内，并能够呈现词汇与各个文本类别的相关性。

<!--more-->

## Introduction

词云通常用于可视化文本中的重要词语，它们常常作为深入分析的起点，帮助判断文本的主要内容或是否与特定的信息需求相关。目前大多数词云针对单个文本进行可视化，很少能够同时可视化多个文本并比较它们的词汇和词汇频率。

这篇论文的目标是比较多个文本的内容，每个词汇只出现一次，而且词汇与每个文本类别的相关性也要在视觉上显示出来。

## RadCloud Visualization

### Design Decisions

1. 词云采用径向布局将所有词汇放在一个圆圈内，圆弧的颜色对应文本类别。
2. 每个词汇只出现一次，字体大小编码该词汇在所有类别中的最大相关性，颜色来源于最大相关性所在的类别。
2. 词汇下方添加一个堆叠的柱状图，以表示它与每个类别的相关性。

![33Alh8.png](https://s2.ax1x.com/2020/02/23/33Alh8.png)

### Intended Word Positions

使用一种新的 springs-based layout algorithm。

假设文本可以划分成一组类别：$C:=\lbrace c_1, ..., c_n \rbrace$

所有词汇的集合可以表示为：$W:=\bigcup_{c \in C} W_c$

计算每个文本类别中词汇的权重，$weight(word_i, category_c)$如何计算讲得不是很清楚，我认为可以用tf或tf-idf表示：

$$
w{^c_i} =
\begin{cases}
0,  & \text{if $w_i \notin W_c$} \\
weight(word_i, category_c) \in R, & \text{if $w_i \in W_c$}
\end{cases}
$$

标准化权重：

$$ 
w{^{'c}_i} := \frac {w{^c_i}} {\sum_{k=1}^{|W|}w{^c_k}} 
$$

词云设计成圆形形状，这个圆围绕着坐标系的原点，每个类别在圆的边上均匀分布。然后通过从圆的顶部开始绕圆心旋转点来获得各个类别的位置。从原点到类别点的向量表示为： $V_c := \overrightarrow{position(category_c)} $

词汇在各个类别所在方向上的权重：
$$
w{^{''c}_i} := \frac {w{^{'c}_i}} {\sum_{k=1}^{|C|}w{^{'k}_i}} 
$$

词汇的位置：
$$
position(word_i) := \sum_{c=1}^{|C|}(V_c \times w_i^{''c})
$$

### Corrected Word Positions

此时的布局结果中，词汇可能会出现重叠。使用Springs建模词汇边界框间的距离，当两个词汇重叠时使其远离彼此少许。但是词汇相对于圆心的位置在语义上是相关的，因此坐标位置不能改变太大。

![33FDaV.png](https://s2.ax1x.com/2020/02/23/33FDaV.png)

## Conclusion

1. 这是一个可视化不同类别文档/文档集合的词云。每个词汇只在词云中出现一次，用户可以查看词汇与每个类别的相关性。
2. 词汇位置是根据它与每个文本类别的相关性来决定的。为了避免词汇重叠，使用Springs布局纠正了词汇的位置，但尽量减少移动词汇。
3. 词汇位置是一个指标，词汇位置越靠近某一类别，则与该类别的相关性越大；词汇下面的堆叠柱状图也用来可视化词汇与每一类文档的相关性，颜色表示不同的文本类别。