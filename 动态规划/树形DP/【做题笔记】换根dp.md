# 换根DP

用于解决：无根树，对所有点的询问。

一般而言有两种思路：

- 根据根节点整棵树拆分为向上的子树和向下的子树两部分。
- 专注于换根的过程答案怎样变化。



无根树的子树：（假定 $1$ 是暂时假定的根节点）

- 对于无根树上的一条边 $(u,v)$，当把 $(u,v)$ 断开后，会形成两棵子树。$u$ 到 $1$ 的距离较近 $\iff$  $u$ 所在的子树会包含 $1$。 
- $v$ 所在的子树记为 $down(v)$，$u$ 所在的子树记为 $up(v)$。子树的根是确定的，即与断边相连的点。
- 任何一棵子树都是某个 $up(i)$ 或 $down(i)$，换根 dp 可以求出任何一棵子树的信息。
- 综上：换根 dp 本质上是可以得到无根树任何一棵子树的 dp 信息。再次注意无根树的子树是有唯一的树根。
- 以 $u$ 为树根时的整棵树的信息也等价于：把整棵树看成一棵子树（把 $u$ 和 $0$ 切开）。



通用套路：

```cpp
// 需要两个 dp 数组，分别是 dp 和 up

void dfs1(int u, int pa) {
     // TODO: dp[u] 的初始化
	for (int v : adj[u]) {
		if (v == pa) {
            continue;
        }
        dfs1(v, u);
        // TODO: 用 dp[v] 更新 dp[u]
    }
}

void dfs2(int u, int pa) {
	for (int v : adj[u]) {
        if (v == pa) {
            continue;
        }
        // TODO: 用 up[u] 和 dp[u] 更新 up[v] （换根 dp 的难点）
        dfs2(v, u);
    }
    // TODO: 用 up[u] 更新 dp[u]
}
```

核心在于，将 `up[v]` 分解为两部分：

- $u$ 向上的唯一子树，即 $up[u]$；
- $u$ 向下的所有子树，除去 $v$ 所在的子树。
    - 如果是计数问题，可以利用容斥，用 $u$ 向下的所有子树贡献之和减去 $v$ 向下所有子树贡献之和。
    - 如果是优化问题，需要维护 $u$ 向下的最优子树和次优子树。如果子树 $v$ 的答案等于 $u$ 的最优子树，那么用 $u$ 的次优子树更新 $up[v]$，否则用 $u$ 的最优子树更新 $up[v]$。 



## [P3478 [POI2008] STA-Station](https://www.luogu.com.cn/problem/P3478)

【题意】

给定一棵树，无边权。求出一个结点，使得以这个结点为根时，所有结点的深度之和最大。

【做法】

先求出以 $1$ 为根时的答案。然后从上向下转移根。

```cpp
#include <bits/stdc++.h>

using i64 = long long;

signed main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);

    int n;
    std::cin >> n;
    std::vector<std::vector<int>> adj(n + 1);

    for (int i = 1; i < n; i++) {
        int u, v;
        std::cin >> u >> v;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    // 每个点的深度，向下的子树大小
    std::vector<int> dep(n + 1), siz(n + 1);
    // dp[u] 表示以 u 为 根时，所有节点的深度之和。
    std::vector<i64> dp(n + 1);

    auto dfs1 = [&](auto self, int u, int pa) -> void {
        siz[u] = 1;
        for (int v : adj[u]) {
            if (v == pa) {
                continue;
            }
            dep[v] = dep[u] + 1;
            self(self, v, u);
            siz[u] += siz[v];
            dp[u] += dp[v];
        }
    };

    dfs1(dfs1, 1, 0);

    auto dfs2 = [&](auto self, int u, int pa) -> void {
        for (int v : adj[u]) {
            if (v == pa) {
                continue;
            }
            dp[v] = dp[u] + (n - siz[v]) - siz[v];
            self(self, v, u);
        }
    };

    dfs2(dfs2, 1, 0);
    std::cout << std::max_element(dp.begin() + 1, dp.end()) - dp.begin();

    return 0;
}
```





## [161D. Distance in Tree](https://codeforces.com/problemset/problem/161/D)

【题意】

给大小为 $n$ 的无根树，求树上距离为 $k$ 的点对数量。

$n \le 5\times 10^4,\ k \le 500$。

【做法】

转换为，对每个点求：有多少个点距离它为 $k$。

```cpp
#include <bits/stdc++.h>

using i64 = long long;

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);

    int n, k;
    std::cin >> n >> k;

    std::vector adj(n + 1, std::vector<int>());
    for (int i = 1; i < n; i++) {
        int u, v;
        std::cin >> u >> v;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    // dp[u][i] 表示：距离 u 为 i 的点数，即以 u 为根时深度为 i 的点数
    std::vector dp(n + 1, std::vector<int>(k + 1, 0)), up(dp);

    auto dfs1 = [&](auto self, int u, int pa) -> void {
        dp[u][0] = 1;
        for (int v : adj[u]) {
            if (v == pa) {
                continue;
            }
            self(self, v, u);
            for (int i = 1; i <= k; i++) {
                dp[u][i] += dp[v][i - 1];
            }
        }
    };

    dfs1(dfs1, 1, 0);

    auto dfs2 = [&](auto self, int u, int pa) -> void {
        for (int v : adj[u]) {
            if (v == pa) {
                continue;
            }
            up[v][1] += 1;
            for (int i = 2; i <= k; i++) {
                up[v][i] += up[u][i - 1];
                up[v][i] += dp[u][i - 1] - dp[v][i - 2];
            }
            self(self, v, u);
        }
        for (int i = 1; i <= k; i++) {
            dp[u][i] += up[u][i];
        }
    };

    dfs2(dfs2, 1, 0);

    i64 ans = 0;
    for (int i = 1; i <= n; i++) {
        ans += dp[i][k];
    }
    std::cout << ans / 2;

    return 0;
}
```



## [708C. Centroids](http://codeforces.com/problemset/problem/708/C)

【题意】

给 $n$ 个点的无根树，可以进行一次操作：删去一条边，再加入一条边，保证改造后还是一棵树。求有多少点可以通过至多一次操作后，满足：如果以某个点为根，每个子树的大小都不大于 $\dfrac{n}{2}$。

【思路】

假设根节点确定为 $u$ ，考虑怎样判断能否通过一次操作使得该根节点满足要求。

- 最大的子树大小不大于 $\dfrac{n}{2}$，那么不需要操作；
- 否则，最多只有一棵子树 $v$ 大小超过 $\dfrac{n}{2}$。这时可以构造出一种贪心地策略：从子树 $v$ 中，拿走一棵大小不超过 $\dfrac{n}{2}$ 且最大的子树，接到根节点上。如果子树 $v$ 的大小减去拿走的这棵子树的大小仍然大于 $\dfrac{n}{2}$，那么 $u$ 作为根节点就无法满足要求。

那么问题就等价于：

- 对于每个点，求与它相连的最大子树。
- 对于每棵子树，求这棵子树中大小不超过 $\dfrac{n}{2}$ 的最大子树。这个是不难转移的：`dp[u] = (siz[v] > n / 2 ? dp[v] : siz[v])`。

```cpp
#include <bits/stdc++.h>

using i64 = long long;

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);

    int n;
    std::cin >> n;
    std::vector adj(n + 1, std::vector<int>());
    for (int i = 1; i < n; i++) {
        int u, v;
        std::cin >> u >> v;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    // siz[u] 表示 u 向下的子树大小。wson[u] 表示 u 向下的重儿子。
    std::vector<int> siz(n + 1), wson(n + 1);
    // down[u][0/1] 表示 u 向下的子树中能取走的大小不超过 n/2 的最大/次大子树大小
    std::vector<std::array<int, 2>> down(n + 1);
    // up[u] 表示 u 向上的子树中能取走的大小不超过 n/2 的最大/次大子树大小
    std::vector<int> up(n + 1);

    auto dfs1 = [&](auto self, int u, int pa) -> void {
        siz[u] = 1;
        for (int v : adj[u]) {
            if (v == pa) {
                continue;
            }
            self(self, v, u);
            // 更新 siz[u] 和 wson[u]
            siz[u] += siz[v];
            if (!wson[u] || siz[v] > siz[wson[u]]) {
                wson[u] = v;
            }
            // 更新 down[u]
            int tmp = siz[v] > n / 2 ? down[v][0] : siz[v];
            for (int i = 0; i <= 1; i++) {
                if (tmp > down[u][i]) {
                    std::swap(tmp, down[u][i]);
                }
            }
        }
    };

    dfs1(dfs1, 1, 0);

    auto dfs2 = [&](auto self, int u, int pa) -> void {
        for (int v : adj[u]) {
            if (v == pa) {
                continue;
            }
            // up[v] 由 down[u] 转移
            if (down[u][0] == (siz[v] > n / 2 ? down[v][0] : siz[v])) {
                up[v] = std::max(up[v], down[u][1]);
            } else {
                up[v] = std::max(up[v], down[u][0]);
            }
            // up[v] 由 up[u] 转移
            if (n - siz[u] <= n / 2) {
                up[v] = std::max(up[v], n - siz[u]);
            } else {
                up[v] = std::max(up[v], up[u]);
            }
            
            self(self, v, u);
        }
    };

    dfs2(dfs2, 1, 0);

    for (int i = 1; i <= n; i++) {
        bool ans = false;
        // 不需要操作就满足要求
        if (siz[wson[i]] <= n / 2 && n - siz[i] <= n / 2) {
            ans = true;
        }
        // i 向下的某棵子树大于 n/2 
        if (siz[wson[i]] > n / 2) {
            int x = wson[i];
            if (siz[x] - down[x][0] <= n / 2) {
                ans = true;
            }
        }
        // i 向上的子树大于 n/2
        if (n - siz[i] > n / 2) {
            if (n - siz[i] - up[i] <= n / 2) {
                ans = true;
            }
        }
        std::cout << ans << " ";
    }

    return 0;
}
```





## [[COCI2014-2015#1] Kamp](https://www.luogu.com.cn/problem/P6419)

【题意】

$n$ 个点的无根树，有边权。指定 $k$ 个特殊点。
对于树上每个点 $i$，求：从点 $i$ 出发经过所有特殊点至少一次的最小路径边权和。

【做法】

由于树的结构特点，题目等价于求：从根出发经过所有特殊点至少一次再返回根的路径边权和，减去到根最远的特殊点的距离。所以本题是要求两个东西：

- 对于每个点，求它与所有关键点形成的虚树的边权和。这个比较容易进行换根转移，属于 “专注于换根的过程答案怎样变化” 的类型。
- 对于每个点，求它到所有关键点当中的最远距离。这个是比较典型的 “拆分为向下和向上的子树”。

对于固定的根节点 $1$，从 $1$ 出发经过所有特殊点至少一次 $\iff$ 从 $1$ 出发经过所有特殊点后再返回 $1$，减去

```cpp
#include <bits/stdc++.h>

using i64 = long long;

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);

    int n, k;
    std::cin >> n >> k;
    std::vector adj(n + 1, std::vector<std::pair<int, i64>>());
    for (int i = 1; i < n; i++) {
        int u, v, w;
        std::cin >> u >> v >> w;
        adj[u].push_back({v, w});
        adj[v].push_back({u, w});
    }

    // cnt[i] 表示 i 向下的子树中，特殊点的数量
    std::vector<int> cnt(n + 1);
    for (int i = 1; i <= n; i++) {
        int x;
        std::cin >> x;
        cnt[x] = 1;
    }

    // dp1[i] 表示从 i 出发经过所有根节点再返回 i 的路径边权和
    std::vector<i64> dp1(n + 1);
    // down[u][0/1] 表示 u 向下的子树中，从 u 出发所能到达的最远/次远特殊点的距离。
    std::vector<std::array<i64, 2>> down(n + 1);
    // up[u] 表示 u 向上的子树中，从 u 出发所能到达的最远特殊点的距离。
    // dp2[u] 表示整棵树中，从 u 出发所能到达的最远特殊点的距离
    std::vector<i64> up(n + 1), dp2(n + 1);

    auto dfs1 = [&](auto self, int u, int pa) -> void {
        for (auto [v, w] : adj[u]) {
            if (v == pa) {
                continue;
            }
            self(self, v, u);
            cnt[u] += cnt[v];
            if (cnt[v] == 0) {
                continue;
            }
            // 更新 dp1[u]
            dp1[u] += dp1[v] + w * 2;
            // 更新 down[u]
            i64 val = down[v][0] + w;
            for (int i = 0; i <= 1; i++) {
                if (val > down[u][i]) {
                    std::swap(val, down[u][i]);
                }
            }
        }
    };

    dfs1(dfs1, 1, 0);

    auto dfs2 = [&](auto self, int u, int pa) -> void {
        for (auto [v, w] : adj[u]) {
            if (v == pa) {
                continue;
            }
            // 更新 dp1[u]
            if (cnt[v] == 0) {  // 如果子树 v 内没有特殊点：原本不需要走 u->v，现在需要走
                dp1[v] = dp1[u] + w * 2;
            } else if (cnt[v] == k) {   // 如果子树 v 内包含所有特殊点：原本需要走 u->v，现在不需要走
                dp1[v] = dp1[u] - w * 2;
            } else {  // 子树 v 内包含特殊点，但不包含全部特殊点：没有变化
                dp1[v] = dp1[u];
            }
            // 更新 up[u]
            if (cnt[v] == k) {
                up[v] = 0;
            } else if (down[u][0] == down[v][0] + w) {
                up[v] = std::max(up[u], down[u][1]) + w;
            } else {
                up[v] = std::max(up[u], down[u][0]) + w;
            }
            self(self, v, u);
        }
        dp2[u] = std::max(down[u][0], up[u]);
    };

    dfs2(dfs2, 1, 0);

    for (int i = 1; i <= n; i++) {
        std::cout << dp1[i] - dp2[i] << "\n";
    }

    return 0;
}
```



## [791D. Bear and Tree Jumps](https://codeforces.com/problemset/problem/791/D)

```cpp
#include <bits/stdc++.h>
using i64 = long long;

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);

    int n, k;
    std::cin >> n >> k;
    std::vector<std::vector<int>> adj(n + 1);

    for (int i = 1; i < n; i++) {
        int u, v;
        std::cin >> u >> v;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    // f[i][i]：以 i 为根的子树中，到点 i 距离模 k 余 i 的点贡献之和；
    // g[i][i]：以 i 为根的子树中，到点 i 距离模 k 余 i 的点个数；
    // 为了方便，将 f[i][0] 写为 f[i][k]，g,upf,upg 同理
    std::vector<std::array<i64, 6>> f(n + 1), g(n + 1), upf(n + 1), upg(n + 1);
    std::vector<int> siz(n + 1);

    std::function<void(int, int)> dfs1 = [&](int u, int pa) {
        g[u][k] = 1;
        siz[u] = 1;
        for (int v : adj[u]) {
            if (v == pa) {
                continue;
            }
            dfs1(v, u);
            siz[u] += siz[v];
            f[u][1] += f[v][k] + g[v][k];
            g[u][1] += g[v][k];
            for (int i = 2; i <= k; i++) {
                f[u][i] += f[v][i - 1];
                g[u][i] += g[v][i - 1];
            }
        }
    };

    std::function<void(int, int)> dfs2 = [&](int u, int pa) {
        if (pa) {
            upf[u][1] = upf[pa][k] + upg[pa][k] + (f[pa][k] - f[u][k - 1]) + (g[pa][k] - g[u][k - 1]);
            upg[u][1] = upg[pa][k] + g[pa][k] - g[u][k - 1];
            for (int i = 2; i <= k; i++) {
                upf[u][i] += upf[pa][i - 1] + f[pa][i - 1] - (i == 2 ? f[u][k] + g[u][k] : f[u][i - 2]);
                upg[u][i] += upg[pa][i - 1] + g[pa][i - 1] - (i == 2 ? g[u][k] : g[u][i - 2]);
            }
        }
        for (int v : adj[u]) {
            if (v != pa) {
                dfs2(v, u);
            }
        }
    };


    dfs1(1, 0);
    dfs2(1, 0);

    if (k == 1) {
        i64 ans = 0;
        for (int i = 1; i <= n; i++) {
            ans += 1ll * siz[i] * (n - siz[i]);
        }
        std::cout << ans;
        return 0;
    }

    i64 ans = 0;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= k; j++) {
            ans += f[i][j] + upf[i][j];
            // fprintf(stderr, "(i:%d,j:%d), f = %lld, g = %lld, upf = %lld, upg = %lld\n", i, j, f[i][j], g[i][j], upf[i][j], upg[i][j]);
        }
    }
    std::cout << ans / 2;

    return 0;
}
```



## [2022CCPC 桂林G. Group Homework](https://codeforces.com/gym/104008/problem/G)

【题意】

给一棵树，每个点有点权。从树中选择两条路径，最大化恰好被其中一条路径覆盖的所有点权之和。输出这些点权的和。

$1\le n\le 2\times 10^5$。

【做法】

首先这两条路径最多只有一个交点，否则可以等价为不超过一个交点的两条链。所以只有两种情况：

- 从一个点 $u$ 出发的 $4$ 条链，它们除了 $u$ 之外没有交点。
- 两条没有交点，分别位于 $u$ 向下的子树与 $u$ 向上的子树（向上的子树不包括 $u$）中。

综上需要维护：

- 从每个点出发的 $4$ 条分属不同子树的最长链；
- 每个子树内的最长链。

```cpp
#include <bits/stdc++.h>

using i64 = long long;

struct Arr {
    static const int N = 4;
    i64 a[N] = {};
    void insert(i64 x) {
        for (int i = 0; i < N; i++) {
            if (x > a[i]) {
                std::swap(x, a[i]);
            }
        }
    }
    i64 operator[](int i) {
        assert(i < N);
        return a[i];
    }
};

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);

    int n;
    std::cin >> n;
    std::vector<int> a(n + 1);
    for (int i = 1; i <= n; i++) {
        std::cin >> a[i];
    }

    std::vector adj(n + 1, std::vector<int>());
    for (int i = 1; i < n; i++) {
        int u, v;
        std::cin >> u >> v;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    if (n == 1) {
        std::cout << 0;
        return 0;
    }

    /**
     * down1：向下的子树内从子树根出发的最大路径
     * up1：向上的子树内从子树根出发的最大路径
     * mx1：维护每个点为根时，所有子树内从子树根出发的前 4 大路径
     * 
     * down2：向下的子树内的最大链
     * up2: 向上的子树内的最大链
     * mx2：维护每个点为根时，所有子树内的前 2 大链
    */

    std::vector<Arr> mx1(n + 1), mx2(n + 1);
    std::vector<i64> down1(n + 1), up1(n + 1), down2(n + 1), up2(n + 1);

    auto dfs1 = [&](auto self, int u, int pa) -> void {
        for (int v : adj[u]) {
            if (v == pa) {
                continue;
            }
            self(self, v, u);
            mx1[u].insert(down1[v]);
            mx2[u].insert(down2[v]);
        }
        down1[u] = mx1[u][0] + a[u];
        down2[u] = std::max(mx2[u][0], mx1[u][0] + mx1[u][1] + a[u]);
    };

    dfs1(dfs1, 1, 0);

    auto dfs2 = [&](auto self, int u, int pa) -> void {
        for (int v : adj[u]) {
            if (v == pa) {
                continue;
            }
            // 更新 up1[v]
            if (mx1[u][0] == down1[v]) {
                up1[v] = std::max(up1[u], mx1[u][1]) + a[u];
            } else {
                up1[v] = std::max(up1[u], mx1[u][0]) + a[u];
            }
            // 用 up1[v] 更新 mx1[v]
            mx1[v].insert(up1[v]);

            // 用经过 u且不进入子树 v 的最大链更新 up2[v]
            if (mx1[u][0] == down1[v]) {
                up2[v] = std::max(up2[v], mx1[u][1] + mx1[u][2] + a[u]);
            } else if (mx1[u][1] == down1[v]) {
                up2[v] = std::max(up2[v], mx1[u][0] + mx1[u][2] + a[u]);
            } else {
                up2[v] = std::max(up2[v], mx1[u][0] + mx1[u][1] + a[u]);
            }
            // 用 u 的非 v 子树中的最大链更新 up2[v]
            if (mx2[u][0] == down2[v]) {
                up2[v] = std::max(up2[v], mx2[u][1]);
            } else {
                up2[v] = std::max(up2[v], mx2[u][0]);
            }
            // 用 up2[v] 更新 mx2[v]
            mx2[v].insert(up2[v]);
            
            self(self, v, u);
        }
    };

    dfs2(dfs2, 1, 0);

    i64 ans = 0;
    for (int u = 1; u <= n; u++) {
        ans = std::max({ans, mx1[u][0] + mx1[u][1] + mx1[u][2] + mx1[u][3], down2[u] + up2[u]});
    }
    std::cout << ans;

    return 0;
}
```





## [1156D. 0-1-Tree](https://codeforces.com/problemset/problem/1156/D)

【题意】

给一棵无根树，边权为 $0/1$。一条合法路径 $(x, y)$（$x\ne y$） 满足：从 $x$ 走到 $y$ 的过程中一旦经过边权为 $1$ 的边，就不能再经过边权为 $0$ 的边。求有多少合法路径。

$2\le n\le 2\times 10^5$。

【做法】

对于每棵子树，求以 $1$ 开始的路径数量和以 $0$ 开始的路径数量。

```cpp
#include <bits/stdc++.h>

using i64 = long long;

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);

    int n;
    std::cin >> n;
    std::vector adj(n + 1, std::vector<std::pair<int, int>>());
    for (int i = 1; i < n; i++) {
        int u, v, w;
        std::cin >> u >> v >> w;
        adj[u].push_back({v, w});
        adj[v].push_back({u, w});
    }

    std::vector<i64> down1(n + 1), up1(n + 1), down0(n + 1), up0(n + 1);

    auto dfs1 = [&](auto self, int u, int pa) -> void {
        for (auto [v, w] : adj[u]) {
            if (v == pa) {
                continue;
            }
            self(self, v, u);
            if (w == 0) {
                down0[u] += down0[v] + down1[v] + 1;
            } else {
                down1[u] += down1[v] + 1;
            }
        }
    };

    auto dfs2 = [&](auto self, int u, int pa) -> void {
        for (auto [v, w] : adj[u]) {
            if (v == pa) {
                continue;
            }
            if (w == 0) {
                i64 tmp = down0[u] - down0[v] + down1[u] - down1[v] - 1;
                up0[v] = tmp + up0[u] + up1[u] + 1;
            } else {
                i64 tmp = down1[u] - down1[v] - 1;
                up1[v] = tmp + up1[u] + 1;
            }
            self(self, v, u);
        }
    };

    dfs1(dfs1, 1, 0);
    dfs2(dfs2, 1, 0);

    i64 ans = 0;
    for (int i = 1; i <= n; i++) {
        ans += down0[i] + down1[i] + up0[i] + up1[i];
    }
    std::cout << ans;

    return 0;
}
```

