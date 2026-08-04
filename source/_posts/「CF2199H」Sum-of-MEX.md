---
title: 「CF2199H」Sum of MEX
date: 2026-08-03 16:42:50
updated: 2026-08-03 16:42:50
categories: Codeforces
tags:
  - DP
---

# Description

Link：[CF2199H](https://codeforces.com/contest/2199/problem/H)

{% note default %}

给出一个数组 $a_1, \dots, a_n$，其中 $-1 \leq a_i \leq n$。

对于每个 $i$ ($1 \leq i \leq n$)，求出 $f([a_1, \dots, a_i])$，其中 $f(s)$ 对于数组 $s$ 定义如下：
- 对于所有将 $s$ 中的 $-1$ 替换成 $[0, n]$ 之间整数的不同方案，$f(s)$ 为这些方案对应数组的 MEX 之和。

保证 $-1$ 的数量不超过 $300$。

数据范围：$1 \leq n \leq 2\times 10^5$，$-1 \leq a_i \leq n$。

时空限制：$5$s / $512$MiB。

{% endnote %}

<!-- more -->

# Solution

乍一看有点像 [「2026 杭电多校 2」1005. 减数游戏](https://acm.hdu.edu.cn/contest/problem?cid=1230&pid=1005)，但是本题要对每个前缀都求一次答案，不方便容斥。

首先将 $\mathrm{mex}(s)$ 转换成 $\sum_{i = 0}^{n - 1} [\mathrm{mex}(s) > i]$，此时 $\mathrm{mex}(s) > i$ 相当于 $0, \dots, i$ 在 $s$ 中均出现过。

**本题的 $-1$ 个数不超过 $300$，所以每个替换方案至多只能填补 $300$ 个空位**。

考虑先预处理一个 dp 数组 $f(i, j)$ 表示：有多少个长度为 $i$ 值域在 $[0, n]$ 且数值 $0, \dots, j - 1$ 均出现过的数组。**我们将必须出现的数值 $0, \dots, j - 1$ 称为关键数字，这 $j$ 个关键数字具体的数值其实我们并不关心，因为这 $n + 1$ 个数字的地位是等价的**。

初态：$f(i, 0) = (n + 1)^i$。

转移方程：分最后一位是否填关键数字，有两种转移

$$
f(i, j) = f(i - 1, j)\times (n + 1 - j) + f(i - 1, j - 1)\times j
$$

这里的第二种转移是“最后一位填关键数字”，那么其他的 $j - 1$ 个关键数字必须出现，由于关键数字的具体取值不影响计数，这和 $f(i - 1, j - 1)$ 的定义是一样的（双射）。

回到统计答案。对于当前前缀，设其中有 $b$ 个 $-1$，设从小到大第 $i$ 个空位的数值为 $x_i$。**那么一旦填上前 $i$ 个空位，可以使得所有 $x_i \leq v < x_{i + 1}$ 都有 $\mathrm{mex}(s) > v$**。则当前前缀的答案为

$$
x_1\times f(b, 0) + \sum_{j = 1}^b (x_{j + 1} - x_j)\times f(b, j)
$$

设 $W$ 表示 $-1$ 的个数，时间复杂度 $\mathcal{O}(nW + W^2)$。

（因为本题只能提交 Kotlin，就叫 ChatGPT 帮我仿写了一份 Kotlin 2.2.0 的代码）

```Kotlin
import java.io.BufferedInputStream

typealias i64 = Long

const val mod = 998244353

fun qpow(a0: Int, b0: Int, p: Int): Int {
    var a = a0
    var b = b0
    var ans = 1

    while (b != 0) {
        if (b and 1 != 0) {
            ans = (1L * ans * a % p).toInt()
        }
        a = (1L * a * a % p).toInt()
        b = b shr 1
    }

    return ans
}

const val W = 301
val f = Array(W + 1) { IntArray(W + 1) }

const val N = 200100

var n = 0
val a = IntArray(N)

val fa = IntArray(N)

fun get(x0: Int): Int {
    var x = x0
    var rt = x

    while (fa[rt] != rt) {
        rt = fa[rt]
    }

    while (fa[x] != x) {
        val y = fa[x]
        fa[x] = rt
        x = y
    }

    return rt
}

var b = 0

fun ask(): Int {
    var x = get(0)
    var ans = (1L * x * f[b][0] % mod).toInt()

    var i = 1
    x = 0
    var y: Int

    while (i <= b) {
        x = get(x)
        y = get(x + 1)

        ans = (
            ans + 1L * (y - x) * f[b][i] % mod
        ).toInt()

        if (ans >= mod) {
            ans -= mod
        }

        i++
        x++
    }

    return ans
}

fun main() {
    val input = FastScanner()

    n = input.nextInt()

    for (i in 1..n) {
        a[i] = input.nextInt()
    }

    for (i in 0..n + 1) {
        fa[i] = i
    }

    val w = minOf(n, W)

    f[0][0] = 1

    for (i in 1..w) {
        f[i][0] = qpow(n + 1, i, mod)

        for (j in 1..w) {
            f[i][j] = (
                (
                    1L * (n + 1 - j) * f[i - 1][j] +
                    1L * j * f[i - 1][j - 1]
                ) % mod
            ).toInt()
        }
    }

    val output = StringBuilder()

    for (i in 1..n) {
        if (a[i] == -1) {
            b++
        } else {
            if (fa[a[i]] == a[i]) {
                fa[a[i]] = a[i] + 1
            }
        }

        output.append(ask())
        output.append(if (i == n) '\n' else ' ')
    }

    print(output)
}

class FastScanner {
    private val input = BufferedInputStream(System.`in`)
    private val buffer = ByteArray(1 shl 16)

    private var len = 0
    private var ptr = 0

    private fun read(): Int {
        if (ptr >= len) {
            len = input.read(buffer)
            ptr = 0

            if (len <= 0) {
                return -1
            }
        }

        return buffer[ptr++].toInt()
    }

    fun nextInt(): Int {
        var c = read()

        while (c <= 32 && c != -1) {
            c = read()
        }

        var sign = 1

        if (c == '-'.code) {
            sign = -1
            c = read()
        }

        var x = 0

        while (c > 32 && c != -1) {
            x = x * 10 + c - '0'.code
            c = read()
        }

        return x * sign
    }
}
```