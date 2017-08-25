---
title: 《机器学习》笔记 —— 线性模型
date: 2017-08-22 10:00:00
categories:
- Meachine Learning
tags:
- Meachine Learning
comments: true
mathjax: true
---

### 基本形式：

给定d个属性的示例$x=(x_1,x_2,\dots ,x_d)$，线性模型试图学得一个通过属性的线性组合来进行预测的函数：
$$f(x)=w_1x_1+w_2x_2+,\dots ,w_dx_d+b$$
一般向量形式：
$$f(x)=W^Tx+b$$


#### 线性回归

给定数据集$D=\{(x_1,y_1),(x_2,y_2),\dots ,(x_m,y_m)\}$，其中$x_i=(x_{i1},\dots ,x_{id})$，$y_i\in \mathbb{R}$，线性回归试图学得一个线性模型以尽可能准确预测实值输出标记。

线性回归试图学得$f(x)=w^Tx+b\approx y_i$，以均方误差作为性能度量，目标是使均方误差最小，即：
$$(w^*,b^*)=\arg\min_{w,b}\sum_{i=1}^m(f(x_i)-y_i)^2$$

基于均方误差最小化来进行建模，求解的方法为“最小二乘法”，其试图找到一条直线，使所有样本到直线的距离最小。
<!-- more -->

以$\hat w=(w;b)$，相应的数据集D表示为$m\text{x}(d+1)$的矩阵$X$，则
$$\hat w=\arg\min_{\hat w}(y-X\hat w)^T(y-X\hat w)$$
令$E_{\hat w}=(y-X\hat w)^T(y-X\hat w)$，又$\hat w$求导得：
$$\frac{\partial E_\hat w}{\partial \hat w}=2X^T(X\hat w-y)$$
当$X^TX$为满秩矩阵或正定矩阵，令上式为0可得：
$$\hat w^*=(X^TX)^{-1}X^Ty$$
则令$\hat x_i=(x_i,1)$

$f(\hat x_i)=\hat x_i^T(X^TX)^{-1}X^Ty$
若$X^TX$不为满秩，则$\hat w^*$将有多组解，通常由学习算法的归纳偏好决定最终解，常见作法是引入正则化。

假设我们认为示例所对应的输出是在指数尺度上变化，则衍生出对数线性回归。

对数线性回归：$\ln y=w^Tx+b$

实际上试图让$e^{w^Tx+b}$逼近y。

更一般的，单调可微函数$g(\cdot)$，令$y=g^{-1}(w^Tx+b)$，称为广义线性模型，$g(\cdot)$称为“联系函数”。

#### 逻辑回归

针对分类任务，只需找一个单调可微函数将分类任务的真实标记与线性回归的预测值联系起来。

单位阶跃函数：
$$y = \left\{
\begin{array}{rl}
0 & \text{if } z < 0 \\
0.5 &\text{if } z = 0  \qquad\text{(Heaviside函数)}\\
1 & \text{if } z > 0
\end{array} \right. $$
由于单位阶跃函数不连续，不能直接作为$g^{-1}(\cdot)$

logistic函数：
$$y=\frac{1}{1+e^{-z}} \qquad\text{(sigmoid)}$$
即：
$$y=\frac{1}{1+e^{-(w^Tx+b)}}$$
两边取对数，得到对数几率：
$$\ln\frac{y}{1-y}=w^Tx+b$$

那么如何确定$w\text{和}b$呢？将$y$视为类后验概率$P(y=1|x)$则，
$$
\ln\frac{p(y=1|x)}{p(y=0)|x}=w^T+b \\
\Rightarrow \qquad p(y=1|x)=\frac{e^{w^Tx+b}}{1+e^{w^Tx+b}} \quad
p(y=0|x)=\frac{1}{e^{w^Tx+b}}
$$
通过最大似然估计$w\text{和}b$给定数据集$\{(x_i,y_i)\}_{i=1}^m$，最大化对数似然：
$$\ell(w,b)=\sum_{i=1}^m\ln p(y_i|x_i;w,b)$$
令$\beta=(w,b),\hat x=(x,1)$则$w^T+b$为$\beta^T\hat x$，再令$p_1(\hat x;\beta)=p(y=1|\hat x;\beta)$,$p_0(\hat x;\beta)=p(y=0|\hat x;\beta)$则：
$$p(y_i|x_i;w,b)=y_ip_i(\hat x;\beta)+(1-y_i)p_0(\hat x_i;\beta)$$
最大化转化为最小化：
$$\ell(\beta)=\sum_{i=1}^m(-y_i\beta^T\hat x_i)+\ln(1+e^{\beta^T\hat x_i})$$
由经典数值优化算法，梯度下降、牛顿法等可解其最优解。

#### 线性判别分析(LDA)

LDA思想：设法将样例投影到一条直线上，使同类样例的投影点尽可能的近，异类样例的投影点尽可能远。

给定数据集$D=\{(x_i,y_i)\}_{i=1}^m,y_i\in\{0,1\}$,令$X_i,\mu_i,\Sigma_i$，分别表示第$i\in\{0,1\}$类示例集合、均值向量、协方差矩阵。

若将数据投影在直线$w$上，则两类样本的中心在直线上的投影分别为$w^T\mu_0\text{和}w^T\mu_1$，两类样本的协方差为$w^T\Sigma_0w\text{和}w^T\Sigma_1w$，直线是一维空间，故$w^T\mu_0、w^T\mu_1、w^T\Sigma_0w\text{和}w^T\Sigma_1w$均为实数。

同类样例投影点近，即同类样例投影点的协方差尽可能小，异类样例投影点尽可能远即类中心距离尽可能大，综合考虑最大化：
$$
\begin{array}{rl}
J & = \frac{||w^T\mu_0-w^T\mu_1||^2}{w^T\Sigma_0w+w^T\Sigma_1w} \\
  & = \frac{w^T(\mu_0-\mu_1)(\mu_0-\mu_1)^Tw}{w^T(\Sigma_0+\Sigma_1)w}
\end{array}
$$

定义类内散度矩阵：
$$
\begin{array}{rl}
S_w & = \Sigma_0+\Sigma_1 \\
    & = \sum_{x\in X_0}(x-\mu_0)(x-\mu_0)^T+\sum_{x\in X_1}(x-\mu_1)(x-\mu_1)^T
\end{array}
$$
类间散度矩阵
$$
S_b = (\mu_0-\mu_1)(\mu_0-\mu_1)^T
$$
则，
$$
J = \frac{w^TS_bw}{w^TS_ww} \qquad \text{(广义瑞利商)}
$$
令$w^TS_ww=1$，则得：
$$
\min_w-w^TS_bw \quad \text{s.t.}w^TS_ww=1
$$
由拉格朗日乘子法，上式等价于
$$S_bw=\lambda S_ww$$

又因$S_bw$的方向恒为$\mu_0-\mu_1$，不妨令$S_bw=\lambda(\mu_0-\mu_1)$，则代入上式得
$$
w=S_w^{-1}(\mu_0-\mu_1)
$$
因数值解的稳定性，对$S_w$进行奇异值分解，即$S_w=V\Sigma^{-1}U^T$，得到$S_w^{-1}=V\Sigma^{-1}U^T$

##### 多类LDA
假定存在N个类，且第i类示例为$m_i$，定义“全局散度矩阵”：
$$
\begin{array}{rl}
S_t & = S_b+S_w \\
    & = \sum_{i=1}^m(x_i-\mu)(x_i-\mu)^T
\end{array}
$$

类内散度矩阵$S_w$重定义为每个类别的散度矩阵之和：
$$
S_w=\sum_{i=1}^NS_{wi}
$$
$$
\begin{array}
\text{其中} \qquad \qquad S_{wi} & =\sum_{x\in X_i}(x-\mu_i)(x-\mu_i)^T \\
\text{所以} \qquad \qquad S_b & = S_t-S_b \\
& = \sum_{i=1}^Nm_i(\mu_i-\mu)(\mu_i-\mu)^T
\end{array}
$$

常见优化目标：
$$
\max_W\frac{tr(W^TS_bW)}{W^TS_wW}
$$
其中$W\in \mathbb{R}^{dx(N-1)}$，$tr(\cdot)$为矩阵的迹。通过广义特征值问题求解：
$$
S_bW = \lambda S_wW
$$
$W$的闭式解则是$S_w^{-1}S_b$的最大广义特征值所对应的特征向量组成的矩阵。

若将$W$视为投影矩阵，则多类LDA将投影到$N-1$维空间，$N-1$通常远小于原有属性数，且投影过程使用了类别信息，所以LDA也常被视为经典的监督降维技术。

#### 多分类学习

利用二分类来解决多分类问题

经典拆分策略：

1）一对一：将N个类别两两配对，从而产生N(N-1)/2个二分类任务，测试阶段将新样本交给所有分类器，最终结果通过投票产生。

2）一对其余：训练N个分类器，在测试阶段若仅有一个分类器预测为正类，则对应类别标记为最终分类结果，否则要考虑各分类器的预测置信度。

3）多对多：每次讲若干个类作为正类，若干个类作为反类，正类、反类构造需特别设计，常用的技术：“纠错输出码”(ECOC)。






