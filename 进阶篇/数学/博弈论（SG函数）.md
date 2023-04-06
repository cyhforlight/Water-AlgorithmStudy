# 博弈论（SG函数）



## SG函数

我们研究这样一种游戏，它满足以下三个性质：

1. 由 2 名玩家交替行动
2. 在游戏进程的任意时刻，可以执行的合法行动与这位玩家无关
3. 不能行动的玩家判负

那么这个游戏被称为**公平组合游戏**。当两名玩家均采取最优策略时，游戏的胜负只和游戏的初始情况有关，和玩家的选择无关（在采取最优策略的情况下）。

我们发现，这类游戏由于性质2，所以游戏似乎可以不记录玩家的状态，只记录游戏目前的情况，那么我们似乎可以将其视为一种类 DP 问题，将游戏的状态使用多维数组甚至状压来保存下来，随后通过不断转移来求解。实际上，它们有一个更为抽象的表达形式：有向图游戏。

给定一个有向无环图，图中有一个唯一的起点，在起点上有一个棋子。两名玩家交替的把这枚棋子沿有向边移动，无法移动者判负。显然的，所有公平组合游戏都可以转化为有向图游戏。

根据前面知识，我们推测图上的每个点都具有必胜或者必败两种状态，可以进行黑白染色。显然，如果一个点可前往的任何点都是必胜点（或者没有点可走），那么它就是一个必败点；相反，如果它可以前往任意一个必败点，那么他就是一个必胜点。

我们有一个更好的方法来表示一个点的必胜或者必败：我们假设 $S$ 是一个集合，那么定义 $\operatorname{mex}(S)$ 为求出不属于集合 $S$ 的最小非负整数的运算，也就是
$$
\operatorname{mex}(S)=\min_{x\in \N,x\notin S}\{x\}
$$
这时候，对于每个节点 $x$，我们记它的所有后继节点为 $y_1,y_2,\cdots,y_n$，那么有
$$
\operatorname{SG}(x)=\operatorname{mex}\{\operatorname{SG}(y_1),\operatorname{SG}(y_2),\cdots,\operatorname{SG}(y_n)\}
$$
特别的，整个有向图游戏的 SG 函数值为起点的 SG 函数值。

那么，我们不难得出定理：

1. 有向图的某个局面必胜，但且仅当对应节点的 SG 函数值大于 0
2. 有向图的某个局面必败，但且仅当对应节点的 SG 函数值为 0

## Nim游戏

> 给定 $n$ 堆石子，第 $i$ 堆石子有 $a_i$ 个石子。两名玩家轮流行动，每次可以任选一堆，从中拿走任意个石子（可以全部拿完，但不可以不拿）。取走最后的石子者获胜（也就是说无法再拿石子的人会输）。在两者均采取最优策略的情况下，问先手是否必胜。

如果使用朴素的转移方法，那么时空复杂度至少为 $O(\prod\limits_{i=1}^na_i)$，是比较难以承受的（还不太好写）。但庆幸的是，这题有一个很朴素的解法：

### 定理

Nim 博弈先手必胜，当且仅当
$$
\operatorname{f}= a_1 \operatorname{xor} a_2 \operatorname{xor} \cdots \operatorname{xor} a_n\not= 0
$$

### 证明

当 $a_i$ 均为 0 时，此时所有物品被取光，此时 $\operatorname{f}=0$ 。

对于某个局面， 倘若 $\operatorname{f}=x\not= 0$，我们记 $x$ 的二进制表示下 1 的最高位在第 k 位，那么至少存在一堆石子 $a_i$，使得其二进制下第 k 位为 1。显然 $a_i \operatorname{xor} x < a_i$ ，所以我们只需要将这对石子拿到 $a_i \operatorname{xor} x$ 个，那么就可以使得 $\operatorname{f} = x \operatorname{xor} a_i\operatorname{xor}a_i \operatorname{xor} x=0$。

相反，对于某个局面，如果$\operatorname{f}=0$，那么无论怎么取，新的 $\operatorname{f}$ 都一定不为 0（涉及亦或的相关知识，可以使用反证法证明）。

根据数学归纳法，我们发现当 $\operatorname{f}=0$ 时先手必败，因为无论怎么走都会使得下一个状态的 $\operatorname{f}$ 不为 0；反之，$\operatorname{f}\not= 0$ 时先手必胜，因为必然存在一种使下一个状态的 $\operatorname{f}$ 为 0 的状态。

### 与 SG 函数的关系

不难发现，$\operatorname{f}$ 就是这个游戏的 SG 函数。

## 多个有向图游戏

假设 $G_1,G_2,\cdots,G_n$ 是 $n$ 个有向图游戏，他们共同组成了一个大有向图游戏 $G$。两名玩家轮流进行，每次选一个子游戏来玩一把，当某个人无法进行任何子游戏时，他将输掉总游戏 $G$。

有向图游戏的和的 SG 函数值等于各个子游戏的 SG 函数值的异或和，也就是
$$
\operatorname{SG}(G)=\operatorname{SG}(G_1)\operatorname{xor}\operatorname{SG}(G_2)\operatorname{xor}\cdots\operatorname{xor}\operatorname{SG}(G_n)
$$
证明方法和 Nim 游戏的证明方式类似，不再赘述（其实我也一时不知道咋证明）。

### 用子游戏的性质来证明 Nim 游戏定理

我们发现，Nim 游戏就可以视作 $n$ 个子游戏：假设在每某子游戏中，有 $m$ 个石子，每次可以选取拿走任意数量的石子，那么我们不难发现，有
$$
\operatorname{SG}(m)=\operatorname{SG}(m-1)\operatorname{xor}\operatorname{SG}(m-2)\operatorname{xor}\cdots\operatorname{xor}\operatorname{SG}(0)
$$
通过逆推发现， $\operatorname{SG}(x)=x$，所以第 i 个游戏中，石子堆有 $a_i$ 个石子，这个子游戏的 SG 值为 $a_i$，所以总游戏的 SG 值就是 $a_1 \operatorname{xor} a_2 \operatorname{xor} \cdots \operatorname{xor} a_n$。通过这种方法，我们从另一角度得到了 Nim 定理。

