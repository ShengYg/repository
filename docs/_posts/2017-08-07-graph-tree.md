---
layout: post
title:  "有向图、无向图环路"
date:   2017-08-07 14:00:00 +0800
categories: [algorithms]
tags: [tree, graph]
description: 判断有向图、无向图中是否有环路
---

## 无向图

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

## 有向图

深度优先遍历，记录每个节点访问次数visited：

0：未访问

1：访问一次

2：访问多次

当下一个要访问的点为1时，表示存在后向边，即存在环，返回False

~~~java
class Solution(object):
    def DFS(self, n, edges):
        graph = [[] for _ in range(n)]
        for item in edges:
            graph[item[0]].append(item[1])
        visited = [0] * n
        for i in range(n):
            if not visited[i]:
                ret = self.DFS_visit(visited, graph, i)
                if not ret:
                    return False
        return True
        
    def DFS_visit(self, visited, graph, i):
        visited[i] = 1
        for j in graph[i]:
            if not visited[j]:
                ret = self.DFS_visit(visited, graph, j)
                if not ret:
                    return False
            elif visited[j] == 1:
                return False
        visited[i] = -1
        return True
~~~












