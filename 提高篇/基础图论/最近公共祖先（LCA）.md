# 最近公共祖先（LCA）

## 倍增法（在线）

求LCA的话，我们可以先划好每个节点的高度，然后每次求的时候都暴力跳即可，不过这样的复杂度是 $O(n)$ 的，不是很能接受。好在我们可以使用倍增来加速这一流程。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 500010;
vector<int> tree[N];
int n, m, s;
int depth[N], lg[N], fa[N][23];
void dfs(int x, int f);
int lca(int x, int y);
int main()
{
	//read & build
	cin >> n >> m >> s;
	for (int i = 1; i < n; i++) {
		int x, y;
		cin >> x >> y;
		tree[x].push_back(y), tree[y].push_back(x);
	}
	//init
	dfs(s, 0);
	lg[1] = 0;
	for (int i = 2; i <= n; i++)
		lg[i] = lg[i / 2] + 1;
	//query
	while (m--) {
		int x, y;
		cin >> x >> y;
		printf("%d\n", lca(x, y));
	}
	return 0;
}
void dfs(int x, int f) {
	depth[x] = depth[f] + 1;
	fa[x][0] = f;
	for (int i = 1; (1 << i) <= depth[x]; i++)
		fa[x][i] = fa[fa[x][i - 1]][i - 1];
	for (int y : tree[x])
		if (y != f) dfs(y, x);
}
int lca(int x, int y) {
	if (depth[x] < depth[y]) swap(x, y);
	while (depth[x] > depth[y])
		x = fa[x][lg[depth[x] - depth[y]]];
	if (x == y) return x;
	for (int k = lg[depth[x]]; k >= 0; k--)
		if (fa[x][k] != fa[y][k])
			x = fa[x][k], y = fa[y][k];
	return fa[x][0];
}
```



## Tarjan法（离线）

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 500010;
int n, m, root;
vector<int> G[N];
//UnionSet
int fa[N];
void init() { for (int i = 1; i <= n; ++i) fa[i] = i; }
int find(int x) {
	if (x != fa[x]) fa[x] = find(fa[x]);
	return fa[x];
}
void merge(int x, int y) {
	x = find(x), y = find(y);
	if (x != y) fa[x] = y;
}
//Tarjan
struct Query {int x, y, id; };
vector<Query> query;
vector<int> Q[N];
int vis[N], ans[N];
void Tarjan(int x, int f) {
    vis[x] = 1;
	for (int y : G[x])
		if (y != f) Tarjan(y, x), merge(y, x);
	for (int id : Q[x]) {
		Query q = query[id];
		int y = x ^ q.x ^ q.y;
		if (vis[y]) ans[q.id] = find(y);
	}
}
//
int main()
{
	scanf("%d%d%d", &n, &m, &root);
	for (int i = 1; i < n; ++i) {
		int u, v;
		scanf("%d%d", &u, &v);
		G[u].push_back(v), G[v].push_back(u);
	}
	query.push_back((Query){0, 0, 0});
	for (int i = 1; i <= m; ++i) {
		int x, y;
		scanf("%d%d", &x, &y);
		query.push_back((Query){x, y, i});
		Q[x].push_back(i), Q[y].push_back(i);
	}
	init();
	Tarjan(root, 0);
	for (int i = 1; i <= m; ++i) printf("%d\n", ans[i]);
	return 0;
}
```

