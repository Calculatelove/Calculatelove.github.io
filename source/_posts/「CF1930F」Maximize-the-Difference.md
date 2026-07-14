---
title: 「CF1930F」Maximize the Difference
date: 2026-07-14 10:39:50
updated: 2026-07-14 10:39:50
categories: Codeforces
tags:
  - 位运算
  - 记忆化搜索
---

# Description

Link：[CF1930F](https://codeforces.com/contest/1930/problem/F)

{% note default %}

对于一个长度为 $m$ 的非负数组 $b$，定义 $f(b)$ 等于

$$
\max_{x\geq 0}\left\{ \max_{i = 1}^m(b_i \operatorname{|} x) - \min_{i = 1}^m(b_i \operatorname{|} x) \right\}
$$

给出值域上限 $n$ 以及操作次数 $Q$，初始时数组 $a$ 为空，有 $Q$ 次**强制在线**的操作：每次操作在数组 $a$ 的末尾加入一个数 $v$ ($0 \leq v < n$)，你需要求出此时的 $f(a)$。

数据范围：$1 \leq n \leq 2^{22}$，$1 \leq Q \leq 10^6$。

时空限制：$3$s / $256$MiB

{% endnote %}

<!-- more -->

# Solution

不妨先求出，对于任意非负整数 $p, q$ 的 $\max\limits_{x\geq 0}\left\{ (p \operatorname{|} x) - (q \operatorname{|} x) \right\}$。发现二进制下的每一位是独立的，对于某一位，只有 $p, q$ 在这一位上分别为 $1, 0$ 时才有贡献。

于是上式的值，可以被表示为 $(p \operatorname{|} q) - q$，也可以被表示为 $p - (p \operatorname{\&} q)$。

注意到原式的**最大化极差**可以放宽为**最大化任意两个数的差**，即

$$
\max_{x\geq 0}\max_{1 \leq i, j \leq m} \left\{ (b_i \operatorname{|} x) - (b_j \operatorname{|} x) \right\} \\
\max_{1 \leq i, j \leq m}\max_{x\geq 0} \left\{ (b_i \operatorname{|} x) - (b_j \operatorname{|} x) \right\} \\
\max_{1 \leq i, j \leq m} \left\{ (b_i \operatorname{|} b_j) - b_j \right\}
$$

考虑新加入一个数 $v$ 对于 $f(a)$ 的影响，当其作为 $b_j$ 的时候，式子为 $(b_i \operatorname{|} b_j) - b_j$，相当于要最大化 $b_i \operatorname{|} v$。

记一个数组 `f[u]` 表示是否有一个数的子集为 $u$。首先 $v$ 中已经为 $1$ 的那些位就可以不用管了，**剩余的位可以从高到低位贪心**，根据 `f` 数组来判断是否可以扩展这一位。

至于 `f` 数组的更新，可以使用类似 [2025 ICPC 南京站 F](https://qoj.ac/contest/2581/problem/14806) 的方式，每次更改一个位地往下记忆化搜索。

当其作为 $b_i$ 的时候，式子为 $b_i - (b_i \operatorname{\&} b_j)$，相当于要最小化 $b_j \operatorname{\&} v$，这是一个对称的问题。

时间复杂度 $\mathcal{O}(n \log n + Q \log n)$。

```c++
#include <bits/stdc++.h>

using i64 = long long;

#define debug(a) std::cout << #a << '=' << (a) << ' '

template <class T>
inline void chmin(T &x, const T &y) {
    if (x > y) {
        x = y;
    }
}
template <class T>
inline void chmax(T &x, const T &y) {
    if (x < y) {
        x = y;
    }
}

const int M = 1 << 22;

int n, Q, m;
int f[M]; // f[u]：是否有数的子集为 u
int g[M]; // g[u]：是否有数的超集为 u

void addf(int u) {
    if (f[u]) {
        return;
    }
    f[u] = 1;
    for (int i = 0; i < m; i ++) {
        if (u >> i & 1) {
            addf(u ^ (1 << i));
        }
    }
}
int askOr(int u) {
    int v = 0;
    for (int i = m - 1; i >= 0; i --) {
        if (!(u >> i & 1) && f[v ^ (1 << i)]) {
            v ^= (1 << i);
        }
    }
    return v | u;
}

void addg(int u) {
    if (u >= (1 << m) || g[u]) {
        return;
    }
    g[u] = 1;
    for (int i = 0; i < m; i ++) {
        if (!(u >> i & 1)) {
            addg(u ^ (1 << i));
        }
    }
}
int askAnd(int u) {
    int v = (1 << m) - 1;
    for (int i = m - 1; i >= 0; i --) {
        if ((u >> i & 1) && g[v ^ (1 << i)]) {
            v ^= (1 << i);
        }
    }
    return v & u;
}

void work() {
    std::cin >> n >> Q;

    m = 1;
    while ((1 << m) < n) {
        m ++;
    }

    int lastans = 0;
    for (int i = 1; i <= Q; i ++) {
        int u;
        std::cin >> u;
        u = (u + lastans) % n;

        chmax(lastans, askOr(u) - u);
        addf(u);

        chmax(lastans, u - askAnd(u));
        addg(u);

        std::cout << lastans << ' ';
    }
    std::cout << '\n';

    for (int i = 0; i < (1 << m); i ++) {
        f[i] = g[i] = 0;
    }
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    int T;
    std::cin >> T;

    while (T --) {
        work();
    }

    return 0;
}
```