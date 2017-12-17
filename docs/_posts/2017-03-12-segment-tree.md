---
layout: post
title:  "Segment-tree 线段树"
date:   2017-03-12 16:00:00 +0800
categories: [algorithms]
tags: [tree]
description: 介绍一种树形数据结构 —— Segment Tree。线段树是一种二叉树形结构，属于**平衡树**的一种。它将线段区间组织成树形的结构，并用每个节点来表示一条线段。
---

## 1. 简介

线段树是一种二叉树形结构，属于**平衡树**的一种。它将线段区间组织成树形的结构，并用每个节点来表示一条线段。

![]({{ site.baseurl }}/assets/pic/4_00.jpg)

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

~~~cpp
template <typename T, class op = plus<T>>
class SegTree {
private:
    struct node {
        int l, r;
        T v, lazy = INF;
        node() {}
        node(int l, int r, T v) : l(l), r(r), v(v) {}
        void modify(T k) {
            v = k;
            lazy = k;
        }
    };
    vector<node> tr;
    void pd(int now) {
        if (tr[now].lazy == INF) return;
        tr[now * 2 + 1].modify(tr[now].lazy);
        tr[now * 2 + 2].modify(tr[now].lazy);
        tr[now].lazy = INF;
    }

public:
    SegTree() {}
    SegTree(int n) : SegTree(vector<T>(n)) {}
    SegTree(const vector<T>& a) {
        int n = a.size();
        tr.resize(4 * n);
        function<void(int, int, int)> cre = [&](int l, int r, int now) {
            if (l + 1 == r) tr[now] = node(l, r, a[l]);
            else {
                int mid = (l + r) / 2;
                cre(l, mid, now * 2 + 1);
                cre(mid, r, now * 2 + 2);
                tr[now] = node(l, r, op()(tr[now * 2 + 1].v, tr[now * 2 + 2].v));
            }
        };
        cre(0, n, 0);
    }
    void update(int l, int r, T tag, int now = 0) {
        if (l >= r || tr[now].r <= l || tr[now].l >= r)
            return;
        else if (tr[now].r <= r && tr[now].l >= l)
            tr[now].modify(tag);
        else {
            pd(now);
            update(l, r, tag, now * 2 + 1);
            update(l, r, tag, now * 2 + 2);
            tr[now].v = op()(tr[now * 2 + 1].v, tr[now * 2 + 2].v);
        }
    }
    T query(int l, int r, int now = 0) {
        if (l >= r || tr[now].r <= l || tr[now].l >= r) return identity_element(op());
        else if (tr[now].r <= r && tr[now].l >= l) return tr[now].v;
        else {
            pd(now);
            return op()(query(l, r, now * 2 + 1), query(l, r, now * 2 + 2));
        }
    }
};

template<typename T>
struct Max {
    T operator()(const T& a, const T& b) const { return max(a, b); }
    friend T identity_element(const Max& s) { return -INF; }
};

int main() {
    SegTree<int, Max<int>> xds(100005);	// MAX
    vector<int> ret;
    int n;
    cin >> n;
    while (n--) {
        int l, r;
        cin >> l >> r;
        l--;
        int ans = xds.query(l, r) + 1;
        ret.push_back(ans);
        xds.update(l, r, ans);
    }
    for(int item: ret)
        cout << item << endl;
    return 0;
~~~


