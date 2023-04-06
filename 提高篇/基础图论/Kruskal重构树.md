# Kruskal重构树

简单来说，就是 Kruskal 的时候，每次要合并两个连通块的时候，都要把他们所属的连通块的最高节点向一个新节点连边（这个点的点权就是边权），最后得到一个 $2n-1$ 大小的树。

随后，我们想要查询图上两点之间的路径，使得路径的最大值最小时，我们直接查询这两个点在树上的 LCA 的点权即可。

下面是15年货车运输的代码：

```cpp
#include<bits/stdc++.h>
using namespace std;
//原图的规模就是10^4，所以我直接把N写成20010了，省的下面老要乘2啥的
const int N = 20010;
int n, m, q;
struct Edge { int x, y, z; };
vector<Edge> edges;
//UnionSet
int ufa[N], val[N];
int find(int x) {
    if (x != ufa[x]) ufa[x] = find(ufa[x]);
    return ufa[x];
}
//Tree
vector<int> G[N << 1];
set<int> roots; //需要考虑一下多个树的情况
//LCA
int dep[N], lg[N], fa[N][20];
void dfs(int x, int f) {
	dep[x] = dep[f] + 1;
	fa[x][0] = f;
	for (int i = 1; (1 << i) <= dep[x]; i++)
		fa[x][i] = fa[fa[x][i - 1]][i - 1];
	for (int y : G[x])
		if (y != f) dfs(y, x);
}
int lca(int x, int y) {
	if (dep[x] < dep[y]) swap(x, y);
	while (dep[x] > dep[y])
		x = fa[x][lg[dep[x] - dep[y]]];
	if (x == y) return x;
	for (int k = lg[dep[x]]; k >= 0; k--)
		if (fa[x][k] != fa[y][k])
			x = fa[x][k], y = fa[y][k];
	return fa[x][0];
}
int main() {
    //read
    cin >> n >> m;
    for (int i = 0; i < m; ++i) {
        int x, y, z;
        cin >> x >> y >> z;
        edges.push_back((Edge){x, y, z});
    }
    //Kruskal
    sort(edges.begin(), edges.end(), [](const Edge A, const Edge B) {
        return A.z > B.z;
    });
    for (int i = 1; i <= n * 2; ++i) ufa[i] = i;
    for (int i = 1; i <= n; ++i) roots.insert(i);
    int tot = n;
    for (Edge e : edges) {
        int x = find(e.x), y = find(e.y);
        if (x == y) continue;
        val[++tot] = e.z, ufa[x] = ufa[y] = tot;
        G[tot].push_back(x), G[tot].push_back(y);
        roots.erase(x), roots.erase(y), roots.insert(tot);
        if (tot == 2 * n - 1) break;
    }
    //buildLCA
    for (int root : roots) dfs(root, 0);
    lg[1] = 0;
    for (int i = 2; i < N; ++i)
        lg[i] = lg[i / 2] + 1;
    //query
    cin >> q;
    while (q--) {
        int x, y;
        cin >> x >> y;
        if (find(x) != find(y)) puts("-1");
        else printf("%d\n", val[lca(x, y)]);
    }
    return 0;
}
```

