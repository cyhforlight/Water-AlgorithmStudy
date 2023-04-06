# ST表

## 倍增介绍

倍增法（英语：binary lifting），顾名思义就是翻倍。当我们在递推时状态空间很大，我们可以通过成倍增长的方式，只递推 $2^k$ 的值作为代表，通过二进制划分的性质，表明其可以推出任意值。

它能够使线性的处理转化为对数级的处理，大大地优化时间复杂度。其核心思想类似于二分，通过二进制性质以证明其正确性，在下面这个代码中得以体现：

```cpp
//在区间[l,l + 2^30 - 2] 上二分查找值
//常数比直接二分要大
int x = l - 1;
for (int k = 29; k >= 0; k--)
    if (check(x + (1 << k))) x += (1 << k);
```

倍增在序列上其实没二分常用，它更常用于不可随机访问的线性和树形数据结构上。

这个方法在很多算法中均有应用，其中最常用的是 RMQ 问题（序列上）和求 LCA（树上），我们接下来主要探讨它在数据结构（ST表上面的应用）。



## 基于倍增的ST表

基于倍增的ST表能够 $O(nlog n)$ 的进行预处理，然后每次 $O(1)$ 的查询区间，预处理和查询方程可见代码。

取对数不要用数学库的对数函数，那玩意复杂度不是常数，还是预处理比较好。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 100010;
int n, a[N];
//ST表
int ST[N][25], lg[N];
void init() {
    //log预处理
    lg[1] = 0;
    for (int i = 2; i <= n; i++)
        lg[i] = lg[i / 2] + 1;
    //ST表预处理
    for (int i = 1; i <= n; ++i)
        ST[i][0] = a[i];
    for (int k = 1; k <= 20; ++k)
        for (int i = 1; i + (1 << k) - 1 <= n; ++i)
            ST[i][k] = max(ST[i][k - 1], ST[i + (1 << (k - 1))][k - 1]);
}
//询问
inline int query(int l, int r) {
    int k = lg[r - l + 1];
    return max(ST[l][k], ST[r - (1 << k) + 1][k]);
}
```

## 基于根号的ST表

基于倍增的ST表有一些小问题（如果 $O(1)$ 查询，那么每次必然是从两个重合的区间中合并得到答案，而并不是所有要维护的信息都满足这一性质的），这时候，我们可以使用基于根号的 ST 表。

我们先开一个 $\text{ST\_small}[N][\sqrt{N}]$ 的数组，$\text{ST\_small}[i][j]$ 表示区间 $[i,i+j-1]$ 的区间信息。

我们再开一个 $\text{ST\_big}[N][\sqrt{N}]$ 的数组，$\text{ST\_big}[i][j]$ 表示区间 $[i,i+j\sqrt{N}-1]$ 的区间信息。

那么有 $\text{ST\_big}[i][1]=\text{ST\_small}[i][\sqrt{N}]$ 。

这种基于根号，一大一小的东西，有点类似北上广深算法（**B**aby **S**tep, **G**iant **S**tep）和光速幂。

对于任意一段查询（例如区间和，虽然我们正常情况下肯定不会用这个数据结构来维护），我们可以转化为
$$
k=\sqrt{n}\\len=r-l+1\\
\operatorname{sum}(l,r)=\text{ST\_big}[l][len/k]+\text{ST\_small}[l+\lfloor\frac{len}{k}\rfloor*k][len-\lfloor\frac{len}{k}\rfloor*k]
$$
这个代码丑的一比，但是确实是可以 $O(1)$ 查询的。

```cpp
//好像还没看到啥必须得用这个写的题，就不去写代码了
```

