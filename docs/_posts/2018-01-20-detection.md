---
layout: post
title:  "detection"
date:   2018-01-20 14:00:00 +0800
categories: [projects]
tags: []
description: detection, deep learning
---


## YOLO
#### 网络输出
<center>
<img src="{{ site.baseurl }}/assets/pic/yolo_net.jpg" height="300px" >
</center>

$$S*S*(B*5)+C$$，整幅图片分成$S*S$个网格，B表示bounding-box信息，包括$(x_{center},y_{center})$，w，h，confidence，$x_{center}$与$y_{center}$相对网格归一化，w，h相对图片归一化，confidence表示置信度（是否是物体+IOU），C表示类别数。

- 相当于1个anchor的faster-rcnn，并且前景-背景与类别同时计算。
- 测试时每个格子只能得到一个结果，不适合大量小物体。
- 不适合奇怪形状的物体。
- 最后两层fc $$(1024*7*7\to 4096*1*1\to 30*7*7)$$ 丢失了位置信息，YOLO2进行了改进

#### 损失函数

<center>
<img src="{{ site.baseurl }}/assets/pic/yolo_loss.jpg" height="400px" >
</center>

<center>
<img src="{{ site.baseurl }}/assets/pic/yolo_loss2.jpg" height="400px" >
</center>

loss的设计对于小物体h与w的回归显然不合理，尽管已经设计成$\sqrt{x}$形式。

## YOLO2
#### 网络输出
<center>
<img src="{{ site.baseurl }}/assets/pic/yolo2_loss.jpg" height="400px" >
</center>

- 预测类别与空间位置解耦（与faster-rcnn一致）
- 每个格子预测5个结果（对ground-truth聚类得到的结果）

## SSD
#### 网络输出

<center>
<img src="{{ site.baseurl }}/assets/pic/ssd_net.jpg" height="300px" >
</center>

- 多尺度划分格子，生成$S*S*K*(4+C+1)$
- 训练策略，loss函数与faster-rcnn一致

## R-FCN
#### 解决问题
<center>
<img src="{{ site.baseurl }}/assets/pic/rfcn.jpg" height="300px" >
</center>
- conv-roi-fc：fc层丢失了位置信息
- conv-roi-conv-fc：roi之后卷积，计算量增大
- conv-position_roi:

位置敏感得分图（position-sensitive score map），每个score map都是只属于“一个类别的一个部位”。
