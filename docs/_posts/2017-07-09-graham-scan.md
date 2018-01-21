---
layout: post
title:  "图算法graham scan"
date:   2017-07-09 14:00:00 +0800
categories: [algorithms]
tags: [geometry]
description: 
---

获得凸包的算法可以算是计算几何中最基础的算法之一了。寻找凸包的算法有很多种，**Graham Scan算法**是一种十分简单高效的二维凸包算法，能够在$O(nlogn)$的时间内找到凸包。

<center>
<img src="{{ site.baseurl }}/assets/pic/9_04.gif" height="300px" >
</center>

**步骤：**
1. 按Y坐标对点排序，Y坐标最低点为Y0（Y相同则取X最小点）
1. 剩下点按与Y0的极角的逆时针顺序排序，角度相同则忽略与Y0较近的点（欧氏距离）
1. 将Y0，Y1，Y2压入栈stack
1. 循环取Yi，栈顶元素top1，top2，若top2，top1，Yi没有向左转，则弹出top，直到满足条件，压入Yi
1. 输出stack


~~~cpp
struct node {
    int x;
    int y;
    int id;
} node[100005], output[100005];
int n;

int cross(node p0, node p1, node p2) {
    //计算叉乘，注意p0,p1,p2的位置，这个决定了方向
    return (int)(p1.x-p0.x)*(p2.y-p0.y)-(int)(p1.y-p0.y)*(p2.x-p0.x);
}

int dis(node a, node b) {
    //计算距离，这个用在了当两个点在一条直线上
    return(int)(a.x-b.x)*(a.x-b.x)+(int)(a.y-b.y)*(a.y-b.y);
}
bool cmp(node p1, node p2) {
    //极角排序
    int z = cross(node[0], p1, p2);
    if( z>0 || (z==0 && dis(node[0],p1) < dis(node[0],p2)))
        return 1;
    return 0;
}
void Graham() {
    int k=0;
    for(int i=0; i<n; i++)
        if(node[i].y<node[k].y || (node[i].y==node[k].y&&node[i].x<node[k].x))
            k = i;
    swap(node[0], node[k]);         //找左下的点p0
    sort(node+1, node+n, cmp);      //以node[0]为基点排序
    int top = 1;
    output[0] = node[0];
    output[1] = node[1];
    for(int i=2; i<n; i++) {        //控制进栈出栈
        while(cross(output[top-1], output[top], node[i]) < 0 && top)
            top--;
        top++;
        output[top] = node[i];
    }
}

int main() {
    cin >> n;
    for(int i=0; i<n; i++) {
        cin >> node[i].x >> node[i].y;
        node[i].id = i;
    }
    Graham();
}
~~~
