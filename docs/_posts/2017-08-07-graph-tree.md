---
layout: post
title:  "图算法"
date:   2017-08-07 14:00:00 +0800
categories: [algorithms]
tags: [tree, graph]
description: 基本图算法、图与树、图与树、最短路径、欧拉图
---

## Table of Contents

- [基本图算法](#1)
  - [有向无环图拓扑排序](#1.1)
- [图与树](#2)
  - [无向图是树](#2.1)
    - [union-find](#2.1.1)
  - [有向图是树](#2.2)
- [最小生成树](#3)
  - [kruskal](#3.1)
- [最短路径](#4)
  - [单源最短路径 Dijkstra](#4.1)
  - [所有节点对的最短路径 Floyd](#4.2)
  - [单源最短路径 SPFA](#4.3)
- [欧拉图](#5)
- [graham scan](#6)

---

<a name='1'></a>
## 基本图算法
<a name='1.1'></a>
### 有向无环图的拓扑排序
深度优先搜索时加入每个点的访问结束时间，按结束时间逆排序。

~~~java
class Solution(object):
    visited = None
    f_time = None
    time = None

    def DFS(self, n, edges):
        graph = [[] for _ in range(n)]
        for item in edges:
            graph[item[0]].append(item[1])
        self.visited = [0] * n
        self.time = 0
        self.f_time = [0] * n
        for i in range(n):
            if not self.visited[i]:
                try:
                    self.DFS_visit(graph, i)
                except:
                    return []
        ret = zip(self.f_time, range(n))
        ret = sorted(ret, key=lambda x: x[0])[::-1]
        return [item[1] for item in ret]

    def DFS_visit(self, graph, i):
        self.time += 1
        self.visited[i] = 1
        for j in graph[i]:
            if not self.visited[j]:
                self.DFS_visit(graph, j)
            elif self.visited[j] == 1:
                raise
        self.visited[i] = -1
        self.time += 1
        self.f_time[i] = self.time
~~~

<a name='2'></a>
## 图与树
<a name='2.1'></a>
### 无向图是树

Given n nodes labeled from 0 to n - 1 and a list of undirected edges (each edge is a pair of nodes), write a function to check whether these edges make up a valid tree.

思路：一棵树必须具备如下特性：

1. 是一个全连通图（所有节点相通）
2. 无回路

~~~java
public class Solution {
    public boolean validTree(int n, int[][] edges) {
        if (edges.length != n-1) return false;  
        int[] roots = new int[n];  
        for(int i=0; i<n; i++) roots[i] = i;  
        for(int i=0; i<edges.length; i++) {  
            int root1 = root(roots, edges[i][0]);  
            int root2 = root(roots, edges[i][1]);  
            if (root1 == root2) return false;  
            roots[root2] = root1;  
        }  
        return true;  
    }
    private int root(int[] roots, int id) {
        if (id == roots[id]) return id;  
        return root(roots, roots[id]);  
    }
} 
~~~

<a name='2.1.1'></a>
#### Union-Find
~~~cpp
int N;
const int MAX_SIZE = 100010;
int root[MAX_SIZE];

int get(int id){
    if(id == root[id])
        return id;
    return get(root[id]);
}

int main() {
    cin >> N;
    int cnt = 0;
    for(int i=0; i<MAX_SIZE; i++)
        root[i] = i;
    vector<string> ret;
    while(N--){				// n组数据
        int choose;
        int id1, id2;
        cin >> choose >> id1 >> id2;

        if(choose == 0){		// 在线设置
            int root1 = get(id1);
            int root2 = get(id2);
            root[root2] = root1;
        }else{				// 在线查询
            int root1 = get(id1);
            int root2 = get(id2);
            if (root1 == root2)
                ret.push_back("yes");
            else
                ret.push_back("no");
        }
    }
    for_each(ret.begin(), ret.end(), [](string s){ cout << s << endl; });
};
~~~

<a name='2.2'></a>
### 有向图是树

深度优先遍历，记录每个节点访问次数visited：

0：未访问

1：访问一次

2：访问多次

当下一个要访问的点为1时，表示存在后向边，即存在环，直接抛出异常，返回False

~~~java
class Solution(object):
    visited = None

    def DFS(self, n, edges):
        graph = [[] for _ in range(n)]
        for item in edges:
            graph[item[0]].append(item[1])
        self.visited = [0] * numCourses
        for i in range(n):
            if not self.visited[i]:
                try:
                    self.DFS_visit(graph, i)
                except:
                    return False
        return True

    def DFS_visit(self, graph, i):
        self.visited[i] = 1
        for j in graph[i]:
            if not self.visited[j]:
                self.DFS_visit(graph, j)
            elif self.visited[j] == 1:
                raise
        self.visited[i] = -1
~~~



<a name='3'></a>
## 最小生成树
<a name='3.1'></a>
### kruskal

~~~java
class Edge{
    public int u;
    public int v;
    public double weight;

    public Edge(int u, int v, double weight){
        this.v = v;
        this.u = u;
        this.weight = weight;
    }
}

public class KruskalMST {
    public static HashSet<Integer>[] adj;
    public static int[] union;
    public static ArrayList<Edge> ret;

    public static int find_union(int u){
        while(union[u] != u)
            u = union[u];
        return u;
    }

    public static void main(String[] args){
        Scanner in = new Scanner(System.in);
        PriorityQueue<Edge> pq = new PriorityQueue<>((a, b) -> {
            if(a.weight > b.weight)
                return 1;
            else if(a.weight < b.weight)
                return -1;
            else
                return 0;
        });
        int N = in.nextInt(); // N vertices
        int M = in.nextInt(); // M edges
        adj = new HashSet[N];
        Arrays.fill(adj, new HashSet<>());
        while(M-- != 0){
            int u = in.nextInt();
            int v = in.nextInt();
            double weight = in.nextDouble();
            pq.add(new Edge(u, v, weight));
            adj[u].add(v);
            adj[v].add(u);
        }
        union = new int[N];
        for(int i = 0; i < N; i++)
            union[i] = i;
        ret = new ArrayList<>();
        while(!pq.isEmpty()){
            Edge e = pq.poll();
            int root1 = find_union(e.u);
            int root2 = find_union(e.v);
            if(root1 != root2) {
                union[root1] = root2;
                ret.add(e);
            }
        }
        for(Edge e: ret){
            System.out.print(e.u);
            System.out.println(e.v);
        }
    }
}
~~~


<a name='4'></a>
## 最短路径
<a name='4.1'></a>
### 单源最短路径 Dijkstra

> Dijkstra算法：**非负权重有向图**上单源最短路径

复杂度：`O(V^2)`

<center>
<img src="{{ site.baseurl }}/assets/pic/dijkstra.png" height="600px" >
</center>


~~~cpp
vector<int> dist;
unordered_map<int, vector<pair<int, int>>> graph;
bool seen[101]={false};
int Dijkstra(int N, int S, int E) {
    // N: vertex
    // S: start node
    // E: end node
    // graph: node -> [(node, weight)...]
    // seen
    // dist
    for(int i = 0; i <= N; i++)
        dist.push_back(numeric_limits<int>::max());
    dist[S] = 0;

    while(true){
        int candNode = -1;
        int candDist = numeric_limits<int>::max();
        for(int i = 1; i <= N; i++){
            if(!seen[i] && dist[i] < candDist){
                candDist = dist[i];
                candNode = i;
            }
        }

        if(candNode<0)
            break;
        seen[candNode] = true;
        if(graph.find(candNode)!=graph.end()){
            for(pair<int, int> p: graph.find(candNode)->second){
                dist[p.first] = min(dist[p.first], dist[candNode]+p.second);
            }
        }
    }
    return dist[E];
}
~~~

<a name='4.2'></a>
### 所有节点对的最短路径 Floyd

设$d_{ij}^{(k)}$为从节点i到节点j所有中间节点取自${1,2,...,k}$的一条最短路径。根据动态规划得到递推式:

$$
d_{ij}^{(k)} = 
\begin{cases}
w_{ij} & k=0 \\
min(d_{ij}^{(k-1)}, d_{ik}^{(k-1)}+d_{kj}^{(k-1)}) & k>0
\end{cases}
$$

复杂度：`O(V^3)`

~~~cpp
int edge[101][101];

int main() {
    memset(edge, 0x3f, 101 * 101 * sizeof(int));
    int N, M;
    cin >> N >> M;
    for(int i=0; i<M; i++){
        int a,b,c;
        cin >> a >> b >> c;
        if (edge[a][b] > c)
            edge[a][b] = edge[b][a] = c;
    }

    for (int k = 1; k <= N; ++k) {
        for (int j = 2; j <= N; ++j) {
            edge[j][j] = 0;
            for (int i = 1; i < j; ++i) {
                if (k != i && k != j) {
                    int value = edge[i][k] + edge[k][j];
                    edge[j][i] = edge[i][j] = (edge[i][j] > value ? value : edge[i][j]);
                }
            }
        }
    }

    // output: edge
    return 0;
}
~~~



<a name='4.3'></a>
### 单源最短路径 SPFA
SPFA: Shortest Path Faster Algorithm

~~~cpp
const int N = 1e5 + 10;
const int M = 1e6 + 10;
int CNT;
int before[N];       // 上一条以i为起点的边
int dis[N];
struct Edge {
    int t;      // 终点
    int n;      // 上一条同样起点的边
    int weight;
} edge[M << 1];

void insert(int from, int to, int weight) {
    edge[CNT].t = to;
    edge[CNT].weight = weight;
    edge[CNT].n = before[from];
    before[from] = CNT++;
    edge[CNT].t = from;
    edge[CNT].weight = weight;
    edge[CNT].n = before[to];
    before[to] = CNT++;
}
bool relax(int &a, int b) { return a > b ? (a = b, true) : false; }

int disj(int s, int e) {
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pqueue;
    dis[s] = 0;
    pqueue.push(pair<int, int>(0, s));
    while (!pqueue.empty()) {
        int prev = pqueue.top().second;
        int d = pqueue.top().first;
        pqueue.pop();
        if (dis[prev] < d)
            continue;
        if (prev == e)
            return d;
        for (int i = before[prev], x; ~i; i = edge[i].n) {
            x = edge[i].t;
            if (relax(dis[x], edge[i].weight + d))
                pqueue.push(pair<int, int>(dis[x], x));
        }
    }
    return dis[e];
}

int main() {
    int n, m, s, e;
    cin >> n >> m >> s >> e;
    fill(before, before + n + 1, -1);
    fill(dis, dis + n + 1, INF);
    int u, v, w;
    for (int i = 0; i < m; i++) {
        cin >> u >> v >> w;
        insert(u, v, w);
    }
    cout << disj(s, e) << endl;
    return 0;
}
~~~


<a name='5'></a>
## 欧拉图

- 无向图欧拉回路：所有顶点的度数都为偶数。
- 有向图欧拉回路：所有顶点的出度与入读相等。
- 无向图欧拉路径： 之多有两个顶点的度数为奇数，其他顶点的度数为偶数。
- 有向图欧拉路径： 至多有两个顶点的入度和出度绝对值差1（若有两个这样的顶点，则必须其中一个出度大于入度，另一个入度大于出度）,其他顶点的入度与出度相等。

<a name='6'></a>
## graham scan

寻找凸包，时间复杂度$O(nlogn)$

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
