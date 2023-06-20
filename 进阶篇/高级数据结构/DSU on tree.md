# 树上启发式合并（DSU on tree）

55555555

6666666

树上启发式合并能够比较高效的查询子树上的各类查询问题（不带修），且框架复杂度要低于树上莫队（复杂度为 $O(n\log n)$）。

> 题目来源：[DongDong数颜色](https://ac.nowcoder.com/acm/contest/904/E)
>
> 给定一个 $n$ 个节点的树（根节点为 1），第 $i$ 个节点的颜色为 $c_i$。
>
> 接下来有 $m$ 次询问，对于节点 $u$，我们都会要求输出以 $u$ 为根节点的子树中的颜色数量。
>
> $1\leq n,m\leq 10^5$

我们可以暴力统计，总复杂度为 $O(nm)$。如果用树上莫队，那复杂度可以降到 $O(n\sqrt{n})$ 规模。

我们现在尝试以暴力为基础，直接预处理出所有答案，随后每次 $O(1)$ 查询。发现，对于每个节点 $u$，其答案由其本身和其若干个子树决定，暴力的操作流程如下：

1. 遍历子树 1，统计答案
2. 清空 cnt 数组，遍历子树 2，统计答案
3. ...
4. 清空 cnt 数组，遍历子树 k，统计答案
5. 遍历所有子树，然后加上节点 $u$，统计答案

我们采取类似轻重链剖分的步骤，先搜索出所有子树的大小，并找出所有的重儿子，然后 DFS 的时候，先处理轻儿子再处理重儿子，重儿子处理完之后不清空 cnt 数组，直接加上别的轻儿子代表的子树，然后得到本子树的答案。

复杂度的严格证明可以参照 OI Wiki，我们知道它预处理 $O(n\log n)$，单次询问复杂度 $O(1)$ 即可。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 100010;
int n, m, col[N];
vector<int> G[N];
//
int Size[N], bigSun[N];
int dfn, L[N], R[N], mp[N];
void dfs1(int x, int fa) {
    L[x] = ++dfn, mp[dfn] = x, Size[x] = 1;
    for (int y : G[x])
        if (y != fa) {
            dfs1(y, x);
            Size[x] += Size[y];
            //初始状态下bigSun_x=0,所以一定能更新
            if (Size[bigSun[x]] < Size[y]) bigSun[x] = y;
        }
    R[x] = dfn;
}
//
int tot, cnt[N], ans[N];
void add(int x) { if (++cnt[col[x]] == 1) ++tot; }
void del(int x) { if (--cnt[col[x]] == 0) --tot; }
void dfs2(int x, int fa, int keep) {
    //
    for (int y : G[x])
        if (y != fa && y != bigSun[x])
            dfs2(y, x, false);
    if (bigSun[x]) dfs2(bigSun[x], x, true);
    //
    for (int y : G[x])
        if (y != fa && y != bigSun[x])
            for (int i = L[y]; i <= R[y]; ++i) add(mp[i]);
    add(x);
    ans[x] = tot;
    //
    if (!keep) for (int i = L[x]; i <= R[x]; ++i) del(mp[i]);
}
int main()
{
    //read
    cin >> n >> m;
    for (int i = 1; i <= n; ++i)
        cin >> col[i];
    for (int i = 1; i < n; ++i) {
        int u, v;
        cin >> u >> v;
        G[u].push_back(v);
        G[v].push_back(u);
    }
    //init
    dfs1(1, 0);
    dfs2(1, 0, false);
    //query
    while (m--) {
        int x;
        cin >> x;
        cout << ans[x] << endl;
    }
    return 0;
}
```



> 对上面的题目进行一手修改：每次查询不仅给定 $x$，还要额外给定一个 $k$，查询子树 $x$ 中出现次数达到 $k$ 的颜色。

上一题给了我们一些错觉，实际上 DSU on tree 本质上还是一个离线算法，所以我们对询问存储一下。

```cpp
struct Node {int v, id; };
vector<Node> Query[N];
//query
for (int i = 1; i <= m; ++i) {
    int x, k;
    cin >> x >> k;
    Query[x].push_back((Node){k, i});
}
```

我们额外开一个 $sum$ 数组，表明达到某次数的颜色的数量，然后修改 add 和 del：

```cpp
int cnt[N], sum[N], ans[N];
void add(int x) {
    ++cnt[col[x]];
    ++sum[cnt[col[x]]];
}
void del(int x) {
    --sum[cnt[col[x]]];
    --cnt[col[x]];
}
```

然后 dfs2 中修改即可：

```cpp
for (Node q : Query[x]) ans[q.id] = sum[q.v];
```

