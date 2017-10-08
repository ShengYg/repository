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
                    result[i][j] += matrixa[i][k] * matrixb[k][j];
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

### 应用：递推式矩阵加速

递推式：

$$f_n = f_{n-1} + f_{n-2}$$

进行矩阵变换：

$$
\begin{bmatrix}
    f_{n+1} \\
    f_n \\
\end{bmatrix}
=
\begin{bmatrix}
    1 & 1 \\
    1 & 0 \\
\end{bmatrix}
\begin{bmatrix}
    f_n \\
    f_{n-1} \\
\end{bmatrix}
=
\begin{bmatrix}
    1 & 1 \\
    1 & 0 \\
\end{bmatrix}^n
\begin{bmatrix}
    f_1 \\
    f_0 \\
\end{bmatrix}
$$

用矩阵幂运算由$O(n)$加速到$O(log(n))$





