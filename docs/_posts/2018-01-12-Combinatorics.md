---
layout: post
title:  "组合数"
date:   2018-01-12 14:00:00 +0800
categories: [algorithms, math]
tags: []
description: 组合数的计算
---

## 组合数C(n,m)的计算

### 方案一
利用$C_n^m=C_{n-1}^{m-1}+C_{n-1}^{m}$，预先计算存表。计算复杂度$O(n^2)$，查询复杂度$O(1)$，耗费空间，基本能处理10000以内的组合数。
~~~cpp
const int MAXN = 1000;
int C[MAXN+1][MAXN+1];
void Initial() {
    int i,j;
    for(i=0; i<=MAXN; ++i) {
        C[0][i] = 0;
        C[i][0] = 1;
    }
    for(i=1; i<=MAXN; ++i) {
        for(j=1; j<=MAXN; ++j)
            C[i][j] = (C[i-1][j] + C[i-1][j-1]) % M;
    }
}

int Combination(int n, int m) {
    return C[n][m];
}
~~~

### 方案二
对$C_n^m=\frac{n!}{m!*(n-m)!}$进行质因数分解，则分解后的结果为$C_n^m=p_1^{a_1-b_1-c_1}\cdot p_2^{a_2-b_2-c_2}\cdot ...\cdot p_k^{a_k-b_k-c_k}$。$n!$对p费解质因数的结果为$n/p+n/p^2+n/p^3...$

~~~cpp
const int MAXN = 1000000;
bool arr[MAXN+1] = {false};     // 辅助计算素数
vector<int> prim;               // 存素数

// 计算素数
void produce_prim_number() {
    prim.push_back(2);
    int i,j;
    for(i=3; i*i<=MAXN; i+=2) {
        if(!arr[i]) {
            prim.push_back(i);
            for(j=i*i; j<=MAXN; j+=i)
                arr[j] = true;
        }
    }
    while(i<=MAXN) {
        if(!arr[i])
            prim.push_back(i);
        i+=2;
    }
}

//计算n!中素因子p的指数
int Cal(int n, int p) {
    int ans = 0;
    long long rec = p;
    while(n>=rec) {
        ans += n/rec;
        rec *= p;
    }
    return ans;
}

//计算n^k对M取模，二分法
int Pow(long long n, int k, int M) {
    long long ans = 1;
    while(k) {
        if(k&1)
        {
            ans = (ans * n) % M;
        }
        n = (n * n) % M;
        k >>= 1;
    }
    return ans;
}

//计算C(n,m)
int Combination(int n, int m) {
    const int M = 10007;
    produce_prim_number();
    long long ans = 1;
    int num;
    for(int i=0; i<prim.size() && prim[i]<=n; ++i) {
        num = Cal(n, prim[i]) - Cal(m, prim[i]) - Cal(n-m, prim[i]);
        ans = (ans * Pow(prim[i], num, M)) % M;
    }
    return ans;
}
~~~

### 方案三
Lucas定理，计算$C_n^m$模质数p的结果，内容为$C_n^m$模p等于p进制数上各位的$C_{n_i}^{m_i}$模p的乘积

~~~cpp
const int M = 10007;
int ff[M+5];        // 记录n!
int p;              // 模质数p

//求最大公因数
int gcd(int a,int b) {
    if(b==0)
        return a;
    else
        return gcd(b, a%b);
}

//解线性同余方程，扩展欧几里德定理
int x,y;
void Extended_gcd(int b,int p) {
    if(p==0) {
        x=1;
        y=0;
    }
    else {
        Extended_gcd(p,b%p);
        long t=x;
        x=y;
        y=t-(b/p)*y;
    }
}

//计算不大的C(n,m)
int C(int a,int b) {
    if(b>a)
        return 0;
    b=(ff[a-b]*ff[b])%p;
    a=ff[a];
    int c=gcd(a,b);
    a/=c;
    b/=c;
    Extended_gcd(b,p);
    x=(x+p)%p;
    x=(x*a)%p;
    return x;
}

//Lucas定理
int Combination(int n, int m, int p) {
    int ans=1;
    int a,b;
    while(m||n) {
        a=n%p;
        b=m%p;
        n/=p;
        m/=p;
        ans=(ans*C(a,b))%p;
    }
    return ans;
}

int main() {
    int m,n;
    scanf("%d%d%d",&n, &m, &p);

    ff[0]=1;
    for(int i=1; i<=M; i++)             //预计算n!
        ff[i]=(ff[i-1]*i)%p;

    printf("%d\n",Combination(n,m,p));
    return 0;
}
~~~









