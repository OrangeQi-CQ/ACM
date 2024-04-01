[TOC]



# 理解

基本概念在 [oiwiki 的相关部分](https://oi-wiki.org/math/game-theory/impartial-game/#%E6%9C%89%E5%90%91%E5%9B%BE%E6%B8%B8%E6%88%8F%E4%B8%8E-sg-%E5%87%BD%E6%95%B0) 都有介绍，这里不赘述。

关键是区分 ”**SG 函数的定义**“ 和 ”**SG 定理**“，理解 ”**组合游戏**“ 和 ”**组合游戏的和**“：

- SG 函数的定义说的是**单个组合游戏**。当前的状态对应于图上的一个点 $x$，$x$ 所能一步到达的点的集合为 $F(x)$，则：

$$
g(x)=\operatorname{mex}\{g(v)\ |\ v\in F(x)\}
$$

- SG 定理说的是**若干组合游戏的和**。当前游戏 G 是由 $n$ 个组合游戏组成，第 $i$ 个组合游戏的状态为 $x_i$，SG 函数为 $g_i$。这 $n$ 个点构成了 G 的当前状态 $x=(x_1,x_2,...,x_n)$，则 G 的 SG 函数值为

$$
g(x)=g((x_1,x_2,...,x_n))=g_1(x_1) \oplus g_2(x_2)\oplus...\oplus g_n(x_n)
$$



# 题目

## [A Chess Game](https://ac.nowcoder.com/acm/problem/236917)

（sg 函数模板题）

题意：有 $n$ 个节点的有向无环图。有若干轮游戏，每轮游戏给出 $m$ 个棋子在图上的哪个节点，可能会有两个棋子在同一个节点上。两个玩家轮流移动，每次可以选择一个棋子将其移动到他的后继节点，直到有一方不能移动任何棋子，判定为输。判断先手必胜还是必败。

注意本题节点编号为 $0$ 到 $n−1$。

解法：看成 $m$ 个完全相同的图游戏。预处理这个图的 SG 函数，运用 SG 定理求解。

```cpp
#include <bits/stdc++.h>
#define int long long

signed main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    std::cout.tie(0);

    int n;
    std::cin >> n;
    std::vector adj(n, std::vector<int>());
    for (int i = 0; i < n; i++) {
        int k;
        std::cin >> k;
        for (int j = 1; j <= k; j++) {
            int x;
            std::cin >> x;
            adj[i].push_back(x);
        }
    }

    std::vector<int> sg(n, -1);
    std::function<void(int)> dfs = [&](int u) {
        if (sg[u] != -1) {
            return;
        }
        std::unordered_set<int> st;
        for (int v : adj[u]) {
            dfs(v);
            st.insert(sg[v]);
        }
        int x = 0;
        while (st.count(x)) {
            x++;
        }
        sg[u] = x;
    };

    for (int i = 0; i < n; i++) {
        dfs(i);
    }

    int m;
    while (std::cin >> m && m > 0) {
        int res = 0;
        for (int i = 1; i <= m; i++) {
            int x;
            std::cin >> x;
            res ^= sg[x];
        }
        std::cout << (res ? "WIN" : "LOSE") << "\n";
    }
    return 0;
}
```





## [Be the Winner](https://ac.nowcoder.com/acm/contest/34655/F)

题意：有 $m$ 个苹果排成一列，初始时分为 $n$（$1 \le n \le 100$）段，每段不超过 $100$ 个。每次可以从一段中取走连续的若干个苹果，加入取走了中间的一段，则左右变为独立的两端。**取走最后一个苹果的人输**。判断先手必胜还是必败。

解法：设 $sg(x)$ 为 ”连续 $x$ 个苹果“ 这个状态的 SG 值。**这是一个反常游戏**，末状态为 $sg(0)=sg(1)=0$。

考虑状态 $x$ 的转移：它会转移成左右两堆连续苹果（可以为空），左右两堆的和不超过 $x$。枚举两堆大小分别为 $l,r$，根据 SG 定理：
$$
sg(x)=\operatorname{mex}\{sg(l)\oplus sg(r) \ |\ l+r<x \}
$$
由于数据范围很小，所以可以直接暴力 $O(x)$ 枚举，预处理所有的 $sg(x)$，复杂度 $O(100^3)$。

而开始时有 $n$ 段，根据 SG 定理，把每一段的 SG 值异或起来就是答案。

但是不太理解为什么要特判全部都是 $1$ 的情况，不懂 为什么 SG 函数无法处理这种情形。

```cpp
#include <bits/stdc++.h>
#define int long long

signed main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    std::cout.tie(0);

    std::vector sg(105, -1);
    sg[0] = sg[1] = 0;
    for (int x = 1; x <= 100; x++) {
        std::unordered_set<int> mex;
        for (int i = 0; i < x; i++) {
            for (int j = 0; i + j < x; j++) {
                mex.insert(sg[i] ^ sg[j]);
            }
        }
        int res = 0;
        while (mex.count(res)) {
            res++;
        }
        sg[x] = res;
    }

    int n, res = 0, cnt1 = 0;
    std::cin >> n;
    std::vector<int> a(n + 1);
    for (int i = 1; i <= n; i++) {
        std::cin >> a[i];
        res ^= a[i];
        cnt1 += (a[i] == 1);
    }
    if (cnt1 == n) {
        std::cout << (n % 2 == 0 ? "Yes" : "No");
    } else {
        std::cout << (res ? "Yes" : "No");
    }

    return 0;
}
```



## [[HNOI2007]分裂游戏](https://ac.nowcoder.com/acm/contest/34655/H)





## [A tree game](https://ac.nowcoder.com/acm/contest/34655/I)

题意：给定一棵树，以 $1$ 为根。两个玩家轮流游戏，每一轮中当前玩家选择一条边，删掉它以及它的子树，无法操作的玩家判负。判断先手必胜还是必败。

解法：



# Nim 博弈变形

## [牛客小白月赛70E. 小d的博弈](https://ac.nowcoder.com/acm/contest/53366/E)

题意：给一个矩形。两个玩家轮流操作，每次沿一行切开或沿一列切开，要求切开后两部分长宽都是正数且两部分面积不相等，保留面积较小的部分。无法操作的玩家判负。判断先手必胜还是必败。

解法：不难发现横纵坐标是独立的，可以转化为取石子问题，以长 $x$ 为例，本质上相当于：一堆 $n$ 个石子，每次操作至少拿走 $\left\lfloor\dfrac{x}{2}\right\rfloor+1$ 个石子。无法操作的人判负。

因为 $x$ 可以转移到 $[1,\left\lceil\dfrac{x}{2}\right\rceil - 1]$ 的所有状态，所以显然 $sg(x)=sg(\left\lceil\dfrac{x}{2}\right\rceil-1)+1$。递归计算即可。

```cpp
#include <bits/stdc++.h>

int sg(int x) {
    if (x <= 2) {
        return 0;
    }
    return sg((x + 1) / 2 - 1) + 1;
}

signed main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    std::cout.tie(0);

    int T;
    std::cin >> T;
    while (T--) {
        int n, m;
        std::cin >> n >> m;
        std::cout << (sg(n) ^ sg(m) ? "Alice\n" : "Bob\n");
    }

    return 0;
}
```



## [S-Nim](https://ac.nowcoder.com/acm/problem/236918)

题意：给定一个集合 $S$，有 $n$ 堆石子，第 $i$ 堆数量为 $a_i$。两个玩家轮流从某一堆式子中选择 $s$ 个，要求 $s\in S$。判断先手必胜还是必败。

解法：每堆石子互不影响，考虑在一堆石子内求 SG 函数。暴力求 `mex` 即可。

```cpp
#include <bits/stdc++.h>
#define int long long

using PII = std::pair<int, int>;

signed main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    std::cout.tie(0);

    int k;
    std::cin >> k;
    std::vector<int> st;
    for (int i = 1; i <= k; i++) {
        int x;
        std::cin >> x;
        st.push_back(x);
    }

    const int MAXN= 1e4 + 5;
    std::vector<int> sg(MAXN, -1);

    std::function<int(int)> getSG = [&](int x) {
        if (sg[x] != -1) {
            return sg[x];
        }
        if (x == 0) {
            return sg[x] = 0;
        }
        std::unordered_set<int> mex;
        for (int v : st) {
            if (x - v >= 0) {
                getSG(x - v);
                mex.insert(sg[x - v]);
            }
        }
        int res = 0;
        while (mex.count(res)) {
            res++;
        }
        return sg[x] = res;
    };

    for (int i = 0; i < MAXN; i++) {
        sg[i] = getSG(i);
    }

    int q;
    std::cin >> q;
    while (q--) {
        int n, res = 0;
        std::cin >> n;
        for (int i = 1; i <= n; i++) {
            int x;
            std::cin >> x;
            res ^= sg[x];
        }
        std::cout << (res ? "W" : "L");
    }

    return 0;
}
```



## [Stone Game](https://ac.nowcoder.com/acm/contest/34655/C)

题意：$n$ 个箱子，第 $i$ 个箱子最多放 $s_i$ 个石子，初始时箱子里的石子数量为 $a_i$。两个人轮流往箱子里放石子，但是每一次放的数量有限制：不能超过当前箱子内石子的平方。不能放石子的人输，判断先手必胜还是必败。

解法：建模方法和上一题相同。本题数据规模有 $10^6$，如果暴力枚举转移关系会 TLE。需要优化。

考虑一个箱子内已经有 $x$，最多放 $s$。假设当前状态的 sg 函数值为 $sg(x,s)$。

首先找到最大的 $t$，满足当前有 $t$ 个石子的时候无法一步放满箱子，即最大的 $t$ 满足 $t + t^2 <s$。讨论当前 $x$ 和 $t$ 的关系：

1. $x>t$：说明 $x$ 可以一步放满。$x$ 可以转移到 $x+1,x+2,..,s$，而 $sg(s,s)=0$，所以有：
    -  $sg(s - 1, s)= \operatorname{mex}\{0\}=1$，
    -  $sg(s-2,s)=  \operatorname{mex}\{0,1\}=2$，
    -  $...$
    -  $sg(x,s)=s-x$
2. $x=t$：本轮无法一步放满，而无论这一轮放多少，对手都可以一步放满，所以必败， $sg(x,s)=0$。
3. $x<t$：如果放了$v$ 个石子后 $x+v>t$，那么对手显然可以一步放满。所以一定要保证自己操作之后总个数不超过 $t$。所以 $sg(x, s)=sg(x, t)$。

```cpp
#include <bits/stdc++.h>
#define int long long

int sg(int x, int s) {
    if (x == 0) {
        return 0ll;
    }
    int t = sqrt(1.0 * s);
    while (t + t * t >= s) {
        t--;
    }
    if (x > t) {
        return s - x;
    }
    if (x == t) {
        return 0ll;
    }
    return sg(x, t);
}

signed main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    std::cout.tie(0);

    int n, res = 0;
    std::cin >> n;
    for (int i = 1; i <= n; i++) {
        int x, s;
        std::cin >> s >> x;
        res ^= sg(x, s);
    }

    std::cout << (res ? "Yes" : "No") << "\n";
    return 0;
}
```





## [2022 北理校赛 J. Stones](https://codeforces.com/gym/104025/problem/J)

题意：$n$ 堆石子，第 $i$ 堆石子有 $a_i$ 个。两个人轮流取石子，每一步只能在一堆当中取 $x$ 个石子，满足 $1 \le x \le\dfrac{a_i+1}{2}$。然后 $a_i \rightarrow a_i-x$。判断先手必胜还是必败。





## [ABC255G- Constrained Nim](https://atcoder.jp/contests/abc255/tasks/abc255_g)

题意：$n$ 堆石子，第 $i$ 堆石子有 $a_i$ 个。两个人轮流取石子。给出 $m$ 组约束 $(x_i,y_i)$，表示不允许从大小为 $x_i$ 的石子中拿走 $y_i$ 个。判断先手必胜还是必败。

$1\le n,m\le 2\times 10^5$，$1\le y_i \le a_i,x_i\le 10^{18}$。

思路：做完这道题，加深了对 “利用 $\operatorname{mex}$ 计算 SG 函数的过程“ 的理解。

按照 SG 定理的套路，考虑：怎样计算一堆石子当中的 SG 函数，然后所有堆异或起来就是答案。

对于题目中的约束 $(x_i,y_i)$，其实就是限制点 $x_i$ 无法转移到点 $x_i-y_i$，否则点 $x$ 都可以转移到任何 $y<x$。

首先，如果 $x,x+1$ 都没有转移的限制，那么一定有 $sg(x+1)=sg(x)+1$。所以我们需要找到 $x$ 左边最靠右的某个点 $t$，要求 $t$ 满足：$t$ 没有转移约束，但是 $t-1$ 存在转移约束。那么 $sg(x)=sg(t)+(x-t)$。具体的，可以用 `std::map<int, int> sg` 储存存在约束的点，以及存在约束的点右边相邻点的 $sg$ 值，然后在 `std::map` 上 `lower_bound` 即可。

```cpp
    std::map<int, int> sg;                  
    sg[0] = 0;

    auto getSG = [&](int x) {
        if (sg.count(x)) {
            return sg[x];
        }
        auto it = prev(sg.lower_bound(x));
        return it->second + x - it->first; // sg(x) = sg(t) + (x-t)
    };
```

关键的地方在于处理存在转移限制的点。从小到大处理所有存在转移限制的 $x$。由于我们是从小到大处理的 $x$，所以其实已经可以 $O(\log n)$ 求任意比 $x$ 小的点的 $sg$ 值了，复杂度仅仅是访问 `std::map` 的复杂度。

对于第一个存在限制的点 $x$，假设它不能转移到当中最小的点是 $y$，那么根据 $\operatorname{mex}$ 函数的定义，一定有 $sg(x)=sg(y)$。但是对于后面的点肯定不满足这个性质，因为现在的点 $x$ 虽然不能转移到 $y$，但是可能有其他 $x$ 可以转移到的点 $z$，满足 $sg(z)=sg(y)$。

那么产生这个现象的原因是什么呢？如果没有限制（最单纯的 Nim 博弈），$sg(x)=x$ 保证了每个 $sg$ 值都只存在一次。但是对于 $x$ 所无法转移到的最小 $sg$ 值，$sg(x)$ 会 ”降落“ 到这个最小的 $sg$ 值，造成一个 $sg$ 值对应两个点。所以维护一个桶 `std::map<int, int> cnt`，记录遍历到 $x$ 的时候，每一个 `sg` 值被额外重复了多少次。

记 $x$ 不能转移到的点的 $sg$ 值构成的集合为 $S$，再开一个桶 `std::map<int, int> tmp` 记录 $S$ 中值的出现次数。从小到大枚举 $S$ 中的值 $v$ 和出现的次数 $c$，如果 `cnt[v] + 1 <= c`，就说明这个点的 $sg$ 值是 $x$ 无法转移到的。因此 $sg[x]=v$。如果不存在这样的值 $v$，就说明 $sg(x)$ 不会 ”下降“，$x$ 在计算的时候当作一个没有约束的点即可。

同时，我们还要维护前缀最大的 $sg$ 值 `maxv`，令 `sg(x+1) = maxv`。

代码不长，但是思维量蛮大。

```cpp
#include <bits/stdc++.h>
#define int long long

signed main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    std::cout.tie(0);

    int n, m;
    std::cin >> n >> m;
    std::vector<int> a(n + 1);
    for (int i = 1; i <= n; i++) {
        std::cin >> a[i];
    }

    std::map<int, std::vector<int>> cons;    // 记录限制, constraint

    for (int i = 1; i <= m; i++) {
        int x, y;
        std::cin >> x >> y;
        cons[x].push_back(x - y);
    }

    std::map<int, int> sg;                  
    sg[0] = 0;

    auto getSG = [&](int x) {
        if (sg.count(x)) {
            return sg[x];
        }
        auto it = prev(sg.lower_bound(x));
        return it->second + x - it->first; // sg(x) = sg(t) + (x-t)
    };

    std::map<int, int> cnt;
    int maxv = -1;

    for (auto [x, vec] : cons) {
        std::sort(vec.begin(), vec.end());
        std::map<int, int> tmp;
        for (int v : vec) {
            tmp[getSG(v)]++;
        }
        int tmpv = 1e18;
        for (auto [v, c] : tmp) {
            if (cnt[v] + 1 <= c) {
                tmpv = v;
                break;
            }
        }
        if (tmpv < 1e18) {
            cnt[tmpv]++;
            sg[x] = tmpv;
        }

        maxv = std::max({getSG(x - 1), getSG(x), maxv});
        sg[x + 1] = maxv + 1;
    }

    int res = 0;
    for (int i = 1; i <= n; i++) {
        res ^= getSG(a[i]);
    }
    std::cout << (res ? "Takahashi" : "Aoki");
    return 0;
}
```



## [ABC297G - Constrained Nim 2](https://atcoder.jp/contests/abc297/tasks/abc297_g)

题意：$n$ 堆石子，第 $i$ 堆石子有 $a_i$ 个。两个人轮流取石子，每一步选择石子数量 $x$ 的范围是 $[l,r]$。其中 $l,r$ 是给定的常数。判断先手必胜还是必败。

解法：参考 [官方题解](https://atcoder.jp/contests/abc297/editorial/6205)，写的很有启发性。

这道题的关键思想是：对于当前的状态 $x$，考虑它转移后 $[\max(0,x-r),x-l]$ 的范围，进而挖掘 SG 函数转移时的性质。

首先考虑 $x < l+r$ 的情况。在这种情况下，转移后的范围都是 $[0,x-l]$。

1. 若 $x<l$。无法操作，$sg(x)=0$；
2. 若 $l\le x < 2l$。转移后的范围 $[0,x-l]$，而 $x-l<l$，也就是说一定会转移到情形 $1$。所以 $sg(x)=1$；
3. 若 $2l\le x < 3l$。转移后的范围 $[0,x-l]$，而 $x-l<2l$，也就是说一定会转移到情形 $1$ 或情形 $2$。所以 $sg(x)=2$；
4. 同理，若 $k\cdot l\le x<(k+1)\cdot l$，转移后的范围 $[0,x-l]$，而 $x-l < k \cdot l$，也就是说一定会转移到情形 $1,2,...,k$，所 $sg(x)=k=\left\lfloor \dfrac{x}{l} \right\rfloor$。

下面考虑 $l+r \le x <2(l+r)$ 的情况。

1. 若 $l+r\le x < (l+r)+l$，那么它转移后的范围是 $[l,l+r]$。它转移后的范围内的 $sg$ 值不包含 $0$，所以 $sg(x)=0$；
2. $...$

到这里其实就已经能找到规律了，$sg(x)$ 会以 $(l+r)$ 为周期。在一个周期内，每当 $x$ 增加 $l$，$sg(x)$ 就会增加 $1$。结论就是 $sg(x)= \left\lfloor \dfrac{x \bmod (l+r)}{l} \right\rfloor$。

```cpp
#include <bits/stdc++.h>
#define int long long

signed main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    std::cout.tie(0);
    
    int n, l, r, res = 0;
    std::cin >> n >> l >> r;
    
    auto sg = [&](int x) {
		return (x % (l + r) / l);
    };
    
    for (int i = 1; i <= n; i++) {
        int x;
        std::cin >> x;
        res ^= sg(x);
    }    
    std::cout << (res ? "First" : "Second");
    return 0;
}
```

