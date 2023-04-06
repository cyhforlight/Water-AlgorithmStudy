# DFS序和相关衍生问题



## DFS序基础

简言之，我们对一个树进行如下的遍历：

```cpp
vector<int> e[N];
int tot, id[N], R[N];
int Val[N];
void dfs(int x, int fa) {
    id[x] = ++tot;
    Val[id[x]] = v[x];
    for (int y : e[x]) if (y != fa) dfs(y);
    R[id[x]] = tot;
}
```

通过这样的操作，我们可以将一个树上的每一个节点都映射到一个序列上面的一个位置。（听起来没啥意思，好像）

不过，DFS序有一个奇妙的性质：一个（子）树所有节点的编号在序列上是连续的。例如经过上面的DFS，对于以节点 x 作为根节点的子树，那么这整个子树在序列上面的区间表示就是 $id[x],R[id[x]]$。

通过这样的操作，我们就可以将树上问题（对子树的增删改查）转变为区间问题（这点可以参考树形背包），例如对某个子树的所有节点加上某个值，就可以转变为区间加，查询某个子树的所有节点的值就变成了区间求和。

我们来看一个例题（[牛客多校9 E Eyjafjalla](https://ac.nowcoder.com/acm/contest/11260/E)）:

> 一个国家的城市分布与连接可以用一个树形结构（有 $n$ 个点的无向树，无边权）来表示，首都是节点 1。
>
> 每个城市都有一个温度 $t_i$ ，如果两个城市 $u,v$ 有边相连，而且 $u$ 到首都的距离比 $v$ 近，那么有 $t_u>t_v$（那显然首都温度最高了)。
>
> 现在，有一个病毒爆发了，它可能在某个城市爆发，每个爆发的城市都会向周围扩散。不过，病毒有它的适宜温度区间 $[l,r]$，它只能够在这些城市生存。
>
> 现在给定一个国家的树结构，随后有 $m$ 次询问，每次询问都会给出一个病毒，它在城市 $x$ 爆发，适宜区间为 $[l,r]$，问这个病毒最多能扩散到多少个城市。（如果初始位置的温度就不适宜，那么这个病毒直接GG，感染城市为0)。
>
> $1\leq n,m\leq 10^5,1\leq t_i,l,r\leq 10^9$

题目里面特意说明了有边相连的城市才满足温度性质，说明并不是纯粹的越远温度越低，而是说只有树上的一条链才满足这性质。所有我们不妨对树从首都开始进行一次DFS，确定下父子关系和深度。

一个城市，无非向它的父节点和它的子节点传染，向子节点传染的时候只要考虑它们的温度是不是太低了，向父节点传染的时候只要考虑其温度是不是太高了。不过父节点向下传的时候还要传给自己，所以一个节点不如先拼命向上传，传不动了再向下传。

向上传的话如果一个个向上跳就太慢了，要用倍增优化一下。

向下传的话就转变成了一个新问题：一个子树有多少个节点的值小于 $v$？

因为不用修改，只要查询，所以我们可以暴力 DSU on tree 来做。

我们都讲了DFS序了，那让我们来看看用这个怎么做：我们DFS一遍，这样就变成了新问题：一段区间里面有多少个值小于 $v$。待修改的话我们可以主席树，不带修改的话就直接归并树了（有点类似归并排序+线段树）

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 100010;
int lg[N];
//
int n, t[N], dep[N], fa[N][25];
vector<int> e[N];
//DFS和倍增
int tot = 0, id[N], RR[N], Val[N];
void dfs(int x, int f, int depth) {
    dep[x] = depth;
    fa[x][0] = f;
    for (int i = 1; (1 << i) <= depth; ++i)
        fa[x][i] = fa[fa[x][i - 1]][i - 1];
    id[x] = ++tot;
    Val[id[x]] = t[x];
    for (int y : e[x])
        if (y != f) dfs(y, x, depth + 1);
    RR[id[x]] = tot;
}
//找最优父亲节点
int getFa(int x, int V) {
    if (t[x] > V) return -1;
    for (int k = lg[dep[x]]; k >= 0; k--) {
        int f = fa[x][k];
        if (f && t[f] <= V) x = f;
    }
    return x;
}
//区间查询：主席树
int Merge[25][N];
void build(int depth, int l, int r) {
    if (l == r) { Merge[depth][l] = Val[l]; return; }
    int mid = (l + r) >> 1;
    build(depth + 1, l, mid);
    build(depth + 1, mid + 1, r);
    //归并
    for(int i = l,j = mid + 1, k = l; i <= mid || j <= r; ){
        if(j > r) Merge[depth][k++] = Merge[depth + 1][i++];
        else if(i > mid || Merge[depth + 1][i] > Merge[depth + 1][j])
            Merge[depth][k++] = Merge[depth + 1][j++];
        else Merge[depth][k++] = Merge[depth + 1][i++];
    }
}

int Query(int l, int r, int V, int L = 1, int R = n, int depth = 0) {
    if (L >= l && R <= r) {
        int res = lower_bound(Merge[depth] + L, Merge[depth] + R + 1, V) - Merge[depth] - L;
        return (R - L + 1) - res;
    }
    int mid = (L + R) >> 1, ans = 0;
    if (mid >= l) ans += Query(l, r, V, L, mid, depth + 1);
    if (mid <  r) ans += Query(l, r, V, mid + 1, R, depth + 1);
    return ans;
}
int main()
{
    //init
    lg[1] = 0;
    for (int i = 2; i < N; ++i)
        lg[i] = lg[i / 2] + 1;
    //read
    scanf("%d", &n);
    for (int i = 1; i < n; ++i) {
        int u, v;
        scanf("%d%d", &u, &v);
        e[u].push_back(v);
        e[v].push_back(u);
    }
    for (int i = 1; i <= n; ++i)
        scanf("%d", &t[i]);
    //build
    dfs(1, 0, 1);
    build(0, 1, n);
    //query
    int q;
    scanf("%d", &q);
    while (q--) {
        int x, l, r;
        scanf("%d%d%d", &x, &l, &r);
        if (t[x] < l || t[x] > r) {
            puts("0");
            continue;
        }
        int root = getFa(x, r);
        printf("%d\n", Query(id[root], RR[id[root]], l));
    }
    return 0;
}
```



## 从子树到链：树链剖分

对于一个 $n$ 个节点的树，我们需要进行如下操作（共 $m$ 次)：

1. 1 x y z：将链 (x,y) 上的每个点都加上 z
2. 2 x y：查询链 (x,y) 上的每个点的值之和

1. 3 x z：将以 x 为根节点的子树的每个点的值都加上 x
2. 4 x：求以 x 为根节点的子树的所有点的值之和

 $1\leq n,m\leq 10^5$

操作3，4都是上面讲过的，略。

这里的树链剖分特指轻重链剖分，只将一条树链剖分为重链和若干条轻链。看了下面的代码后就会知道，一条重链上面的节点在DFS序上是连续的。那么，我们只需要将一条树链剖分成若干个重链的组合，即可用树状数组或线段树来解决。

根据下面巧妙的构造方法，保证链的操作的复杂度被控制在 $O(\log^2n)$。

```cpp
//P3384 【模板】轻重链剖分
#include<bits/stdc++.h>
using namespace std;
#define LL long long
const int N = 100010;
LL mod;
//SegmentTree
struct node {
    int l, r, len;
    LL val, tag;
} a[N << 2];
inline int ls(int x) { return x << 1; }
inline int rs(int x) { return x << 1 | 1; }
inline void pushup(int x) {
    a[x].val = (a[ls(x)].val + a[rs(x)].val) % mod;
}
void build (int x, int l, int r, LL *arr) {
    a[x].l = l, a[x].r = r, a[x].len = r - l + 1;
    a[x].tag = 0;
    if (l == r) {
        a[x].val = arr[l] % mod;
        return;
    }
    int mid = (l + r) >> 1;
    build(ls(x), l, mid, arr);
    build(rs(x), mid + 1, r, arr);
    pushup(x);
}
inline void f(int x, LL k) {
    a[x].tag += k;
    a[x].val += a[x].len * k;
    a[x].val %= mod;
}
inline void pushdown(int x) {
    f(ls(x), a[x].tag);
    f(rs(x), a[x].tag);
    a[x].tag = 0;
}
void update(int x, int l, int r, LL val) {
    if (l <= a[x].l && a[x].r <= r) {
        f(x, val);
        return;
    }
    pushdown(x);
    int mid = (a[x].l + a[x].r) >> 1;
    if (l <= mid) update(ls(x), l, r, val);
    if (r >  mid) update(rs(x), l, r, val);
    pushup(x);
    return;
}
LL query(int x, int l, int r) {
    if (l <= a[x].l && a[x].r <= r)
        return a[x].val;
    pushdown(x);
    int mid = (a[x].l + a[x].r) >> 1;
    LL sum = 0;
    if (l <= mid) sum += query(ls(x), l, r);
    if (r >  mid) sum += query(rs(x), l, r);
    return sum % mod;
}
//树链剖分
int n, root;
LL w[N];
vector<int> Tree[N];
int fa[N], depth[N], Size[N], Son[N];
void dfs1(int x, int f) {
    depth[x] = depth[f] + 1, fa[x] = f, Size[x] = 1;
    for (int y : Tree[x]) {
        if (y == f) continue;
        dfs1(y, x);
        Size[x] += Size[y];
        if (Size[y] > Size[Son[x]]) Son[x] = y;
    }
}
int cnt, Top[N], id[N];
LL wt[N];
void dfs2(int x, int topf) {
    id[x] = ++cnt, wt[cnt] = w[x];
    Top[x] = topf;
    if (Son[x]) dfs2(Son[x], topf);
    for (int y : Tree[x])
        if (y != fa[x] && y != Son[x]) dfs2(y, y);
}
//
void Change1(int x, int y, LL val) {
    val %= mod;
    while (Top[x] != Top[y]) {
        if (depth[Top[x]] < depth[Top[y]]) swap(x, y);
        update(1, id[Top[x]], id[x], val);
        x = fa[Top[x]];
    }
    if (depth[x] < depth[y]) swap(x, y);
    update(1, id[y], id[x], val);
}
LL Query1(int x, int y) {
    LL sum = 0;
    while (Top[x] != Top[y]) {
        if (depth[Top[x]] < depth[Top[y]]) swap(x, y);
        sum += query(1, id[Top[x]], id[x]), sum %= mod;
        x = fa[Top[x]];
    }
    if (depth[x] < depth[y]) swap(x, y);
    sum += query(1, id[y], id[x]), sum %= mod;
    return sum;
}
//子树查询，简单的一
void Change2(int x, int val) {
    update(1, id[x], id[x] + Size[x] - 1, val);
}
LL Query2(int x) {
    return query(1, id[x], id[x] + Size[x] - 1);
}
int main()
{
    //input
    int T;
    scanf("%d%d%d%lld", &n, &T, &root, &mod);
    for (int i = 1; i <= n; ++i)
        scanf("%lld", &w[i]);
    for (int i = 1; i < n; ++i) {
        int u, v;
        scanf("%d%d", &u, &v);
        Tree[u].push_back(v);
        Tree[v].push_back(u);
    }
    //build
    dfs1(root, 0);
    dfs2(root, root);
    build(1, 1, n, wt);
    //query
    while (T--) {
        int opt, x, y;
        LL val, ans;
        scanf("%d", &opt);
        switch (opt) {
        case 1:
            scanf("%d%d%lld", &x, &y, &val);
            Change1(x, y, val);
            break;
        case 2:
            scanf("%d%d", &x, &y);
            printf("%lld\n", Query1(x, y));
            break;
        case 3:
            scanf("%d%lld", &x, &val);
            Change2(x, val);
            break;
        case 4:
            scanf("%d", &x);
            printf("%lld\n", Query2(x));
            break;
        }
    }
    return 0;
}
```