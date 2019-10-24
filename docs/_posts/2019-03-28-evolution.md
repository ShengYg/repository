---
layout: post
title:  "进化算法"
date:   2019-03-28 10:00:00 +0800
categories: [Machine learning]
tags: []
description: 遗传算法，进化策略，神经网络进化
---

- 目录
{:toc #markdown-toc}

通用算法：
- init：集合$A(s=0)$
- repeat：
    - evaluate：集合$A(s)$中每个元素$a_i$的适应度$\phi_i$
    - select：依据适应度$\phi_i$按概率挑选$B(s)$
    - reproduce：从$B(s)$继承得到$C(s)$
    - $A(s+1)=C(s)$
- 满足条件时break


### 遗传算法 Genetic Algorithms
~~~python
pop = np.random.randint(2, size=(POP_SIZE, DNA_SIZE))   # 种群DNA

fitness = get_fitness(pop)      # 得到种群适应度
pop = select(pop, fitness)      # 按适应度根据概率选 pop
pop_copy = pop.copy()
for parent in pop:
    child = crossover(parent, pop_copy)     # 从 pop_copy 中随机选一个与 parent 进行基因交叉配对
    child = mutate(child)                   # 变异，随机0-1变换
    parent[:] = child
~~~

**Microbial GA**：挑选两个pop，对比fitness，胜者不变，败者进行crossover和mutate。

<center>
<img src="{{ site.baseurl }}/assets/pic/MGA.png" height="300px" >
</center>

### 进化策略 Evolution Strategy
GA与ES的差别：
- GA用二进制DNA; ES用实数DNA（均值+标准差）
- GA通过0-1变换进行变异；ES通过标准差变异
- GA选优秀父母进行繁殖; ES繁殖后选优秀孩子

~~~python
for _ in range(N_GENERATIONS):
    kids = crossover(pop)
    kids = mutate(pop)
    pop = kill_bad(pop, kids)
~~~

**Natural ES**：将kid的梯度作为fitness的权重进行优化，指导向适合的方向进化

<center>
<img src="{{ site.baseurl }}/assets/pic/NES.png" height="300px" >
</center>

### 神经网络进化 Neuro-Evolution
利用GA、ES算法改变神经网络的结构

#### NEAT算法
[Evolving Neural Networks through Augmenting Topologies](http://nn.cs.utexas.edu/downloads/papers/stanley.ec02.pdf)

1. 对网络编码，表示成Node Genes+Connect Genes
    <center>
    <img src="{{ site.baseurl }}/assets/pic/genotype.png" height="300px" >
    </center>
2. crossover：根据ID对齐，遗传一方的信息
    <center>
    <img src="{{ site.baseurl }}/assets/pic/crossover.png" height="500px" >
    </center>
3. mutate，对connect或node
    <center>
    <img src="{{ site.baseurl }}/assets/pic/mutate.png" height="300px" >
    </center>

#### ES与强化学习
参考[Parameter Noise](https://openai.com/blog/better-exploration-with-parameter-noise/)与[Evolution Strategies](https://openai.com/blog/evolution-strategies/)，ES对网络参数扰动，能提升RL网络结果。