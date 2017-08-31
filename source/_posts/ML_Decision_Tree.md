---
title: 《机器学习》笔记 —— 决策树
date: 2017-08-31 19:00:00
categories:
- Meachine Learning
tags:
- Meachine Learning
comments: true
mathjax: true
---

### 基本流程

一般的，一棵决策树包含一个根结点，若干个内部结点和若干个叶结点，叶结点对应于决策结果，其他每个结点对应于一个属性测试，每个结点包含的样本集合根据属性测试的结果被划分到子结点中，从根结点到叶结点的路径对应了一个判断测试序列。

决策树学习基本算法:

```
输入：训练集D={(x1,y1),(x2,y2),...,(xm,ym)}
	 属性集合A={a1,a2,...,ad}
过程：
	TreeGenerate(D,A)
	生成结点node
	if D中样本全属于同一类C then
		将node结点标记为C类叶结点；return
	end if
	if A=null或D中样本在A上取值相同 then
		将node标记为叶结点，其类别标记为D种样本最多的类；return
	end if
	从A中选择最优划分属性a*
	for a* 中每一个值a*^ do
		为node生成一个分支:令D_v表示D中在a*上取值为a*^的样本子集
		if D_v=null then
			将分支标记为叶结点，其类别为D中样本最多的类；return
		else
			以TreeGenerate(D_v,A\{a*})为分支结点
		end if
	end

```
<!-- more -->

### 划分选择

#### 信息熵增益

信息熵：度量样本集合纯度最常用的一种指标，假定样本集D中第k类样本所占比例为$p_k(k=1,2,\dots,|y|)$，则D的信息熵：
$$
Ent(D)=-\sum_{k=1}^{|y|}p_k\log_2p_k
$$

Ent(D)越小，D的纯度越高。

假定离散数学a有v个可能的取值${a^1,a^2,\dots ,a^v}$，若使用a来对样本集D划分，则会产生v个结点，第v个结点包含样本D中所有在属性a上取值为$a_v$的样本记为$D^v$，根据不同分支包含的样本数不同，给分支结点赋权重$\frac{|D^v|}{|D|}$，则信息熵增益为：
$$
Gain(D,a)=Ent(D)-\sum_{i=1}^v\frac{|D^v|}{|D|}Ent(D^v)
$$
Gain(D,a)越大，以a划分，纯度提升越大，故划分选择：
$a_*=\arg\max_{a\in A}Gain(D,a)$

#### 增益率

信息熵增准则对可取数目较多的属性有所偏好，为减少这种偏好可能的不利影响，C4.5算法使用增益率来选择最优划分属性：
$$
Gain\_radio(D,a)=\frac{Gain(D,a)}{IV(a)}
$$

其中$IV(a)=-\sum\limits_{v=1}^V\frac{|D^v|}{|D|}\log_2 \frac{|D^v|}{D}$,称为属性a的固有值，属性a的取值数目越多(v越大)，则IV(a)越大。

增益率对数目较少的属性有所偏好，因此C4.5并不直接选取增益率最大的，而是使用一个启发式方法，先从候选划分属性中找出信息熵高于平均水平的属性，再选增益率最高的。

### 基尼指数
CART决策树使用"基尼指数"：
$$
\begin{align}
Gini(D) & = \sum_{k=1}^{|y|}\sum_{k'\neq k}p_kp_{k'}
& = 1-\sum_{k=1}^{|y|}p_k^2
\end{align}
$$
则对属性a，
$$
\text{Gini_index}(D,a)=\sum_{i=1}^|v|\frac{|D^v|}{|D|}Gini(D^v) \\
a_* = \arg\min_{a \in A}\text{Gini_index}(D,a)
$$

### 剪枝处理

决策树对方“过拟合”的主要手段。

#### 预剪枝

决策树生成过程中对每个结点在划分前先进行估计，若当前结点的划分不能带来决策树泛化性能提升，则停止划分并将当前结点标记为叶结点。

预剪枝使得决策树的很多分支都没有展开，这不仅降低了过拟合的风险，还减少了训练期间开销和测试时间，但有些分支的当前划分虽不能提升泛化性能，但在其基础上进行后续划分却可能提升泛化性能，其基于“贪心”本质禁止分支展开，故有欠拟合风险。

#### 后剪枝

先生成一颗完整决策树，再自底向上对非叶结点进行考察，若将该结点对应的子树替换为叶结点能提升性能则进行替换。

优点：欠拟合风险很小，泛化性能优于预剪枝决策树

缺点：训练时间开销比未剪枝和预剪枝决策树都要大很多

#### 连续值处理(连续属性离散化)

C4.5决策树处理策略：二分法

对样本集D和连续属性a，假的a有n个取值，将这些值进行排序，记为${a^1,a^2,\cdots,a^n}$，基于划分点$t$可将D划分为$D_t^-$和$D_t^+$，对相邻属性取值$a^i$和$a^{i+1}$，t在$[a^i,a^{i+1})$中任意值所产生的划分结果相同。

考察包含n-1个元素的候选划分点集合：
$$
T_a=\bigg \{\frac{a^i+a^{i+1}}{2}\bigg| 1 \leq i \geq n-1 \bigg\}
$$

即，
$$
\begin{align}
\text{Gain}(D,a) & =\max_{t\in T_a}\text{Gain}(D,a,t) \\
& = \max_{t \in T_a}\text{Ent}(D)-\sum_{\lambda \in \{-,+\}}\frac{|D_t^\lambda|}{|D|}\text{Ent}(D_t^\lambda)
\end{align}
$$

### 缺少值处理

需要解决两个问题：

(1) 如何在属性缺失的情况下进行划分属性选择

给定训练集D合属性a，令$\tilde{D}$表示D中在a上没有缺失值的子集，假设属性a有$v$个可能取值$\{a^1,a^2,\cdots,a^v \}$，令$\tilde{D}^v$表示$\tilde{D}$中在属性a上取值为$a^v$的样本子集，$\tilde{D}_k$表示$\tilde{D}$中属于第k类$(k=1,2,\cdots,|y|)$的样本子集，则$\tilde{D}=U_{k=1}^{|y|}\tilde{D}_k$，$\tilde{D}_{v=1}^V\tilde{D}^v$，假定对每个样本赋权重$w_x$，

定义：
$$
\rho = \frac{\sum_{x\in \tilde{D}}w_x}{\sum_{x\in D}w_x} \\
\tilde{p}_k = \frac{\sum_{x \in \tilde{D}_k}w_x}{\sum_{x \in \tilde{D}}w_x} \\
\tilde{\gamma}_v = \frac{\sum_{x\in \tilde{D}}w_x}{\sum_{x \in \tilde{D}}w_x}
$$

$\rho$表示无缺失值样本所占比例

$\tilde{p}_k$表示无缺失值样本中第k类所占比例

$\tilde{\gamma}_v$表示无缺失值样本中在属性$a$上取值$a^v$的样本所占比例

则，$\sum_{k=1}^{|y|}\tilde{p}_k=1$，$\sum_{v=1}^v \tilde{\gamma}_v=1$

将信息熵增益推广：
$$
\begin{align}
text{Gain}(D,a) & =\rho \times \text{Gain}(\tilde{D},a)
& = \rho \times (\text{Ent}(\tilde{D})-\sum_{v=1}^V\tilde{\gamma}_v \text{Ent}(\tilde{D}^v))
\end{align}
$$

其中
$$
\text{Ent}(\tilde{D})=-\sum_{k=1}^{|y|}\tilde{p}_k\log_2 \tilde{p}_k
$$

(2) 给定划分属性，若样本在该属性缺失，如何对属性进行划分

若样本$x$在划分属性$a$上的取值已知，则将$x$划分入与其取值对应子结点，且样本权值在子结点中保持为$w_x$，若样本$x$在划分属性$a$上取值未知，则$x$同时划入所有子结点且样本权值在与属性$a^v$对应的子结点中调整为$\tilde{r}_vw_x$，直观上让同一样本以不同概率划入不同子结点中。













