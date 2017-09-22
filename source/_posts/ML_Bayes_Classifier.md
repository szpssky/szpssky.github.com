---
title: 《机器学习》笔记 —— 贝叶斯分类器
date: 2017-09-22 14:00:00
categories:
- Meachine Learning
tags:
- Meachine Learning
comments: true
mathjax: true
---

### 贝叶斯决策论

假设有N种可能的类别标记，即$y=\{c_1,c_2,\dots ,c_N\}$，$\lambda_{ij}$是将一个真实标记为$c_i$的标记误分为$c_i$所产生的损失，基于后验概率$p(c_i|x)$可获得将样本$x$分类为$c_i$所产生的期望损失，即在样本x上的条件风险：
$$
R(c_i|x)=\sum_{j=1}^N \lambda_{ij}p(c_j|x)
$$

目标是寻找判定准则h:$\mathcal{X} \rightarrow \mathcal{Y}$以最小化总体风险。
$$
R(h) = \mathbb{E}_x[R(h(x)|x)]
$$

若h最小化条件风险$R(h(x)|x)$，则总体风险$R(x)$也将被最小化。
<!-- more -->

贝叶斯判定准则：为最小化总体风险，只需在每个样本上选择那个能使条件风险$R(c|x)$最小的类别标记，即：
$$
h^*(x)=\arg\min_{c \in \mathcal{y}}R(c|x)
$$
$h^*$为贝叶斯最优分类器，$R(h^*)$称为贝叶斯风险，$1-R(h^*)$反应分类器能达到的最好性能，即精度理论上限。

具体的，若目标是最小化分类错误率，则误判损失$\lambda_{ij}$：
$$\lambda_{ij} = \left\{
\begin{array}{rl}
& 0  \qquad  \text{if }i=j  \\
& 1   \qquad  \text{otherwise}\\
\end{array} \right. $$

此时条件风险：
$$
R(c|x)=1-p(c|x)
$$

最小化分类错误率的贝叶斯分类器：
$$
h^*(x)=\arg\max_{c\in \mathcal{y}}p(c|x)
$$
即对每个样本，选择能使其后验概率$p(c|x)$最大的类别标记。

机器学习实现基于有限的训练样本集尽可能准确地估计出后验概率$p(c|x)$，两种策略：

(1)给定x可通过直接建模$p(c|x)$来预测c

(2)先对联合概率分布$p(x,c)$建模，再由此获得$p(c|x)$

(1)为判别式模型，(2)为生成式模型

常见判别式模型：逻辑回归、SVM、线性回归、神经网络、条件随机场CRF、线性判别分析LDA、Boosting等。

常见生成式模型：高斯混合模型、隐马尔可夫模型、信念网络、朴素贝叶斯、贝叶斯网络、主题生成模型LDA、马尔可夫随机场等。

对生成模型比如考虑：
$$
P(c|x)=\frac{P(x,c)}{P(x)}
$$

基于贝叶斯定理：
$$
P(c|x)=\frac{P(c)P(x|c)}{P(x)}
$$

$P(c)$为类先验概率表达了各类样本所占比例，当训练集包含充足的独立同分布样本时$P(c)$可通过各类样本出现频率来估计，$P(x|c)$为样本x相对于类标记c的类条件概率（似然），$P(x|c)$涉及x所有属性的联合概率，通常存在未被观测到的情况，所以无法通过频率来估计，只能通过“极大似然估计”，$P(x)$为证据因子，与类标签无关。

### 极大似然估计

估计类条件概率的一种常见策略是先假定其具有某种确定的概率分布形式，再基于训练样本对概率分布参数进行估计。

令$D_c$表示训练集D中第c类样本组成的集合，假设这些样本是独立同分布的则参数$\theta_c$对于数据集$D_c$的似然是：
$$
P(D_c|\theta_c)=\prod_{x \in D_c}P(x|\theta_c)
$$

对$\theta_c$进行极大似然估计寻找能最大化似然$P(D_c|\theta_c)$的参数值$\hat \theta_c$，连乘操作容易下溢，通常用对数似然：
$$
\begin{align}
LL(\theta_c) &= \log P(D_c|\theta_c) \\
&= \sum_{x \in D_c}\log P(x|\theta_c)
\end{align}
$$

参数$\theta_c$的极大似然估计$\hat \theta_c$：
$$
\hat \theta_c = \arg\max_{\theta_c} LL(\theta_c)
$$

注意：这种参数化的方法虽然使类条件概率估计变得相对简单，但估计结果严重依赖于所假设的概率分布形式是否符合真实的数据分布。

### 朴素贝叶斯分类器

$P(x|c)$是所有属性上的联合概率，难以从有限样本直接估计（因为在计算上将会遭遇组合爆炸，在数据上将会遭遇样本稀疏且属性越多问题越严重），为了避开这个问题，朴素贝叶斯分类器采用“属性条件独立性假设”（对已知类别，假设所有属性相互独立）。

基于该假设：
$$
\begin{align}
P(c|x)&= \frac{P(c)P(x|c)}{P(x)} \\
&= \frac{P(c)}{P(x)}\prod_{i=1}^dP(x_i|c)
\end{align}
$$
其中d表示属性数目，$x_i$表示x在第i个属性上的取值。由于所有类别$P(x)$相同，因此朴素贝叶斯表达式为：
$$
h_{nb}(x)=\arg\max_{c \in \mathcal{Y}} P(c)\prod_{i=1}^dP(x_i|c)
$$

令$D_c$表示训练集D中第c类样本组成的集合，若有充分的独立同分布样本，则容易估算出类先验概率$P(c)$：
$$
P(c)=\frac{|D_c|}{|D|}
$$

令$D_{c，x_i}$表示$D_c$中第i个属性取值为$x_i$的样本组成的集合，则：
$$
P(x_i|c) = \frac{|D_{c,x_i}|}{|D_c|}
$$

对连续属性：假设$P(x_i|c)\sim\mathcal{N}(\mu_{c,i},\sigma^2_{c,i})$，则：
$$
P(x_i|c)=\frac{1}{\sqrt{2\pi}\sigma_{c,i}}exp(-\frac{(x_i-\mu_{c,i})^2}{2\sigma^2_{c,i}})
$$

若某个属性值在训练集中没有与某个类同时出现过，则直接进行概率估计将出现问题，连乘式概率为0，为避免其他属性的信息被“抹去”，需要进行“平滑”。

拉布拉斯修正：

令N表示训练集D中可能的类别数，$N_i$表示第i个属性可能的取值数则：
$$
\begin{align}
\hat P(c)&=\frac{|D_c|+1}{|D|+N} \\
\hat P(x_i|c)&=\frac{|D_{c,x_i}|+1}{|D_c|+N_i}
\end{align}
$$

### 半朴素贝叶斯分类器

“属性条件独立性假设”在现实中往往很难成立，对该假设进行一定程度放松则得到“半朴素贝叶斯分类器”

“独依赖估计”是半朴素贝叶斯分类器最常用的策略，即假设每个属性在类别之外最多依赖于一个其他属性，即：
$$
P(c|x) \propto P(c)\prod_{i=1}^dP(x_i|c,pa_i})
$$

其中$pa_i$为$x_i$所依赖的属性，称为$x_i$的父属性，若$pa_i$已知，则可估计概率值$P(x_i|c,pa_i)$。

1.SPODE：假设所有属性都依赖于同一个属性，称为超父，然后通过交叉验证等方式确定超父属性。

2.TAN：在最大带权生成树的基础上通过以下步骤将属性依赖简约为周志华机器学习P115所示的树形结构。

(1) 计算任意两个属性之间的条件互信息。$I(x_i,x_j|y)=\sum_{x_i,x_j;c\in \mathcal{Y}}P(x_i,x_j;|c)\log\frac{P(x_i,x_j|c)}{P(x_i|c)P(x_j|c)}$

(2) 以属性为结点构建完全图，任意两个结点之间边的权重为$I(x_i,x_j|\mathcal{Y})$

(3) 构建此完全图的最大带权生成树，挑选根变量将边置为有向。

TAN仅保留了强相关属性的依赖性

3.AODE：基于集成学习，更强大的独依赖分类器，尝试将每个属性作为超父构建SPODE，然后将具有足够数据支撑的SPOE集成起来：
$$
P(c|x) \propto \sum_{\substack{i=1 \\ |D_{x_i}| \geq m'}}^d
$$

其中$D_{x_i}$是在第i个属性上的取值为$x_i$的样本集合，$m'$为阈值常数，AODE需估计$P(c,x_i)$和$P(x_j|c,x_i)$：
$$
\begin{align}
\hat P(c,x_i)&=\frac{|D_{c,x_i}|+1}{|D|+N_i} \\
\hat P(x_j|c,x_i) &=\frac{|D_{c,x_i,x_j}|+1}{|D_c,x_i|+N_j}
\end{align}
$$
若使用高阶依赖，则当训练数据非常充分，泛化性能有可能提升，但在有限样本下则又陷入估计高阶联合概率分布的泥泽。






