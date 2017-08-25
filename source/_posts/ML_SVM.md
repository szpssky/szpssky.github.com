---
title: 《机器学习》笔记 —— 支持向量机
date: 2017-08-25 08:00:00
categories:
- Meachine Learning
tags:
- Meachine Learning
comments: true
mathjax: true
---
### 支持向量机基本形式

分类学习的思想基本思想是基于训练集D在样本空间找到一个划分超平面，将不同的样本分开，而SVM试图找到一个最大间隔的划分超平面。

划分超平面可通过如下线性方程来描述：
$$
w^Tx+b=0
$$
其中，$w$为法向量，决定超平面方向，$b$为位移量决定超平面与原点的距离。

样本空间任意点$x$到超平面的距离：
$$r=\frac{w^Tx+b}{||w||}$$
假设超平面$(w,b)$能将训练样本正确分类，即对于$(x_i,y_i)\in D)，若$y_i=+1$，则$w^T_i+b>0$，若$y_i=-1$则有$w^Tx_i+b<0$，令
$$
\left \{
\begin{array}{rl}
w^Tx_i+b\geq +1 \text{, }y_i=+1 \\ \\
w^Tx_i+b\leq -1 \text{, }y_i=-1
\end{array}
\right .
$$
距离超平面最近的几个训练样本使上式等号成立，它们被称为“支持向量”，两个异类支持向量到超平面的距离之和：
$$
\gamma=\frac{2}{||w||}
$$
最大间隔，即找到$w,b$，使得$\gama$最大：
$$
\max_{w,b}\frac{2}{||w||} \\
s.t.\quad \, y_i(w^Tx_i+b)\geq 1 \text{, }i=1,2,\dots,m 
$$
即，
$$
\min_{w,b}\frac{1}{2}||w||^2 \\
s.t.\quad y_i(w^Tx_i+b)\geq 1 \text{, }i=1,2,\dots,m
$$
此为支持向量机(SVM)的基本型。
<!-- more -->

### 对偶问题

求解SVM得到大间隔划分超平面所对应的模型：
$$f(x)=w^Tx+b$$
对SVM基本型使用拉格朗日乘子法，则该问题的拉格朗日函数可写为：
$$
L(w,b,\alpha)=\frac{1}{2}||w||^2+\sum_{i=1}^m\alpha_i(1-y_i(w^Tx_i+b)) \qquad (1)
$$
其中$\alpha_i \geq 0$

因为：
$$
\max_\alpha_i L(w,b,\alpha)=\frac{1}{2}||w||^2
$$
所以SVM原问题可以表示为：
$$
\min_{w,b}\max_\alpha_i L(w,b,\alpha)
$$

令$L(w,b,\alpha)$对$w\text{和}b$的偏导为零：
$$
\begin{array}{}
w =\sum_{i=1}^m\alpha_iy_ix_i \qquad &(2) \\
0 =\sum_{i=1}^m\alpha_iy_i    \qquad &(3)
\end{array}
$$
将(2)式代入(1)式，再考虑(3)式的约束，则得SVM得对偶问题：
$$
\max_\alpha \sum_{i=1}^m\alpha_i-\frac{1}{2}\sum_{i=1}^m\sum_{j=1}^m\alpha_i\alpha_jy_iy_jx_i^Tx_j \qquad (4)\\
s.t.\quad \sum_{i=1}^m\alpha_iy_i=0 \qquad \alpha_i\geq 0 \text{, }i=1,2,\dots,m
$$
解得$\alpha$后得：
$$
f(x)=w^Tx+b=\sum_{i=1}^m\alpha_iy_ix_i^Tx+b \qquad (5)
$$
因SVM存在不等式约束，因此上述过程满足KKT条件，即
$$
\left \{
\begin{array}{}
\alpha_i \geq 0 \\ \\
y_if(x_i)-1 \geq 0 \\ \\
\alpha_i(y_if(x_i)-1)=0
\end{array}
\right.
$$
对任意$(x_i,y_i)$,总有$\alpha_i=0$或$y_if(x_i)=1$，若$\alpha_i=0$，则该样本不会在(5)式中出现，也不会影响$f(x)$，若$\alpha_i >0$，则$y_if(x_i)=1$，则该样本位于最大间隔边界上，是一个支持向量。（训练完成后，大部分训练样本都不需要保留。）

***
SVM为什么使用对偶问题求解？

1）原问题求解在样本空间维度很大时求解困难。

2）在解决非线性问题时，需要通过核函数映射到高纬空间，这使得求解更加困难，而使用对偶问题后，复杂度只与样本数量有关。

***

#### SMO算法

SMO算法的基本思想是先固定$\alpha_i$之外的所有参数，然后求$\alpha_i$上的极值，由于存在约束$\sum_{i=1}^m\alpha_iy_i=0$，则固定$\alpha_i$之外的其他变量，$\alpha_i$可由其他变量导出，于是SMO每次选择两个变量$\alpha_i\text{和}\alpha_j$，并固定其他参数，SMO不断迭代如下两个步骤直至收敛：

1）选取一对需要更新的变量$\alpha_i\text{和}\alpha_j$

2) 固定$\alpha_i\text{和}\alpha_j$以外的参数，求解(5)式获得更新后的$\alpha_i\text{和}\alpha_j$

只要选取的$\alpha_i\text{和}\alpha_j$中有一个不满足KTT条件，目标函数就会在迭代后减小，KTT条件违背的程度越大，则变量更新后可能导致的目标函数值减幅越大。SMO采取启发式选取：使选取的两个变量所对应的样本间隔最大。

SMO优化两个参数的过程非常高效，因为仅考虑$\alpha_i\text{和}\alpha_j$的情况是，上述(4)式中的约束条件可改写成：
$$
\alpha_iy_i+\alpha_jy_i=c \text{, } \qquad \alpha_i \geq 0,\alpha_j \geq 0
$$
其中：
$$
c = -\sum_{k\neq i,j}\alpha_ky_k
$$
则根据$\alpha_iy_i+\alpha_jy_i=c$可消去$\alpha_j$，得到关于$\alpha_i$的单变量二次规划问题，此问题拥有闭式解，可以高效更新$\alpha_i\text{和}\alpha_j$

又对任意一个支持向量$(x_s,y_s)$，根据KTT条件可知$y_sf(x_s)=1$，所以：
$$
y_s(\sum_{i\in S}\alpha_iy_ix_i^Tx_s+b)=1
$$
其中$S$为所有支持向量的下标集合，通常根据所有支持向量求解的平均值解得b的值。

### 核函数

原始样本空间内也许不存在一个能正确划分两类样本的超平面，对于这样的问题，可将样本从原始空间映射到一个更高维的特征空间，如果原始空间是有限维，那一定存在一个高维特征空间使样本可分。

令$\phi(x)$表示将$x$映射后的特征向量，则划分超平面对应的模型为：
$$
f(x)=w^T\phi(x)+b
$$
则：
$$
\min_{a,b}\frac{1}{2}||w||^2 \\
s.t. \quad y_i(w^T\phi(x_i))+b \geq 1 \text{, }i=1,2,\dots,m
$$
其对偶问题：
$$
\min_\alpha \sum_{i=1}^m \alpha_i-\frac{1}{2}\sum_{i=1}^m\sum_{j=1}^m \alpha_i\alpha_jy_iy_j\phi(x_i)^T\phi(x_j) \qquad (6) \\
s.t. \quad \sum_{i=1}^m \alpha_iy_i=0 \text{, } \quad \alpha_i \geq 0 \text{, }i=1,2,\dots,m
$$

由于特征空间维数可能很高，直接计算$\phi(x_i)^T\phi(x_j)$通常很困难，为了避开这个障碍，则可令：
$$
\kappa(x_i,x_j)=\langle\phi(x_i),\phi(x_j)\rangle=\phi(x_i)^T\phi(x_j)
$$
即$x_i$与$x_j$在特征空间的内积等于它们在原始空间通过$\kappa(\cdot,\cdot)$计算的结果。

因此(6)式可改为：
$$
\max_\alpha \sum{i=1}^m\alpha_i-\frac{1}{2}\sum_{i=1}^m\sum_{j=1}^m\alpha_i\alpha_jy_iy_j\kappa(x_i,x_j) \\
s.t. \quad \sum_{i=1}^m\alpha_iy_i=0 \text{, } \quad \alpha_i \geq 0 \text{, }i=1,2,\dots,m
$$
求解得：
$$
\begin{array}{}
f(x) & =w^T\phi(x)+b \\
& = \sum_{i=1}^m\alpha_iy_i\phi(x_i)^T\phi(x)+b \\
& = \sum_{i=1}^m\alpha_iy_i\kappa(x_i,x_j)+b
\end{array} \qquad (7)
$$
$\kappa(\cdot,\dot)$称为核函数，式(7)又称“支持向量展式”

常用核函数：
| 核函数      | 形式  |  说明  |
| :--------: |:-----:  | :----:  |
| 线性核     | $\kappa(x_i,x_j)=x_i^Tx^j$ |        |
| 多项式核   | $\kappa(x_i,x_j)=(x_i^Tx_j)^d$     |   $d \geq 1$为多项式的次数   |
| 高斯核     | $\kappa(x_i,x_j)=exp(-\frac{||x_i-x_j||^2}{2\sigma^2})$      |  $\sigma > 0$为高斯核带宽  |
|拉布拉斯核  | $\kappa(x_i,x_j)=exp(-\frac{||x_i-x_j||}{\sigma^2})$ |$\sigma > 0$|
|sigmoid核  | $\kappa(x_i,x_j)=tanh(\beta x_i^Tx_j + \theta)$ | $\beta>0,\theta<0$|

此外还可进行函数组合：
$$
\gamma_1\kappa_1+\gamma_2\kappa_2 \text{、} 
\kappa_1 \otimes \kappa_2(x,z)=\kappa_1(x_1,z)\kappa_2(x_2,z) \text{、}
\kappa(x,z)=g(x)\kappa_1(x,z)g(z)
$$

### 软间隔与正则化

软间隔允许支持向量机在一些样本出错，即软间隔允许某些样本不满足约束：$y_i(w^Tx_i+b) \geq 1$

优化目标：
$$
\min_{a,b}\frac{1}{2}||w||^2+C\sum_{i=1}^m\ell_{0/1}(y_i(w^Tx_i+b)-1)
$$
其中C>0，$\ell_{0/1}$是0/1损失函数：
$$
\ell_{0/1}(z)= \left \{
\begin{array}{}
1 \text{, if }z < 0 \\ \\
0 \text{, otherwise}
\end{array}
\right .
$$
替代损失：

hige损失：$\ell_{hinge}(z)=\max(0,1-z)$

指数损失：$\ell_{exp}(z)=exp(-z)$

对率损失：$\ell_{\log}(z)=\log(1+exp(-z))$

引入“松弛变量”$\xi \geq 0$，则
$$
\min_{w,b,\xi_i} \frac{1}{2}||w||^2+C\sum_{i=1}^m \xi_i \\
s.t. \quad y_i(w^Tx_i+b) \geq 1-\xi_i \text{, } \quad \xi \geq 0 \text{, }i=1,2,\dots,m
$$
亦称“软间隔支持向量机”

如果使用对率损失函数$\ell_{\log}$替换0/1损失函数，则几乎得到逻辑回归模型，实际上SVM与逻辑回归的优化目标相近，性能通常也相当，但逻辑回归能直接用于多分类任务。

由于hinge损失有一块“平坦”的零区域，使得SVM的解具有稀疏性，而对率损失是光滑的单调递减函数，不能导出支持向量的概念，故逻辑回归的解依赖更多训练样本。

SVM更一般的形式：
$$
\min_f\Omega(f)+C\sum_{i=1}^m\ell(f(x_i),y_i)
$$

$\Omega$为结构风险(描述$f$的性质)，$\ell$为经验风险(描述模型与训练数据契合程度)，C用来进行折中。

从经验风险最小化的角度，$\Omega(f)$表示获得具有何种性质的模型，另一方面，改信息有助于消减假设空间，降低过拟合风险，上式称为正则化问题，$\Omega$称为正则项，C称为正则化系，$L_p$范数为常用正则化项。

$L_2$范数：w的分量取尽量均衡，即非零向量尽量稠密。
$L_1$范数：w的分量尽量稀疏，即非零分量个数尽量少。

### 支持向量回归(SVR)

假设能容忍$f(x)$与$y$之间最多有$\epsilon$的偏差，即仅当$|f(x)-y|>\epsilon$时才计算损失。

SVR问题形式为：
$$
\min_{w,b}\frac{1}{2}||w||^2+C\sum_{i=1}^m\ell_\epsilon(f(x_i)-y_i)
$$
其中C为正则化常数，$\ell_\epsilon$为$\epsilon$不敏感损失函数。
$$
\ell_\epsilon(z)= \left \{
\begin{array}{}
0 \qquad & \text{if } |z| \leq \epsilon \\ \\
|z|-\epsilon &  \text{otherwise}
\end{array}
\right .
$$
引入松弛变量$\xi_i,\hat \xi_i$:
$$
\min_{w,b,\xi_i,\hat \xi_i}\frac{1}{2}||w||^2+C\sum_{x=1}^m(\xi_i+\hat \xi_i) \\
\begin{array}{}
s.t. \quad & f(x_i)-y_i \leq \epsilon + \xi_i \\
& y_i-f(x_i) \leq \epsilon + \hat \xi_i \\
& \xi_i \geq 0,\quad \hat \xi_i \geq 0 , \quad i=1,2,\dots,m
\end{array}
$$

其解法与SVM类似。














