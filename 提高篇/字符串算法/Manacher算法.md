# Manacher算法



```cpp
//P3805 【模板】manacher 算法
#include <bits/stdc++.h>
using namespace std;
const int N = 1.2e7;
char s[N], data[N << 1];
int p[N << 1];
int main()
{
    //read & init
    scanf("%s", s + 1);
    int n = strlen(s + 1);
    data[0] = '~', data[1] = '|';
    for (int i = 1; i <= n; ++i)
        data[2 * i] = s[i], data[2 * i + 1] = '|';
    data[2 * n + 2] = '|';
    //Manacher
    int r = 0, mid = 0;
    for (int i = 1; i <= 2 * n + 1; ++i) {
        p[i] = i < r ? min(p[(mid << 1) - i], r - i) : 1;
        while (data[i - p[i]] == data[i + p[i]]) ++p[i];
        if (p[i] + i > r) r = p[i] + i, mid = i;
    }
    int ans = 0;
    for (int i = 1; i <= 2 * n + 1; ++i)
        ans = max(ans, p[i]);
    printf("%d\n", ans - 1);
    return 0;
}
```



考虑到回文串有两种情况，所以我们在原来串里面每两个字符间都插入一个其他字符，这样就可以明确以某个点作为核心了。（注意 $data[0]$ 和 $data[2n+2]$ 这两个位置的特殊取值，而且他俩不可以相同）

随后维护 p 数组即可，算法完成后，$p_i$ 就表示以 $data[i]$ 作为中心，能够延伸的最长距离（再加1）？

