---
title: 《Deep Learning》笔记 —— 卷积神经网络
date: 2017-09-01 20:00:00
categories:
- Deep Learning Book
tags:
- Deep Learning
comments: true
mathjax: true
---

卷积网络是一种专门用来处理具有类似网格结构的数据的神经网络，例如时间序列数据(可以认为是在时间轴上有规律地采样形成的一维网格)和图像数据(二维像素网格)，卷积网络在诸多应用领域都表现优异。卷积网络是指那些至少网络的一层中使用卷积运算来代替一般乘法运算的神经网络。

### 卷积运算

在通常形式中，卷积是对两个实变函数的一种数学运算。卷积运算通常用星号表示：
$$
s(t)=(x*w)(t)
$$

卷积的第一个参数通常叫作输入，第二个参数叫作核函数，输出有时被称作特征映射。

在机器学习应用中，输入通常是多维数组的数据，而核通常是由学习算法优化得到的多维数组参数。如果把一张二维图像I作为输入，也使用一个二维的核K：
$$
S(i,j)=(I*K)(i,j)=\sum_m\sum_n I(m,n)K(i-m,j-n)
$$
<!-- more -->


卷积是可交换的，所以上式等价于：
$$
S(i,j)=(K*I)(i,j)=\sum_m\sum_n I(i-m,j-n)K(m,n)
$$

卷积运算可交换性是将核相对输入进行了翻转，将核翻转的唯一目的是实现可交换性，尽管可交换性在证明时有用，但在神经网络的应用中却不是一个重要的性质。

### 动机

卷积运算通过三个重要的思想来帮助改进机器学习系统：稀疏交互、参数共享、等变表示。

传统神经网络使用矩阵乘法来建立输入与输出的连接关系，其中，参数矩阵中每个单独的参数都描述了一个输入单元与一个输出单元间的交互，然后卷积网络具有稀疏交互(稀疏连接)，通过使核大小远小于输入的大小来达到这个特点。当处理一个图像时通过只使用几十到几百个像素点的核来检测一些小的有意义的特征，例如图像的边缘。尽管神经单元的直接连接都是稀疏的，但是在深度卷网络中，处在深层的单元间接地连接到全部或大部分的输入。

在卷积网络中，核的每个元素都作用在输入的每一个位置上，卷积运算中的参数共享保证了只需要学习一个参数集合，而不是对于每一个位置都需要学习一个单独的参数集合，因此卷积在存储和统计效率方面极大地优于稠密矩阵的乘法运算。

对于卷积，参数共享的特殊形式使得神经网络层具有平移等变的性质，如果一个函数满足输入改变，输出也以同样的方式改变着一个性质，则称这是等变的。

### 池化

池化函数使某一位置相邻输出的总体统计特征来代替网络在该位置的输出，例如最大池化函数给出相邻矩形区域内的最大值，其他常用的池化函数包括相邻区域内的平均值、$L^2$范数以及基于距中心像素距离的加权平均函数。

当输入做出少量平移时，经过池化函数后的大多数输出并不会发生改变。局部平移不变性是一个非常有用性质，尤其是当我们只需要关心某个特征是否出现而不需要关心它出现的具体位置时。

使用池化可以看做增加了一个无限强的先验：这一层学得的函数必须具有对少量平移的不变性。当这个假设成立时，池化可以极大提高网络的统计效率。

### 卷积与池化作为一种无限强的先验

卷积的使用相当于对网络中的一层的参数引入一个无限强的概率分布先验证：一个隐藏层单元的权重必须和它的邻居的权重相同，但可以在空间上移动，即该先验说明该层应该学得的函数只包含局部连接关系并且对平移具有等变性。

池化也是一个无限强的先验：每一个单元都具有少量平移不变性。

卷积和池化可能导致欠拟合，卷积和池化只有当先验的假设合理且正确时才有用，如果一项任务依赖于保存精确的空间信息，那么在所有特征上使用池化将增大训练误差。

### 基本卷积函数的变体

非共享卷积：

在一些情况下，并不是真的想使用卷积，而是想用一些局部连接的网络层，在这种情况下多层感知机对应的邻接矩阵是相同，但每一个连接都有它自己的权重。因为它和具有一个小核的离散卷积运算很像，但并不跨位置来共享参数。

平铺卷积：

对卷积层和局部连接层进行折中，这里并不是对每一个空间位置的权重集合进行学习，而是学习一组核使得在空间移动时它们可以被循环利用，这意味着在近邻的位置上拥有不同的滤波器，就像局部连接层一样，但是对于参数的存储需求仅仅会增长常数倍，如下图对局部连接、平铺卷积和标准卷积进行比较：

{% asset_img compare.png %}

***
现在仍不清楚为什么卷积网络在一般的反向传播网络被认为已经失败时反而成功了，这可能可以简单的归结为卷积网络比全连接网络计算效率更高，因此使用它们运行多个实验并调整它们的实现和超参数更容易，更大的网络也似乎更容易训练。

卷积网络提供了一种方法来特化神经网络，使其能够处理具有清楚的网格结构拓扑的数据，以及将这样的模型扩展到非常大的规模，这种方法在二维图像拓扑上是最成功的。














