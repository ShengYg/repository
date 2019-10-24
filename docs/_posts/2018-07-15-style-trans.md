---
layout: post
title:  "style tansfer"
date:   2018-07-15 14:00:00 +0800
categories: [projects]
tags: []
description: 个人项目总结，风格迁移
---

## WCT
### encoder-decoder
**方法**：训练decoder网络，使其恢复encoder的信息
**作用**：提升速度。
$$L=||I_o - I_i||^2_2+\lambda||\Phi(I_o)-\Phi(I_i)||^2_2$$
### WCT（Whitening and coloring transforms）
**方法**：Encoder和Decoder网络中间加WCT，对content特征和style特征进行融合。
**作用**：考虑了特征间的相关性（实对称矩阵对角化，得到对角阵和正交阵），使得融合更均匀。
#### Whitening transform
$$
\begin{cases}
f_c = f_c-\mu_c,  & \text{减去均值} \\
\hat{f}_c = E_cD^{-1/2}_cE^T_cf_c, & f_cf^T_c=E_cD_cE^T_c \\
\end{cases}
$$

最终满足，
$$\hat{f}_c\hat{f}^T_c=I$$
#### Coloring transform
$$
\begin{cases}
f_s = f_s-\mu_s,  & \text{减去均值} \\
\hat{f}_{cs} = E_sD^{1/2}_sE^T_s\hat{f}_c, & f_sf^T_s=E_sD_sE^T_s \\
\hat{f}_{cs} = \hat{f}_{cs}+m_s
\end{cases}
$$

最终满足，
$$\hat{f}_{cs}\hat{f}^T_{cs}=f_sf^T_s$$

## PhotoWCT