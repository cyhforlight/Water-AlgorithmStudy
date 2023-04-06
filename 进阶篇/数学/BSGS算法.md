# BSGS算法（Baby-Step Giant-Step）

考虑如下方程（其中 $a,p$ 互质，且 $0\leq x<p$）：
$$
a^x\equiv b(\bmod p)
$$
BSGS算法，即大步小步算法，可以 $O(\sqrt{p})$ 的复杂度内解决该问题。

我们考虑令 $t=\lceil\sqrt{p}\rceil,x=At-B$，那么原同余式可化为
$$
a^{At-B}\equiv b(\bmod p)
\\
a^{At}\equiv b*a^B(\bmod p)
\\
(a^{t})^A\equiv b*a^B(\bmod p)
$$
因为 $0\leq B<t$，所以我们依次将 $b*a^0,b*a^1,\cdots,b*a^{t-1}$ 放入集合 $S$ 中（当然，要记得取模）。随后，我们从 $0$ 开始枚举 $A$，依次计算 $(a^t)^A$ 是否在集合内出现过，出现过的话即可得到对应的 $B$，从而算出最小的 $x$。

根据费马小定理，$\gcd(a,p)=1$ 时，有 $a^{p-1}\equiv 1(\bmod p)$，所以解会控制在 $[0,p-1)$ 内，故总复杂度 $O(\sqrt{p})$（如果集合是用哈希之类的数据结构存的话，用平衡树就多带一个 log）。

```cpp
//P3846 [TJOI2007] 可爱的质数/【模板】BSGS
#include<bits/stdc++.h>
using namespace std;
#define LL long long
LL power(LL a, LL b, LL p) {
    LL res = 1;
    while (b) {
        if (b & 1) res = res * a % p;
        a = a * a % p;
        b >>= 1;
    }
    return res;
}
LL BSGS(LL a, LL b, LL p) {
    if (b == 1) return 0;
    LL t = sqrt(p) + 1;
    map<LL, LL> vis;
    //因为x = At-B，所以尽量保留B大的
    for (int i = 0; i < t; ++i)
        vis[b * power(a, i, p) % p] = i;
    LL aa = power(a, t, p);
    for (int A = 1; A <= t; ++A) {
        LL x = power(aa, A, p);
        if (vis.find(x) != vis.end()) {
            LL B = vis[x];
            return A * t - B;
        }
    }
    //遍历了整个解空间都找不到解，返回-1
    return -1;
}
int main()
{
    LL p, a, b;
    cin >> p >> a >> b;
    LL ans = BSGS(a, b, p);
    if (ans == -1)
        cout << "no solution" << endl;
    else cout << ans << endl;
    return 0;
}
```

