---
title: 《机器学习》笔记 —— EM算法
date: 2017-09-28 14:00:00
categories:
- Meachine Learning
tags:
- Meachine Learning
comments: true
mathjax: true
---

### EM算法（对含隐变量的模型进行参数估计）

令x表示已观测变量集，z表示隐变量集，$\theta$表示模型参数，若对$\theta$做极大似然估计，则应最大化对数似然。
$$
LL(\theta|x,z)=\ln P(x,z|\theta)
$$

由于存在隐变量z，此式无法直接求解，通过对z计算期望，求最大化已观测数据的对数边际似然：
$$
LL(\theta,x) = \ln P(x|\theta)=\ln \sum_zP(x,z|\theta)
$$

EM算法基本思想：

若$\theta$已知，则可根据训练数据推断最优隐变量z（E步）

若z的值已知，则可根据z对参数$\theta$做极大似然估计（M步）

迭代直至收敛：
（1）基于$\theta^t$推断z的期望记为$z^t$

（2）基于已观测变量x和$z^t$对$\theta$做极大似然估计，记为$\theta^{t+1}$

隐变量估计问题也可以通过梯度下降等优化算法求解，但求和项数随隐变量的数目以指数级上升，会给梯度计算带来麻烦，而EM算法为非梯度算法。
