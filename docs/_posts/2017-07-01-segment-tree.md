---
layout: post
title:  "Segment-tree 线段树"
date:   2017-07-01 16:00:00 +0800
categories: [algorithms]
tags: [tree]
description: 介绍一种树形数据结构 —— Segment Tree
---

## 1. 简介

线段树是一种二叉树形结构，属于**平衡树**的一种。它将线段区间组织成树形的结构，并用每个节点来表示一条线段。

![]({{ site.url }}/assets/pic/4_00.jpg)

线段树的每个节点表示了一条前闭后开，即[a,b)的线段，这样表示的好处在于，有时候处理的区间并不是连续区间，可能是离散的点，如数组等。这样就可以用[a,a＋1)的节点来表示一个数组的元素a，做到连续与离散的统一。

线段树的根节点表示了所要处理的最大线段区间，而叶节点则表示了形如[a,a+1)的单位区间。对于每个非叶节点（包括根节点）所表示的区间[s,t)，令`m = (s + t) / 2`，则其左儿子节点表示区间[s,m)，右儿子节点表示区间[m,t)。这样建树的好处在于，对于每条要处理的线段，可以类似“二分”的进入线段树中处理，使得时间复杂度在`O(logN)`量级，这也是线段树之所以高效的原因。

## 2. 性质

1. 线段树是一个平衡树，树的高度为`logN`。
1. 线段树把区间上的任意一条长度为L的线段都分成不超过2logL条线段的并。
1. 给定一个叶子p, 从根到p路径上所有结点代表的区间都包含点p, 且其他结点代表的区间都不包含点p
1. 线段树叶子节点的区间是单位长度，不能再分了。

## 3. 线段树的实现

~~~java
public class SegmentTree {

    public class SegmentTreeNode{
        int start, end;
        SegmentTreeNode left, right;
        int sum;

        public SegmentTreeNode(int start, int end){
            this.start = start;
            this.end = end;
            this.left = null;
            this.right = null;
            this.sum = 0;
        }
    }

    SegmentTreeNode root = null;

    public SegmentTree(int[] nums) {
        root = buildtree(nums, 0, nums.length - 1);
    }

    public SegmentTreeNode buildtree(int[] nums, int start, int end){
        if(start > end)
            return null;
        else {
            SegmentTreeNode ret = new SegmentTreeNode(start, end);
            if (start == end)
                ret.sum = nums[start];
            else {
                int mid = start + (end - start) / 2;
                ret.left = buildtree(nums, start, mid);
                ret.right = buildtree(nums, mid + 1, end);
                ret.sum = ret.left.sum + ret.right.sum;
            }
            return ret;
        }
    }

    public void update(int i, int val) {
        update(root, i, val);
    }

    public void update(SegmentTreeNode root, int pos, int val){
        if (root.start == root.end)
            root.sum = val;
        else {
            int mid = root.start + (root.end - root.start) / 2;
            if(pos <= mid)
                update(root.left, pos, val);
            else
                update(root.right, pos, val);
            root.sum = root.left.sum + root.right.sum;
        }
    }

    public int sumRange(int i, int j) {
        return sumRange(root, i, j);
    }

    public int sumRange(SegmentTreeNode root, int start, int end) {
        if (root.end == end && root.start == start) {
            return root.sum;
        } else {
            int mid = root.start + (root.end - root.start) / 2;
            if (end <= mid) {
                return sumRange(root.left, start, end);
            } else if (start >= mid+1) {
                return sumRange(root.right, start, end);
            }  else {
                return sumRange(root.right, mid+1, end) + sumRange(root.left, start, mid);
            }
        }
    }
}
~~~

~~~python
class SegmentTree(object):

    class SegmentTreeNode(object):
        def __init__(self, start, end):
            self.start = start
            self.end = end
            self.total = 0
            self.left = None
            self.right = None


    def __init__(self, nums):
        def buildtree(nums, start, end):
            if start > end:
                return None

            if start == end:
                n = SegmentTreeNode(start, end)
                n.total = nums[start]
                return n
            
            root = SegmentTreeNode(start, end)

            mid = start + (end - start) / 2
            root.left = buildtree(nums, start, mid)
            root.right = buildtree(nums, mid+1, end)
            root.total = root.left.total + root.right.total
                
            return root
        
        self.root = buildtree(nums, 0, len(nums)-1)
            
    def update(self, i, val):
        def updateVal(root, i, val):

            if root.start == root.end:
                root.total = val
                return val
        
            mid = root.start + (root.end - root.start) / 2
            
            if i <= mid:
                updateVal(root.left, i, val)
            else:
                updateVal(root.right, i, val)
            
            root.total = root.left.total + root.right.total
            return root.total

        return updateVal(self.root, i, val)

    def sumRange(self, i, j):
        def rangeSum(root, i, j):
            if root.start == i and root.end == j:
                return root.total
            
            mid = root.start + (root.end - root.start) / 2
            if j <= mid:
                return rangeSum(root.left, i, j)
            elif i >= mid + 1:
                return rangeSum(root.right, i, j)
            else:
                return rangeSum(root.left, i, mid) + rangeSum(root.right, mid+1, j)
        
        return rangeSum(self.root, i, j)
~~~




