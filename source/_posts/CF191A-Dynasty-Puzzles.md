---
title: CF191A Dynasty Puzzles
toc: true
permalink: /posts/CodeForces-191A
date: 2023-08-13 00:41:37
categories: DP
tags: 线性DP
---

## 题意

按时间顺序给了 $n(1\le n \le 5\cdot 10^5)$ 个国王的名字，找这样的国王排列的国王个数最多有多少个：

1. 国王的名字必须是按照时间的先后顺序排列
2. 前一个国王的名字的尾字母必须和后一个国王的名字首字母相同
3. 第一个国王的首字母必须和最后一个国王的尾字母相同

<!--more-->

## 分析

**状态表示：**

- **集合：**$f(i,j)$ 表示所有以字符 `i` 开始， 以字符 `j` 结束，符合上述条件的字符串
- **属性：**长度的最大值

**状态计算：**
$$
f(i,j)=\max \left( \sum_{k=a}^{z}f(i, k) + f(k, j)\right)
$$
若此时输入了一个新的字符串 `S = a.....b`, 以a开头，b结尾。 

如果之前存在一个以 a 结尾的 $f(i, a)$ ，则 $f(i, b) = \max \left(f(i,b), f(i,a)+len \right)$ 

最后特判一下，若输入的字符串长度 len 比转移来的 $f(a,b)$ 还要长的话，直接更新其值

## 代码

```c++
#include <iostream>
#include <algorithm>
#include <cstring>
#include <string>
using namespace std;

const int N = 30;

int f[N][N];
int main() 
{
    int n;
    cin >> n;
    string s;
    while (n --)
    {
    	cin >> s;
    	int len = s.size();
    	int a = s[0]-'a'+1, b = s[len-1]-'a'+1;
    	for (int i = 1; i <= 26; i ++) {
    		if (f[i][a]) 
    			f[i][b] = max(f[i][b], f[i][a] + len);
    	}
    	if (f[a][b] < len) f[a][b] = len;
    }
    int ans = 0;
    for (int i = 1; i <= 26; i ++) {
    	ans = max(ans, f[i][i]);
    }
    cout << ans;
    return 0;
}
```

