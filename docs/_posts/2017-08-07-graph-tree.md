---
layout: post
title:  "一些图算法"
date:   2017-08-07 14:00:00 +0800
categories: [algorithms]
tags: [tree, graph]
description: 判断有向图、无向图中是否有环路，有向无环图拓扑排序
---

## Table of Contents

- [无向图是树](#1)
- [有向图是树](#2)
- [有向无环图拓扑排序](#3)
- [最小生成树](#4)
- [欧拉图](#5)
- [单源最短路径](#6)

---

<a name='1'></a>
## 无向图是树

Given n nodes labeled from 0 to n - 1 and a list of undirected edges (each edge is a pair of nodes), write a function to check whether these edges make up a valid tree.

思路：一棵树必须具备如下特性：

1. 是一个全连通图（所有节点相通）
2. 无回路

### 方法一：广度优先搜索。

时间复杂度O(n)

空间复杂度O(n)

~~~java
public class Solution {
    public boolean validTree(int n, int[][] edges) {
        Map<Integer, Set<Integer>> graph = new HashMap<>();  
        for(int i=0; i<edges.length; i++) {  
            for(int j=0; j<2; j++) {  
                Set<Integer> pairs = graph.get(edges[i][j]);  
                if (pairs == null) {  
                    pairs = new HashSet<>();  
                    graph.put(edges[i][j], pairs);  
                }  
                pairs.add(edges[i][1-j]);  
            }  
        }  
        Set<Integer> visited = new HashSet<>();
        Set<Integer> current = new HashSet<>(); 
	
	// BFS
        visited.add(0);  
        current.add(0);  
        while (!current.isEmpty()) {  
            Set<Integer> next = new HashSet<>();  
            for(Integer node: current) {  
                Set<Integer> pairs = graph.get(node);  
                if (pairs == null) continue;  
                for(Integer pair: pairs) {  
                    if (visited.contains(pair)) return false;  
                    next.add(pair);  
                    visited.add(pair);  
                    graph.get(pair).remove(node);  
                }  
            }  
            current = next;  
        }  
        return visited.size() == n;  
    }
}
~~~

### 方法二：深度优先搜索，搜索目标是遍历全部节点。

~~~java
public class Solution {
    private boolean[] visited;
    private int visits = 0;
    private boolean isTree = true;
    private void check(int prev, int curr, List<Integer>[] graph) {
        if (!isTree) return;  
        if (visited[curr]) {  
            isTree = false;  
            return;  
        }  
        visited[curr] = true;  
        visits ++;  
        for(int next: graph[curr]) {  
            if (next == prev) continue;  
            check(curr, next, graph);  
            if (!isTree) return;  
        }  
          
    }
    public boolean validTree(int n, int[][] edges) {
        visited = new boolean[n];  
        List<Integer>[] graph = new List[n];  
        for(int i=0; i<n; i++) graph[i] = new ArrayList<>();  
        for(int[] edge: edges) {  
            graph[edge[0]].add(edge[1]);  
            graph[edge[1]].add(edge[0]);  
        }  
        check(-1, 0, graph);  
        return isTree && visits == n;  
    }
}
~~~

### 方法三：按节点大小对边进行排序，原理类似并查集。

~~~java
public class Solution {
    public boolean validTree(int n, int[][] edges) {
        if (edges.length != n-1) return false;  
        Arrays.sort(edges, new Comparator<int[]>() {  
           @Override  
           public int compare(int[] e1, int[] e2) {  
               return e1[0] - e2[0];  
           }  
        });  
        int[] sets = new int[n];  
        for(int i=0; i<n; i++) sets[i] = i;  
        for(int i=0; i<edges.length; i++) {  
            if (sets[edges[i][0]] == sets[edges[i][1]]) return false;  
            if (sets[edges[i][0]] == 0) {  
                sets[edges[i][1]] = 0;  
            } else if (sets[edges[i][1]] == 0) {  
                sets[edges[i][0]] = 0;  
            } else {  
                sets[edges[i][1]] = sets[edges[i][0]];  
            }  
        }  
        return true;  
    }
}
~~~

### 方法四：Union-Find

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

<a name='2'></a>
## 有向图是树

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
## 有向无环图的拓扑排序

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

<a name='4'></a>
## 最小生成树

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

<a name='5'></a>
## 5.欧拉图

- 无向图欧拉回路：所有顶点的度数都为偶数。
- 有向图欧拉回路：所有顶点的出度与入读相等。
- 无向图欧拉路径： 之多有两个顶点的度数为奇数，其他顶点的度数为偶数。
- 有向图欧拉路径： 至多有两个顶点的入度和出度绝对值差1（若有两个这样的顶点，则必须其中一个出度大于入度，另一个入度大于出度）,其他顶点的入度与出度相等。

<a name='6'></a>
## 6.单源最短路径

Dijkstra算法：非负权重有向图上单源最短路径
~~~cpp
class Solution {
public:
    vector<int> dist;
    unordered_map<int, vector<pair<int, int>>> graph;
    bool seen[101]={false};
    int Dijkstra(int N, int K) {
        // N: edges
        // K: start node
        // graph: node -> [(node, weight)...]
		// seen
		// dist
        for(int i = 0; i <= N; i++)
            dist.push_back(numeric_limits<int>::max());
        dist[K] = 0;

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

        int ret = 0;
        for(int i = 1; i <= N; i++){
            int cand = dist[i];
            if(cand == numeric_limits<int>::max())
                return -1;
            ret = max(ret, cand);
        }
        return ret;
    }
};
~~~




