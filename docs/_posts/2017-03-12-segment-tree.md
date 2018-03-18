---
layout: post
title:  "线段树 Segment-tree"
date:   2017-03-12 16:00:00 +0800
categories: [algorithms]
tags: [tree]
description: 介绍一种树形数据结构 —— Segment Tree。线段树是一种二叉树形结构，属于**平衡树**的一种。它将线段区间组织成树形的结构，并用每个节点来表示一条线段。
---

## RMQ-ST
区间最值（Range Minimum/Maximum Query）。

思路：把区间拆成长度为类似1、2、4的2的幂，预处理得到每个区间的最值。设`pre[i, j]`表示从j开始长度为2^i的区间的最值，则区间`[Li,Ri]`的最小值为`min{pre[T, Li], pre[T, Ri-T+1]}`，T为小于这个区间长度的最大的2的幂。

注意，与线段树的区别：
1. 数值固定，不能支持更新
2. 区间长度为2的幂，因此左右区间有重叠

C++代码：
~~~cpp
const int MAXNUM = 1000010;
int pre[20][MAXNUM];

int main() {
    int N, Q;
    int l, r, len;
    cin >> N;
    for(int i = 0; i < N; i++)
        scanf("%d", &pre[0][i]);
    for(int i = 1; (1 << i) <= N; i++)			// 循环，得到每个区间的最值
        for(int j = 0; j <= N - (1 << i); j++)
            pre[i][j] = min(pre[i-1][j], pre[i-1][j+(1 << (i-1))]);

    vector<int> ret;
    cin >> Q;
    while(Q--) {					// 多次查询
        scanf("%d%d", &l, &r);
        len = r - l + 1;
        int i;
        for(i = 0; len != 1; i++, len >>= 1);		// i为小于这个区间长度的最大的2的幂
        ret.push_back(min(pre[i][l], pre[i][r-(1<<i)+1]));
    }

    for_each(ret.begin(), ret.end(), [](int i){ cout << i << endl; });
}
~~~

## 线段树

线段树是一种二叉树形结构，属于**平衡树**的一种。它将线段区间组织成树形的结构，并用每个节点来表示一条线段。

![]({{ site.baseurl }}/assets/pic/4_00.jpg)

两种情况：
1. 离散型：区间**每个点**有意义，叶子节点是[i, i]
2. 连续性：区间**每一段**有意义，叶子节点是[i, i + 1]

C++代码：
1. 区间`[i,j)`左闭右开
2. 每次置位一个数时，不需要修改`lazy`，代码可以直接使用，此时`update(i,j)`中`j=i+1`。
3. 每次置位多个数时，可能需要修改`lazy`的更新模式，在函数`pd()`、`modify()`中。

~~~cpp
const int INF = 0x3f3f3f3f;

template <typename T, class op = plus<T>>
class Segtree {
private:
    struct node {
        int l, r;
        T val, lazy = INF;	    // val存储题目需要的结果，lazy缓存节点值
        node() {}
        node(int l, int r, T v) : l(l), r(r), val(v) {}
        void modify(T k) { val = k; lazy = k;}		// 可能会变
    };
    vector<node> tree;

    void pd(int now) {
        if (tree[now].lazy == INF)
            return;
        if(tree[now].r == tree[now].l + 1)
            return;
        tree[now * 2 + 1].modify(tree[now].lazy);
        tree[now * 2 + 2].modify(tree[now].lazy);
        tree[now].lazy = INF;
    }

    template <typename U>
    void cre(int l, int r, int now, const vector<U>& a){
        if (l + 1 == r) {
            tree[now] = node(l, r, a[l]);
        }
        else {
            int mid = (l + r) / 2;
            cre(l, mid, now * 2 + 1, a);
            cre(mid, r, now * 2 + 2, a);
            tree[now] = node(l, r, op()(tree[now * 2 + 1].val, tree[now * 2 + 2].val));
        }
    }

public:
    Segtree() {}
    Segtree(int n) : Segtree(vector<T>(n)) {}
    Segtree(const vector<T>& a) {
        int n = a.size();
        tree.resize(4 * n);
        cre<T>(0, n, 0, a);
    }
    void update(int l, int r, T val, int now = 0) {
        if (l >= r || tree[now].r <= l || tree[now].l >= r)
            return;
        else if (tree[now].r <= r && tree[now].l >= l) {
            tree[now].modify(val);
        }
        else {
            pd(now);
            update(l, r, val, now * 2 + 1);
            update(l, r, val, now * 2 + 2);
            tree[now].val = op()(tree[now * 2 + 1].val, tree[now * 2 + 2].val);
        }
    }
    T query(int l, int r, int now = 0) {
        if (l >= r || tree[now].r <= l || tree[now].l >= r)
            return identity_element(op());
        else if (tree[now].r <= r && tree[now].l >= l){
            return tree[now].val;
        }
        else {
            pd(now);
            return op()(query(l, r, now * 2 + 1), query(l, r, now * 2 + 2));
        }
    }
};

template<typename T>
struct Min {
    T operator()(const T& a, const T& b) const { return min(a, b); }
    friend T identity_element(const Min& s) { return INF; }
};

int main() {
    int N;
    scanf("%d", &N);

    vector<int> init{INF};				// 注意init[0]的值
    for(int i=1; i<=N; i++){
        int val;
        scanf("%d", &val);
        init.push_back(val);
    }
    Segtree<int, Min<int>> xds(init);			// 不同操作符

    int Q;
    scanf("%d", &Q);
    while(Q--){
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);
        if(a == 0){
            printf("%d\n", xds.query(b, c+1));
        }else{
            xds.update(b, b+1, c);
        }

    }
    return 0;
}
~~~

求和版本：[hihocoder#1078](http://hihocoder.com/problemset/solution/1262169)

