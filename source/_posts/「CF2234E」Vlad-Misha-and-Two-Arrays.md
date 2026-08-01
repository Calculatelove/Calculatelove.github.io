---
title: 「CF2234E」Vlad, Misha and Two Arrays
date: 2026-08-01 20:46:03
updated: 2026-08-01 20:46:03
categories: Codeforces
tags:
  - 启发式分裂
  - 最值分治
---

# Description

Link：[CF2234E](https://codeforces.com/problemset/problem/2234/E)

{% note default %}

对于一个长度为 $n$ 的排列 $p$，记 $a_i$ 表示有多少个区间 $[l, r]$ ($1 \leq l \leq r \leq n$) 满足 $p_l, \dots, p_r$ 中的最小值为 $p_i$。

现给出 $a_1, \dots, a_n$，求出有多少个可能的排列 $p$。

数据范围：$1 \leq n \leq 5\times 10^5$，$1 \leq a_i \leq 10^{12}$。

时空限制：$2$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

一个比较歪的方向：根据 $a$ 数组算出每个数左边和右边第一个比它小的数 ...

关注一下最小值 $1$：假设 $1$ 在位置 $i$，那么必有 $a_i = i(n - i + 1)$。**注意到 $1$ 将区间分成了独立的左右两部分**，因为跨过位置 $i$ 的区间，最小值必定是 $1$。先选出一部分的数分给左边，另一部分分给右边，此时的划分方案数为 $\binom{n - 1}{i - 1}$。

于是考虑分治。设 $f(l, r)$ 表示 $a_l, \dots, a_r$ 能复原多少个关于 $r - l + 1$ 的排列。先找到最小值的位置 $p$，满足 $a_p = (p - l + 1)(r - p + 1)$，然后再将剩余的数划分给左右两部分，于是有

$$
f(l, r) = f(l, p - 1)\times f(p + 1, r)\times \binom{r - l}{p - l}
$$

但是寻找 $p$ 的过程，不能正着扫一遍或者反着扫一遍，否则都会被构造数据卡成 $\mathcal{O}(n^2)$。

每次从左侧取一个数判断，再从右侧取一个数判断。如此往复直到找到满足条件的 $p$ 为止。容易发现枚举 $p$ 的开销，**取决于较小区间的长度**。本质上是一个启发式分裂。

时间复杂度 $\mathcal{O}(n \log n)$。

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

const int mod = 1e9 + 7;

template <class T>
inline int qpow(int a, T b, int p) {
    int ans = 1;
    for (; b; b >>= 1) {
        if (b & 1) ans = 1ll * ans * a % p;
        a = 1ll * a * a % p;
    }
    return ans;
}

struct BinomCoef {
    std::vector<int> fact, facv;

    BinomCoef() {}
    BinomCoef(int n) {
        init(n);
    }

    void init(int n) {
        fact.resize(n + 1), facv.resize(n + 1);
        fact[0] = 1;
        for (int i = 1; i <= n; i ++) {
            fact[i] = 1ll * fact[i - 1] * i % mod;
        }
        facv[n] = qpow(fact[n], mod - 2, mod);
        for (int i = n - 1; i >= 0; i --) {
            facv[i] = 1ll * facv[i + 1] * (i + 1) % mod;
        }
    }

    int binom(int n, int m) {
        if (n < m || m < 0) {
            return 0;
        }
        return 1ll * facv[m] * facv[n - m] % mod * fact[n] % mod;
    }
} bc(500000);

const int N = 500100;

int n;
i64 a[N];

int solve(int l, int r) {
    if (l > r) {
        return 1;
    }
    auto check = [&] (int i) {
        return a[i] == 1ll * (i - l + 1) * (r - i + 1);
    };

    int i = l, j = r;
    while (i <= j) {
        if (i <= j) {
            if (check(i)) {
                return 1ll * solve(l, i - 1) * solve(i + 1, r) % mod * bc.binom(r - l, i - l) % mod;
            }
            i ++;
        }
        if (i <= j) {
            if (check(j)) {
                return 1ll * solve(l, j - 1) * solve(j + 1, r) % mod * bc.binom(r - l, j - l) % mod;
            }
            j --;
        }
    }
    return 0;
}

void work() {
    std::cin >> n;

    for (int i = 1; i <= n; i ++) {
        std::cin >> a[i];
    }

    std::cout << solve(1, n) << '\n';
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