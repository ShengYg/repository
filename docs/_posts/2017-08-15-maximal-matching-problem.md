---
layout: post
title:  "一些图算法"
date:   2017-08-15 14:00:00 +0800
categories: [algorithms]
tags: [graph]
description: 二分图最大匹配
---

~~~java
public class Main {
    static boolean[] vis;
    static int[] link;
    static int[][] map;

    public static boolean find(int x, int n2){
        for(int i=1; i<=n2;i++) {
            if(map[x][i] != 0 && !vis[i]) {
                vis[i]=true;
                if(link[i]==0||find(link[i], n2)){
                    link[i]=x;
                    return true;
                }
            }
        }
        return false;
    }

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);

        int n1 = 4;
        int n2 = 4;
        map = new int[n1+1][n2+1];
        map[1][1] = 1;
        map[1][3] = 1;
        map[2][1] = 1;
        map[3][1] = 1;
        map[3][2] = 1;
        map[4][3] = 1;
        map[4][4] = 1;

        link = new int[n2+1];

        int s = 0;
        for(int i=1;i<=n1;i++){
            vis = new boolean[n2+1];
            if(find(i, n1))
                s++;
        }
        System.out.println(s);
    }
}
~~~
