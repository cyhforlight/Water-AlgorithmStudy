# 线性DP

## 最长上升子序列

### DP方程

> 给定一个数列 $\{a_n\}$，求出其中最长严格上升子序列的长度。
>
> $n\leq 10^4,0<a_i\leq 10^9$

我们记 $dp_i$ 为以  $a_i$ 为结尾的最长严格上升子序列的长度，有
$$
dp_i=\max\limits_{0\leq j < i,a_j<a_i}dp_j+1
$$


```cpp
const int N = 10010;
int n, a[N], dp[N];
int solve() {
    //dp
    for (int i = 1; i <= n; ++i)
        for (int j = 0; j < i; ++j)
            if (a[j] < a[i]) dp[i] = max(dp[i], dp[j] + 1);
    //return
    int ans = 0;
    for (int i = 1; i <= n; ++i)
        ans = max(ans, dp[i]);
    return ans;
}
```

最长上升，不增不降啥的都可以基于此修修补补。

### DP复杂度优化

#### 方法一：二分

我们开一个 $\{d_n\}$，记 $d_i$ 为最长上升子序列长度为 $i$ 的序列的结尾的值（**有多个的话则取最小的那个**），然后 $len$ 记为目前最长的子序列长度。初始状态下，$d_1=a_1,len=1$。

 每当我们考虑添加一个新元素 $x$ 时：

1. 当 $d_{len}<x$ 时，显然有 $d_{len+1}=x$，并更新 $len$

2. 当 $d_{len}\geq x$ 时，我们显然需要找到最大的 $t$，且 $d_t<x$

   数组 $\{d_n\}$ 有一个不太显然的单调性：当 $i<j$ 时，有 $d_i<d_j$。

   反证法：如果存在 $i<j,d_i\geq d_j$，那么对于这个长度为 $j$ 的最长上升子序列，显然第 $i$ 个元素是小于 $d_i$ 的，但是以它作为终点又可以构成一个最长上升子序列，这就与定义相矛盾了。

那么，我们显然可以在 $\{d_n\}$ 上二分答案，这样就能够将复杂度降到 $O(n\log n)$ 了。

```cpp
int n, a[N], d[N], len;
int solve() {
    d[0] = -1, d[1] = a[1], len = 1;
    for (int i = 2; i <= n; ++i) {
        if (a[i] > d[len]) {
            d[++len] = a[i];
            continue;
        }
        int l = -1, r = len - 1;
        while (l < r) {
            int mid = (l + r + 1) >> 1;
            if (a[i] > d[mid]) l = mid;
            else r = mid - 1;
        }
        d[l + 1] = min(d[l + 1], a[i]);
    }
    return len;
}
```

#### 方法二：树状数组

这种方法受限于值域，所以经常被用来求排列的 LIS。

我们假设现在处理到 $i$，那么我们要找到 $\max\limits_{j<i,a_j<a_i}dp_j$。我们开一个线段树，然后让 $a_j$ 为下标，$dp_j$ 为对应的值（的最大值），那么我们就是要求 $[1,a_i-1]$ 上的最大值。查完之后，我们还要将 $a_i$ 下标上面值改为 $dp_i$（如果原来这个位置上面的值没这个大）。

考虑到值需要进行前缀上的查询，这个甚至可以只跑一个树状数组。

```cpp
namespace BIT {
    int s[N];
    inline int lowbit(int x) { return x & (-x); }
    int ask(int x) {
        int res = 0;
        for (; x; x -= lowbit(x)) res = max(res, s[x]);
        return res;
    }
    void add(int x, int val) {
        for (; x <= n; x += lowbit(x)) s[x] = max(s[x], val);
    }
}
//
int ans = 0;
for (int i = 1; i <= n; ++i) {
    int x = BIT::ask(a[i] - 1) + 1;
    BIT::add(a[i], x);
    ans = max(ans, x);
}
cout << ans;
```



### Dilworth 定理

定理：偏序集能划分成的最少的全序集个数等于最大反链的元素个数。

在 OI 中的应用：例如我们需要求出一个数列中至少能分成多少个单调不增子序列，那么其个数等价于该数列的最长上升子序列的长度。

我们记 $(a,b)$ 分别为下标和值，那么就有偏序关系 $a_1<a_2,b_1\geq b_2$，既然需要求解最少的全序集个数，那么我们直接求最大反链（$a_1\leq a_2,b_1<b_2$ 的元素个数即可（也就是长度））。



## 最长公共子序列

我们记 $f_{i,j}$ 为字符串 $a$ 的前 $i$ 个字符和字符串 $b$ 的前 $j$ 个字符能匹配出的最长公共子序列的长度。

考虑其来源：

1. 从 $f_{i-1,j}$ 转移而来
2. 从 $f_{i,j-1}$ 转移而来
3. 当 $a_i=b_j$ 时，从 $f_{i-1,j-1}$ 转移而来

$$
f_{i,j}=\max(f_{i-1,j},f_{i,j-1},f_{i-1,j-1}+(a_i=b_j))
$$

我们转化一下思路，也可以变为求字符串的公共子序列的长度：
$$
f_{i,j}=f_{i-1,j}+f_{i,j-1}-f_{i-1,j-1}+
\begin{cases}
f_{i-1,j-1}+1&a_i=b_j
\\
0&a_i\not= b_j
\end{cases}
$$

