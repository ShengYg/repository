---
layout: post
title:  "Reservoir Sampling"
date:   2017-03-06 21:00:00 +0800
categories: [algorithms]
tags: [sample]
description: 水塘抽样算法 -- Reservoir Sampling。在序列流中取k个数，确保取出某个数据的概率为:k/n
---

### 在序列流中取k个数，如何确保随机性，即取出某个数据的概率为:k/(已读取数据个数)

建立一个数组，将序列流里的前k个数，保存在数组中。对于第n个数`A[n]`，以`k/n`的概率取`A[n]`并以`1/k`的概率随机替换“蓄水池”中的某个元素；否则“蓄水池”数组不变。依次类推，可以保证取到数据的随机性。

数学归纳法证明如下：

- 当n=k时，显然“蓄水池”中任何一个数都满足，保留这个数的概率为`k/k`。

- 假设当n=m(m>k)时，“蓄水池”中任何一个数都满足，保留这个数的概率为`k/m`。 

- 当n=m+1时，以`k/(m+1)`的概率取`A[n]`，并以`1/k`的概率，随机替换“蓄水池”中的某个元素。第m+1个元素留下的概率为`k/(m+1)`，前m个元素留下的概率均为：`k/m * (1 - k/(m+1) * 1/k) = k/(m+1)`。

~~~python
from random import randint

def sample(iterable, n):
    reservoir = []
    for t, item in enumerate(iterable):
        if t < n:
            reservoir.append(item)
        else:
            m = random.randint(0,t)
            if m < n:
                reservoir[m] = item
    return reservoir
~~~

~~~java
public int[] reservoirSampling(int[] data,int k){
    if(data == null){
        return new int[0];
    }
    if(data.length < k){
        return new int[0];
    }
    int[] sample = new int[k];
    int n = data.length;
    for(int i = 0; i < n; i++){
        if(i < k){
            sample[i]=data[i];
        }else{
            int j = new Random().nextInt(i);
            if(j < k){
                sample[j] = data[i];
            }
        }
    }
    return sample;
}
~~~
