---
title: 「CF1930E」2..3...4.... Wonderful! Wonderful!
date: 2026-07-14 10:01:43
updated: 2026-07-14 10:01:43
categories: Codeforces
tags:
  - 组合数学
---

# Description

Link：[CF1930E](https://codeforces.com/contest/1930/problem/E)

{% note default %}

有一个长度为 $n$ 的数组 $a$，初始满足 $a_i = i$。对于一个参数 $k$ ($1 \leq k \leq \lfloor \frac{n - 1}{2} \rfloor$)，你可以对 $a$ 进行任意次数（可能为 $0$ 次）的以下操作：
- 从 $a$ 中选择一个长度为 $2k + 1$ 的子序列，然后删除其中的前 $k$ 个元素和后 $k$ 个元素。

求对于每个 $k$ ($1 \leq k \leq \lfloor \frac{n - 1}{2} \rfloor$)，最终可以得到多少个不同的数组 $a$。

数据范围：$3 \leq n \leq 10^6$。

时空限制：$3$s / $256$MiB

{% endnote %}

<!-- more -->

# Solution

对于一种可能的局面，我们将已经被删除的元素标记成 $1$，没有被删除的元素标记成 $0$。那么有一个简单的必要条件：
- $1$ 的个数是 $2k$ 的倍数。
- 至少存在一个 $0$，使得其左边和右边各自至少有 $k$ 个 $1$。

然后发现这也是充分的：**先找到一个 $0$ 作为操作中心，满足前后各自至少有 $k$ 个 $1$**。将其视为最后一次操作，那么考虑退回这次操作。先将这个 $0$ 前后的 $k - 1$ 个 $1$ 还原成 $0$，此时不妨设左边 $1$ 的数量大于右边 $1$ 的数量，将右边第 $k$ 个 $1$ 还原成 $0$，此时全局应有奇数个 $1$，**选取最中间的那个 $1$ 还原成 $0$**（这个 $1$ 位于左侧，符合要求），然后将这个 $0$ 作为倒二次的操作中心，依此类推。

考虑计数，假设选的参数为 $k$，一共删了 $x$ 个数（$x$ 为 $k$ 的倍数）。正难则反，考虑使用总方案数减去不合法的方案数。

**可以先将这 $x$ 个 $1$ 摆好，然后将剩下的 $0$ 插入进去**。要想使得其不合法，$0$ 只能插在前 $k$ 个 $1$ 的左侧或后 $k$ 个 $1$ 的右侧，插在中间的空隙是不行的。将 $n - x$ 个 $0$ 分成 $2k$ 个组是一个简单的插板法，于是贡献为

$$
\binom{n}{x} - \binom{n - x + 2k - 1}{2k - 1}
$$

时间复杂度 $\mathcal{O}(n \log n)$，瓶颈在于调和级数式枚举。

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

const int mod = 998244353;

inline void add(int &x, const int &y) {
    x += y; if (x >= mod) x -= mod;
}
inline void dec(int &x, const int &y) {
    x -= y; if (x < 0) x += mod;
}

template <class T>
constexpr int qpow(int a, T b, int p) {
    int ans = 1;
    for (; b; b >>= 1) {
        if (b & 1) ans = 1ll * ans * a % p;
        a = 1ll * a * a % p;
    }
    return ans;
}

const int N = 1000100;

int n;

struct BinomCoef {
    std::vector<int> fact, facv;

    BinomCoef() {}
    BinomCoef(int n) {
        init(n);
    }

    void init(const int &n) {
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
};

void work() {
    std::cin >> n;

    BinomCoef bc(n * 2);

    for (int k = 1; k <= (n - 1) / 2; k ++) {
        int ans = 1;
        for (int x = 2 * k; x < n; x += 2 * k) {
            add(ans, bc.binom(n, x));
            dec(ans, bc.binom(n - x + 2 * k - 1, 2 * k - 1));
        }
        std::cout << ans << ' ';
    }
    std::cout << '\n';
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