# 字典树



## 一、普通字典树

> 给定 $n$ 个字符串，进行 $m$ 次点名，每次回答“第一次点到这个名字”，“之前点到过这个名字”，“没有这名字”。
>
> $1\leq n \leq 1000,1\leq m \leq 10^5,|s|\leq 50$

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 10010 * 50;
int n, m;
//Trie
int Trie[N][26], End[N], tot = 0;
void insert(char *s) {
    int len = strlen(s), p = 0;
    for (int i = 0; i < len; ++i) {
        int c = s[i] - 'a';
        if (!Trie[p][c]) Trie[p][c] = ++tot;
        p = Trie[p][c];
    }
    End[p] = 1;
}
int search(char *s) {
    int len = strlen(s), p = 0;
    for (int i = 0; i < len; ++i) {
        p = Trie[p][s[i] - 'a'];
        if (!p) return 0;
    }
    if (End[p] == 1) {
        End[p] = 2;
        return 1;
    }
    return End[p];
}
int main()
{
    char s[50];
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i) {
        scanf("%s", s);
        insert(s);
    }
    scanf("%d", &m);
    while (m--) {
        scanf("%s", s);
        int flag = search(s);
        if (flag == 1) puts("OK");
        else if (flag == 2) puts("REPEAT");
        else puts("WRONG");
    }
    return 0;
}
```

## 二、01 Trie树

> 给你一棵带边权的 $n$ 个点的树，第 $i$ 条边的边权为 $w_i$，求点对  $(u,v)$ 使得 $u$ 到 $v$ 的路径上的边权异或和最大，输出这个最大值。
>
> $2 \leq n \leq 10^5,0\leq w_i < 2^{31}$

我们随便指定一个点作为 $root$ ，记点 $x$ 到 $root$ 上的边权异或和为 $f(x)$ ，那么 $(u,v)$ 路径上的边权就是 $f(u) \operatorname{xor}f(v)$，因为 `LCA` 上面的相同路程被全部亦或掉了。

我们先遍历一遍，求出所有的点的函数值，加入一个数组中，问题就转化成了：从数组中选取两个数，使得其异或和最大。

基于二进制数的性质，我们有一个贪心的选法：将他们全部转化为二进制数，然后每次都尽力反着选，使得高位尽可能为1。

不难发现，我们可以建一个类似 Trie 树的东西，因为只有 0 和 1，所以又被称作 **01 Trie树**。

这个数据结构被广泛应用于需要进行异或的场合，需要熟练掌握。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 100010;
int n;
//Tree
namespace Tree {
    const int M = N << 1;
    int tot = 0;
    int ver[M], edge[M];
    int Head[N], Next[M];
    void addEdge(int u, int v, int w) {
        ver[++tot] = v, edge[tot] = w;
        Next[tot] = Head[u], Head[u] = tot;
    }
    void build_Tree() {
        int u, v, w;
        for (int i = 1; i < n; ++i) {
            scanf("%d%d%d", &u, &v, &w);
            addEdge(u, v, w);
            addEdge(v, u, w);
        }
    }
    void dfs(int x, int fa, int *arr) {
        for (int i = Head[x]; i; i = Next[i]) {
            int to = ver[i];
            if (to == fa) continue;
            arr[to] = arr[x] ^ edge[i];
            dfs(to, x, arr);
        }
    }
};
namespace Trie01 {
    int a[N];
    int Trie[N << 5][2], tot = 0;
    void insert(int x) {
        int p = 0;
        for (int i = 30; i >= 0; i--) {
            int c = (x >> i) & 1;
            if (!Trie[p][c]) Trie[p][c] = ++tot;
            p = Trie[p][c];
        }
    }
    void build01Trie() {
        for (int i = 1; i <= n; ++i) insert(a[i]);
    }
    int query(int x) {
        int p = 0, ans = 0;
        for (int i = 30; i >= 0; i--) {
            int c = (x >> i) & 1;
            if (Trie[p][c ^ 1]) p = Trie[p][c ^ 1], ans |= (1 << i);
            else p = Trie[p][c];
        }
        return ans;
    }
};
int main()
{
    scanf("%d", &n);
    //build tree
    Tree::build_Tree();
    Tree::dfs(1, 0, Trie01::a);
    //Trie
    Trie01::build01Trie();
    //我们选择了一种比较讨巧的方式：每次在Trie中寻找可以和x异或出的最大值
    //这种方法的复杂度是O(31 * n) 的，但是不会影响题目的总复杂度
    //理论上可以直接在构造好的Trie树上面取，但我懒得写了（逃
    int ans = 0;
    for (int i = 1; i <= n; ++i)
        ans = max(ans, Trie01::query(Trie01::a[i]));
    printf("%d", ans);
    return 0;
}
```