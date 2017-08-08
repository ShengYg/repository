---
layout: post
title:  "Batchnorm"
date:   2017-05-30 14:00:00 +0800
categories: [machine learning]
tags: []
description: 
---


Google在ICML文中描述的非常清晰，即在每次SGD时，通过mini-batch来对相应的activation做规范化操作，使得结果（输出信号各个维度）的均值为0，方差为1。而最后的“scale and shift”操作则是为了让因训练所需而“刻意”加入的BN能够有可能还原最初的输入，从而保证整个network的capacity。


$$\mu_\beta = \frac{1}{m}\sum_{i=1}^{m}x_i$$
$$\sigma_\beta^2 = \frac{1}{m}\sum_{i=1}^{m}(x_i-\mu_\beta)^2$$
$$\hat{x_i} = \frac{x_i-\mu_\beta}{\sqrt{\sigma_\beta^2+\epsilon}}$$
$$y_i = \gamma\hat{x_i}+\beta$$


Batch Normalization作用:

1. 允许网络使用较高的learning rate

1. 移除或使用较低的dropout

1. 降低L2权重衰减系数

1. 取消Local Response Normalization层

1. 减少图像扭曲的使用






