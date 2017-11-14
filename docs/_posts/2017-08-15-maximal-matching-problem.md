---
layout: post
title:  "二分图"
date:   2017-08-15 14:00:00 +0800
categories: [algorithms]
tags: [graph]
description: 二分图最大匹配
---

## 二分图最大匹配求解

#### 1、问题简介
给定一个二分图G，在G的一个子图M中，M的边集{E}中的任意两条边都不依附于同一个顶点，则称M是一个匹配。选择这样的子集中边数最大的子集称为图的最大匹配问题(maximal matching problem)。

#### 2、概念

1. **匹配**(matching)是一个边集，满足边集中的边两两不邻接。匹配又称边独立集(edge independent set)。
1. 在匹配中的点称为**匹配点**(matched vertex)或饱和点；反之，称为未匹配点(unmatched vertex)或未饱和点。
1. **交错轨**(alternating path)是图的一条简单路径，满足任意相邻的两条边，一条在匹配内，一条不在匹配内。
1. **增广轨**(augmenting path)是一个始点与终点都为未匹配点的交错轨。
1. **最大匹配**(maximum matching)是具有最多边的匹配。
1. **增广轨定理**：一个匹配是最大匹配当且仅当没有增广轨。

#### 3、匈牙利算法

根据**一个匹配是最大匹配当且仅当没有增广路**，求最大匹配就是找增广轨，直到找不到增广轨，就找到了最大匹配。遍历每个点，查找增广路，若找到增广路，则修改匹配集和匹配数，否则，终止算法，返回最大匹配数。

匈牙利算法步骤（令G = （X,*,Y）是一个二分图，其中，X = {x1,x2,...xm}, Y = {y1,y2,...yn}。令M为G中的任一个匹配)
 
1. 置M为空
1. 从G中找出一个未匹配点v，如果没有则算法结束，否则，以v为起点，查找增广路（邻接点是为未匹配点，则返回寻找完成，若v的邻接点u是匹配点，则从u开始查找，直至查找到有未匹配点终止）即满足增广路的性质，如果没有找到增广路，则算法终止。
1. 找出一条增广路径P，通过异或操作获得更大的匹配M’代替M(方便要输出增广矩阵以及进一步查找)，匹配数加1。
1. 重复(2)(3)操作直到找不出增广路径为止。

~~~java
public class Main {
    static boolean[] vis;
    static int[] link;
    static boolean [][] map;
    static int n1;
    static int n2;

    public static boolean find(int x){
        for(int i = 1; i <= n2; i++) {
            if(map[x][i] && !vis[i]) {
                vis[i] = true;
                if(link[i] == 0 || find(link[i])){
                    link[i] = x;
                    return true;
                }
            }
        }
        return false;
    }

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);

        n1 = 4;
        n2 = 4;
        map = new boolean[n1+1][n2+1];
        map[1][1] = true;
        map[1][3] = true;
        map[2][1] = true;
        map[3][1] = true;
        map[3][2] = true;
        map[4][3] = true;
        map[4][4] = true;

        link = new int[n2+1];

        int sum = 0;
        for(int i = 1; i <= n1; i++){
            vis = new boolean[n2+1];
            if(find(i))
                sum++;
        }
        System.out.println(sum);
    }
}
~~~
