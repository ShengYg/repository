---
layout: post
title:  "梯度下降算法"
date:   2017-06-03 14:00:00 +0800
categories: [machine learning]
tags: []
description: 
---

> 本文摘自[梯度下降优化算法综述](http://blog.csdn.net/google19890102/article/details/69942970)

## Table of Contents

- [梯度下降法的变形形式](#1)
  - [批梯度下降法](#1.1)
  - [随机梯度下降法](#1.2)
  - [小批量梯度下降法](#1.3)
- [挑战](#2)
- [梯度下降优化算法](#3)
  - [动量法 Momentum](#3.1)
  - [Nesterov加速梯度下降法](#3.2)
  - [Adagrad](#3.3)
  - [Adadelta](#3.4)
  - [Adam](#3.5)
  - [算法可视化](#3.6)
  - [选择使用哪种优化算法](#3.7)

---

<a name='1'></a>
## 1. 梯度下降法的变形形式

<a name='1.1'></a>
### 1.1 批梯度下降法

Vanilla梯度下降法，又称为批梯度下降法（batch gradient descent），在**整个训练数据集**上计算损失函数关于参数θ的梯度:

$$\theta = \theta - \eta\cdot\bigtriangledown_\theta J(\theta)$$

因为在执行每次更新时，我们需要在整个数据集上计算所有的梯度，所以批梯度下降法的速度会很**慢**，同时，批梯度下降法无法处理超出内存容量限制的数据集。批梯度下降法同样也不能在线更新模型，即在运行的过程中，不能增加新的样本。

我们利用梯度的方向和学习率更新参数，学习率决定我们将以多大的步长更新参数。对于凸误差函数，批梯度下降法能够保证收敛到全局最小值，对于非凸函数，则收敛到一个局部最小值。

<a name='1.2'></a>
### 1.2 随机梯度下降法

随机梯度下降法（stochastic gradient descent, SGD）根据每一条训练样本$x^{(i)}$和标签$y^{(i)}$更新参数：

$$\theta = \theta - \eta\cdot\bigtriangledown_\theta J(\theta;x^{(i)};y^{(i)})$$

对于大数据集，因为批梯度下降法在每一个参数更新之前，会对相似的样本计算梯度，所以在计算过程中会有冗余。而SGD在每一次更新中只执行一次，从而消除了冗余。因而，通常SGD的**运行速度更快**，同时，可以用于**在线学习**。SGD以高方差频繁地更新，导致目标函数出现如图1所示的剧烈波动。

与批梯度下降法的收敛会使得损失函数陷入局部最小相比，由于SGD的波动性，一方面，波动性使得SGD可以跳到新的和潜在更好的局部最优。另一方面，这使得最终收敛到特定最小值的过程变得复杂，因为SGD会一直持续波动。然而，已经证明当我们缓慢减小学习率，SGD与批梯度下降法具有相同的收敛行为，对于非凸优化和凸优化，可以分别收敛到局部最小值和全局最小值。

<a name='1.3'></a>
### 1.3 小批量梯度下降法

小批量梯度下降法最终结合了上述两种方法的优点，在每次更新时使用n个小批量训练样本：

$$\theta = \theta - \eta\cdot\bigtriangledown_\theta J(\theta;x^{(i:i+n)};y^{(i:i+n)})$$

<a name='2'></a>
## 2. 挑战

虽然Vanilla小批量梯度下降法并不能保证较好的收敛性，并且，给我们留下了如下的一些挑战：

1. 选择一个合适的学习率可能是困难的

1. 学习率调整算法试图在训练的过程中通过例如退火的方法调整学习率，即根据预定义的策略或者当相邻两代之间的下降值小于某个阈值时减小学习率。然而，策略和阈值需要预先设定好，因此无法适应数据集的特点

1. 高度非凸误差函数普遍出现在神经网络中，在优化这类函数时，另一个关键的挑战是使函数避免陷入无数次优的局部最小值

<a name='3'></a>
## 3. 梯度下降优化算法

<a name='3.1'></a>
### 3.1 动量法 Momentum

动量法将历史步长的更新向量的一个分量$\gamma$增加到当前的更新向量中:

$$\upsilon_t = \gamma\upsilon_{t-1}+\eta\bigtriangledown_\theta J(\theta)$$
$$\theta = \theta - \upsilon_t$$

从本质上说，动量法，就像我们从山上推下一个球，球在滚下来的过程中累积动量，变得越来越快（直到达到终极速度，如果有空气阻力的存在，则$\gamma$<1）。同样的事情也发生在参数的更新过程中：对于在梯度点处具有相同的方向的维度，其动量项增大，对于在梯度点处改变方向的维度，其动量项减小。因此，我们可以得到更快的收敛速度，同时可以减少摇摆。

<a name='3.2'></a>
### 3.2 Nesterov加速梯度下降法

球从山上滚下的时候，盲目地沿着斜率方向，往往并不能令人满意。我们希望有一个智能的球，这个球能够知道它将要去哪，以至于在重新遇到斜率上升时能够知道减速。

Nesterov加速梯度下降法（Nesterov accelerated gradient，NAG）是一种能够给动量项这样的预知能力的方法。我们知道，我们利用动量项$\gamma\upsilon_{t-1}$来更新参数$\theta$。通过计算$\theta-\gamma\upsilon_{t-1}$能够告诉我们参数未来位置的一个近似值（梯度并不是完全更新），这也就是告诉我们参数大致将变为多少。通过计算关于参数未来的近似位置的梯度，而不是关于当前的参数θ的梯度，我们可以高效的求解 ：

$$\upsilon_t = \gamma\upsilon_{t-1}+\eta\bigtriangledown_\theta J(\theta-\gamma\upsilon_{t-1})$$
$$\theta = \theta - \upsilon_t$$

动量法首先计算当前的梯度值（小的蓝色向量），然后在更新的累积梯度（大的蓝色向量）方向上前进一大步，Nesterov加速梯度下降法NAG首先在先前累积梯度（棕色的向量）方向上前进一大步，计算梯度值，然后做一个修正（绿色的向量）。这个具有预见性的更新防止我们前进得太快，同时增强了算法的响应能力，这一点在很多的任务中对于RNN的性能提升有着重要的意义。

<img src="{{ site.baseurl }}/assets/pic/9_01.png" height="100px" width="400px" >

<a name='3.3'></a>
### 3.3 Adagrad

Adagrad是这样的一种基于梯度的优化算法：让学习率适应参数，对于出现次数较少的特征，我们对其采用更大的学习率，对于出现次数较多的特征，我们对其采用较小的学习率。因此，Adagrad非常适合处理**稀疏数据**。

前面，我们每次更新所有的参数$\theta$时，每一个参数$\theta_i$都使用的是相同的学习率$\eta$。Adagrad在t时刻对每一个参数$\theta_i$使用了不同的学习率。我们首先介绍Adagrad对每一个参数的更新，然后我们对其向量化。为了简洁，令$g_{t,i}$为在$t$时刻目标函数关于参数$\theta_i$的梯度：

$$g_{t,i} = \bigtriangledown_\theta J(\theta_i)$$

在t时刻，对每个参数$\theta_i$的更新过程变为：

$$\theta_{t+1,i} = \theta_{t,i}-\eta\cdot g_{t,i}$$

对于上述的更新规则，在t时刻，基于对$\theta_i$计算过的历史梯度，Adagrad修正了对每一个参数$\theta_i$的学习率：

$$\theta_{t+1,i} = \theta_{t,i}-\frac{\eta}{\sqrt{G_{t,ii}+\epsilon}}\cdot g_{t,i}$$

其中，$G_t\in R^{d\times d}$是一个对角矩阵，对角线上的元素$i,i$是直到t时刻为止，所有关于$\theta_i$的**梯度的平方和**，$\epsilon$是平滑项，用于防止除数为0（通常大约设置为1e−8）。比较有意思的是，如果没有平方根的操作，算法的效果会变得很差。

由于$G_t$的对角线上包含了关于所有参数$theta$的历史梯度的平方和，现在，我们可以通过$G_t$和$g_t$之间的元素向量乘法$\odot$向量化上述的操作：

$$\theta_{t+1} = \theta_{t}-\frac{\eta}{\sqrt{G_{t}+\epsilon}}\cdot g_{t}$$

Adagrad算法的一个主要优点是无需手动调整学习率。在大多数的应用场景中，通常采用常数0.01。

Adagrad的一个主要缺点是它在**分母中累加梯度的平方**：由于每增加一个正项，在整个训练过程中，累加的和会持续增长。这会导致学习率变小以至于最终变得无限小，在学习率无限小时，Adagrad算法将无法取得额外的信息。接下来的算法旨在解决这个不足。

<a name='3.4'></a>
### 3.4 Adadelta

Adadelta是Adagrad的一种扩展算法，以处理Adagrad学习速率单调递减的问题。不是计算所有的梯度平方，Adadelta将累计历史梯度的大小限制为一个固定值$\omega$。

在Adadelta中，无需存储先前的$\omega$个平方梯度，而是将梯度的平方递归地表示成所有历史梯度平方的均值。在t时刻的均值$E[g^2]_t$只取决于先前的均值和当前的梯度（分量$\gamma$类似于动量项）:

$$E[g^2]_t=\gamma\cdot E[g^2]_{t-1} + (1-\gamma)g^2_t$$

我们先前得到的Adagrad参数更新向量变为：
$$\theta_{t} = -\frac{\eta}{\sqrt{G_{t}+\epsilon}}\odot g_{t}$$

现在，我们简单将对角矩阵$G_t$替换成历史梯度的均值$E[g^2]_t$：

$$\theta_{t} = -\frac{\eta}{\sqrt{E[g^2]_t+\epsilon}}\odot g_{t}$$

由于分母仅仅是梯度的均方根（root mean squared，RMS）误差，我们可以简写为：

$$\theta_{t} = -\frac{\eta}{RMS[g]_t}\odot g_{t}$$

上述更新公式中的每个部分（与SGD，动量法或者Adagrad）并不一致，即更新规则中必须与参数具有相同的假设单位。为了实现这个要求，定义另一个指数衰减均值，这次不是梯度平方，而是参数的平方的更新：

$$E[\Delta\theta^2]_t=\gamma\cdot E[\Delta\theta^2]_{t-1} + (1-\gamma)\Delta\theta^2_t$$

因此，参数更新的均方根误差为：

$$RMS[\Delta\theta]_t=\sqrt{E[\Delta\theta^2]_t+\epsilon}$$

由于$RMS[\Delta\theta]_{t}$是未知的，我们利用参数的均方根误差来近似更新。利用$RMS[\Delta\theta]_{t-1}$替换先前的更新规则中的学习率$\eta$，最终得到Adadelta的更新规则：

$$\Delta\theta_t = \frac{RMS[\Delta\theta]_{t-1}}{RMS[g]_t}g_t$$

$$\theta_{t+1} = \theta_t + \Delta\theta_t$$

使用Adadelta算法，我们甚至都无需设置默认的学习率，因为更新规则中已经移除了学习率。

<a name='3.5'></a>
### 3.5 Adam

自适应矩估计（Adaptive Moment Estimation，Adam）是另一种自适应学习率的算法，Adam对每一个参数都计算自适应的学习率。除了像Adadelta和RMSprop一样存储一个指数衰减的历史平方梯度的平均$\upsilon_t$，Adam同时还保存一个历史梯度的指数衰减均值$m_t$，类似于动量：

$$m_t = \beta_1m_{t-1}+(1-\beta_1)g_t$$
$$\upsilon_t = \beta_2\upsilon_{t-1}+(1-\beta_2)g^2_t$$

$m_t$和$\upsilon_t$分别是对梯度的一阶矩（均值）和二阶矩（非确定的方差）的估计，正如该算法的名称。当$m_t$和$\upsilon_t$初始化为0向量时，Adam的作者发现它们都偏向于0，尤其是在初始化的步骤和当衰减率很小的时候（例如$\beta_1$和$\beta_2$趋向于1）。

通过计算偏差校正的一阶矩和二阶矩估计来抵消偏差：

$$\hat{m_t} = \frac{m_t}{1-\beta^t_1}$$
$$\hat{\upsilon_t} = \frac{\upsilon_t}{1-\beta^t_2}$$

正如我们在Adadelta和RMSprop中看到的那样，他们利用上述的公式更新参数，由此生成了Adam的更新规则：

$$\theta_{t+1} = \theta_t - \frac{\eta}{\sqrt{\hat{\upsilon_t}+\epsilon}}\hat{m_t}$$

<a name='3.6'></a>
### 3.6 算法可视化

<center>
<img src="{{ site.baseurl }}/assets/pic/9_02.gif" height="240px">
<img src="{{ site.baseurl }}/assets/pic/9_03.gif" height="240px">
</center>

Adagrad，Adadelta和RMSprop能够立即转移到正确的移动方向上并以类似的速度收敛，而动量法和NAG会导致偏离。然而，NAG能够在偏离之后快速修正其路线，因为NAG通过对最优点的预见增强其响应能力。

SGD，动量法和NAG在鞍点处很难打破对称性，尽管后面两个算法最终设法逃离了鞍点。而Adagrad，RMSprop和Adadelta能够快速想着梯度为负的方向移动，其中Adadelta走在最前面。

<a name='3.7'></a>
### 3.7 选择使用哪种优化算法？

总的来说，RMSprop是Adagrad的扩展形式，用于处理在Adagrad中急速递减的学习率。RMSprop与Adadelta相同，所不同的是Adadelta在更新规则中使用参数的均方根进行更新。最后，Adam是将偏差校正和动量加入到RMSprop中。在这样的情况下，RMSprop、Adadelta和Adam是很相似的算法并且在相似的环境中性能都不错。综合看来，Adam可能是最佳的选择。









