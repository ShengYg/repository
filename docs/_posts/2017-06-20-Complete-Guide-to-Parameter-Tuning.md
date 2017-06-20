---
layout: post
title:  "Complete Guide to Parameter Tuning in GBM and Xgboost"
date:   2017-06-19 10:00:00 +0800
categories: jekyll update
---

## Table of Contents
1. How Boosting Works
1. Understanding GBM Parameters
1. Tuning Parameters

-------

## 1. How Boosting Works
Boosting is a sequential technique which works on the principle of **ensemble**. It combines a set of **weak learners** and delivers improved prediction accuracy. At any instant t, the model outcomes are weighed based on the outcomes of previous instant t-1. The outcomes predicted correctly are given a lower weight and the ones miss-classified are weighted higher.

## 2. GBM Parameters
The overall parameters can be divided into 3 categories:

1. **Tree-Specific Parameters**: These affect each individual tree in the model.
1. **Boosting Parameters**: These affect the boosting operation in the model.
1. **Miscellaneous Parameters**: Other parameters for overall functioning.

### 2.1 Tree-Specific Parameters
![](images/0_00.png)
