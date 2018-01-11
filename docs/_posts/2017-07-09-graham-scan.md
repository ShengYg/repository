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


~~~python
class Point(object):
    def __init__(self, a=0, b=0):
        self.x = a
        self.y = b
        
    def __str__(self):
        return "{}, {}".format(self.x, self.y)

def comp(a, b):
    if a.y == b.y:
        return 1 if a.x - b.x > 0 else -1
    return 1 if a.y - b.y > 0 else -1

# counterclockwise
def cck(p, a, b):
    return (a.x - p.x) * (b.y - p.y) - (a.y - p.y) * (b.x - p.x)

def dis(p, a):
    return (a.x - p.x) ** 2 + (a.y - p.y) ** 2

def polarorder(p, a, b):
    area = cck(p, a[0], b[0])
    if area == 0:
        dis1 = dis(p, a[0])
        dis2 = dis(p, b[0])
        if dis1 < dis2:
            a[1] = False
        else:
            b[1] = False
        return 1 if dis1 - dis2 > 0 else -1
    else:
        return 1 if - area > 0 else -1

def GrahamScan(points):
    hull = []
    n = len(points)
    point_list = [[p, True] for p in points]
    
    point_list = sorted(point_list, cmp=lambda a, b: comp(a[0], b[0]))
    hull.append(point_list[0][0])
    
    point_list = sorted(point_list[1:], cmp=lambda a, b: polarorder(point_list[0][0], a, b))
    point_list = [p[0] for p in point_list if p[1]]
    
    if len(point_list) < 2:
        return
    hull.append(point_list[0])
    hull.append(point_list[1])
    for i in range(2, len(point_list)):
        p = point_list[i]
        top = hull.pop()
        while(cck(hull[-1], top, p) <= 0):
            top = hull.pop()
        hull.append(top)
        hull.append(point_list[i])
    return hull
~~~
