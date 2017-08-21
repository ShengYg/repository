---
layout: post
title:  "matrix math"
date:   2017-07-01 14:00:00 +0800
categories: [java]
tags: []
description: 
---

~~~java
class BasicMatrixMath{
    private int mod = 1000000007;
    public long[][] add(long[][] matrixa, long[][] matrixb) {
        for(int i = 0; i < matrixa.length; i++)
            for(int j = 0; j < matrixa[0].length; j++) {
                matrixa[i][j] = matrixa[i][j] + matrixb[i][j];
                matrixa[i][j] %= mod;
            }
        return matrixa;
    }

    public long[][] sub(long[][] matrixa, long[][] matrixb) {
        for(int i=0; i<matrixa.length; i++)
            for(int j=0; j<matrixa[0].length; j++) {
                matrixa[i][j] = matrixa[i][j] - matrixb[i][j];
                matrixa[i][j] %= mod;
            }
        return matrixa;
    }

    public long[][] mul(long[][] matrixa, long[][] matrixb) {
        long[][] result = new long[matrixa.length][matrixb[0].length];
        for(int i = 0; i < matrixa.length; i++) {
            for(int j = 0; j < matrixb[0].length; j++) {
                result[i][j] = 0;
                for (int k = 0; k < matrixa[0].length; k++) {
                    result[i][j] += matrixa[i][k] * matrixa[k][j];
                    result[i][j] %= mod;
                }
            }
        }
        return result;
    }

    public long[][] power(long[][] a, int n){
        if(n == 1)
            return a;
        long[][] c;
        long[][] b = power(a, n/2);
        c = mul(b, b);
        if(n % 2 == 0)
            return c;
        else{
            b = mul(c, a);
            return b;
        }
    }
}
~~~
