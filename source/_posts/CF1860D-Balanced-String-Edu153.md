---
title: CF1860D Balanced String
toc: true
permalink: /posts/CF1860D
date: 2023-08-19 00:52:38
categories: DP
tags: 线性DP
---

## 题意

[原题链接](https://codeforces.com/problemset/problem/1860/D)

对于 $01$ 串，找到最少的交换次数，使得

- 子序列 $01$ 的数量等于子序列 $10$ 的数量

$3 \le |s| \le 100$

<!--more-->

## 分析

长度为2的子序列一共4种情况，分别为 $00,01,10,11$。

- 在进行交换时，无论如何 $00$ 和 $11$ 的子序列个数是不变的，同时四种子序列的总数也不变。
- 则 $10$ 和 $01$ 的子序列总和不变，当我们让 $10$ 和 $01$ 的数量都为它们总和的一半时，才会相等。

考虑每一位 `s[i] = 1` 对子序列的贡献，若当前位为1的话， 那么 $01$ 和 $11$ 的子序列个数应该都增加 $i$ ，则 $01$ 和 $11$ 的子序列个数可求

为了方便表示，记：

-  $c_0$ 为 $0$ 的个数
-  $c_1$ 为 $1$ 的个数
- $sum(n)$ 为 n 的子序列个数-> $sum(n) = \dfrac{n(n-1)}{2}$
- $need$ 为最后所需要的 $01$ 或 $10$ 的数量，则有如下推导

$$
sum(n) = sum(00)+sum(11)+sum(01)+sum(10)
$$


$$
sum(n) = sum(00)+sum(11)+2\cdot need
$$

$$
need = \dfrac{sum(n)-sum(00)-sum(11)}{2}
$$


但是我们对于每个位置是统计 $sum(01) + sum(11)$ ，故需要一步转化
$$
sum(01)+sum(11) =\dfrac{sum(n)-sum(00)-sum(11)}{2}+sum(11)
$$

$$
k = \dfrac{sum(n)-sum(00)+sum(11)}{2}, k=sum(01)+sum(11)
$$

此时我们就将原来的need转换为k了：既当 $sum(01)+sum(11)=k$时，$sum(01)=need$

### 状态表示与状态计算

令 $f[i][j][k]$ 表示前 $i$ 个位置有 $j$ 个 $1$ 且此时的 $01$ 和 $11$ 的子序列总和为 $k$

转移方程：

- 若当前 $i$ 不放 $1$：$f[i][j][k]=f[i-1][j][k]$
- 若当前 $i$ 放 $1$ 且原序列中此处为 $1$：$f[i][j][k]=f[i-1][j-1][k-i]$
- 若当前 $i$ 放 $1$ 且原序列中此处**不为** $1$：$f[i][j][k]=f[i-1][j-1][k-i]+1$（swap)

类似与 01背包问题，这里我们也可以优化一维

边界：$f[0][0] = 0$

答案：$f[c1][need]$

## 代码

```c++
const int N = 110, M = N * (N - 1) / 2;

int f[N][M];

void solve()
{
    string s;
    cin >> s;
    int n = s.size();
    int c1 = count(s.begin(), s.end(), '1'), c0 = n - c1;
    int need = (n * (n - 1) / 2 - c0 * (c0 - 1) / 2 + c1 * (c1 - 1) / 2) / 2;
    memset(f, 0x3f, sizeof f);
    f[0][0] = 0;
    for (int i = 0; i < n; i++) {
        for (int j = c1; j >= 1; j--) {
            for (int k = i; k <= need; k++) {
                f[j][k] = min(f[j][k], f[j - 1][k - i] + (s[i] == '0'));
            }
        }
    }
    cout << f[c1][need] << endl;
}
```

