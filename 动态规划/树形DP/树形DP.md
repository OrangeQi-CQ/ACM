### 树上背包

##### [[CTSC1997] 选课 - 洛谷](https://www.luogu.com.cn/problem/P2014)

【题意】

给一个有向森林，每个点有点权。如果选择了点 $x$，那么 $x$ 的所有父节点都需要选择。总共可以选择 $M$ 个点。
求选出的点权和最大值。

```cpp
void dfs(int u) {
    siz[u] = 1;
    f[u][1] = s[u];

    for (int v : g[u]) { // 枚举分组
        dfs(v);
        siz[u] += sz[v];

        for (int j = sz[u]; j >= 0; j--) { // 枚举体积
            for (int k = 0; k <= sz[v]; k++) { // 枚举选择
                if (j > k) {
                    f[u][j] = max(f[u][j], f[u][j - k] + f[v][k]);
                }
            }
        }
    }
}
```





##### [二叉苹果树 - 洛谷](https://www.luogu.com.cn/problem/P2015)

【题意】

给一棵有根二叉树，每条边有边权。需要选择 $k$ 条边，要求选出的边构成一棵树且包含原树的根节点。

求选择的边权和最大值。

```cpp
void dfs(int u, int pa) {
    for (auto [v, w] : g[u]) { // 枚举分组
        if (v == pa) {
            continue;
        }

        dfs(v, u);

        for (int j = m; j >= 0; j--) { // 枚举体积
            f[u][j] = f[u][j];

            for (int k = 0; k <= j - 1; k++) { // 枚举选择
                f[u][j] = max(f[u][j], f[u][j - k - 1] + f[v][k] + w);
            }
        }
    }
}
```





##### [有线电视网 - 洛谷](https://www.luogu.com.cn/problem/P1273)

【题意】

给一棵有根树，有点权和边权。叶子节点的点权为正数，其余节点点权为零，边权为正数。

现在需要选择一个连通的、包含根节点的子图。子图的收益是所有原图叶子节点的点权 减去 子图中所有边的边权。

求在收益为正的前提下，最多包含多少个原图的叶子节点。







##### [重建道路 - 洛谷](https://www.luogu.com.cn/problem/P1272)

【题意】

$n$ 元无根树，给定整数 $p$，求至少删掉几条边，形成的森林中存在有 $p$ 个节点的树。














### 树上状态机



##### [P1352 没有上司的舞会](https://www.luogu.com.cn/problem/P1352)

【题意】

求树上的最大独立集。即：

给一棵有根树，每个点有点权。需要选择一些点，要求即选出的点不能有边相连。求选出的点权和最大值。

```cpp
void dfs(int u) {
    f[u][1] = a[u];
    f[u][0] = 0;

    for (int i : g[u]) {
        dfs(v);
        f[u][0] = f[u][0] + max(f[v][1], f[v][0]);
        f[u][1] = f[u][1] + f[v][0];
    }
}
```





##### [P2016 战略游戏](https://www.luogu.com.cn/problem/P2016)

【题意】

求树的最小点覆盖，即:
给一棵树。选择一些点，满足树上任意一条边的两个端点至少有一个在集合中。求最小的点集合。





##### [皇宫看守 - Acwing](https://www.acwing.com/problem/content/1079/) 

【题意】

给定无根树。一个关键点可以管辖它自身以及和它相邻的节点。求至少多少个关键点能管辖整棵树。





##### [P2279 [HNOI2003]消防局的设立](https://www.luogu.com.cn/problem/P2279)

【题意】

给定无根树。一个关键节点可以管辖它自身以及和它距离不超过 2 的节点。求至少需要多少个关键点能管辖整棵树。













### 换根DP

换根dp 的通用特点：

- 解决无根树问题
- 需要两次 dfs

一般而言，在无根树上的树形dp 都是假定一个根，然后顺带假定方向。这就导致了两个局限，分别对应了使用换根 dp 的两种情境：

1. 无法处理 $u \rightarrow pa$ 方向的子树。

> 我们可以在一次 dfs 当中根据子树信息合并根节点信息，但是无法直接处理向上的子树。这时我们就需要进行第二次 dfs，从上向下处理 “向上的子树” 的信息。
>
> 设 $f[u]$ 为 $u$ 向下的子树信息，$g[u]$ 为 $u$ 向上的子树信息，当前的递归状态是 `dfs(u, pa)` 。$g[u]$ 只能由 $pa$ 转移过来，需要考虑两部分：
>
> - $pa$ 向上的子树，即用 $g[pa]$ 更新 $g[u]$。
> - $pa$ 除了 $u$ 之外的子树。所以通常需要维护一些诸如 “次大值” 的向下信息。
>
> 代表题目
>
> [CF161D - Distance in Tree](https://codeforces.com/problemset/problem/161/D)
>
> 

2. 题目要分别求出以所有点为根的情况——此时枚举根节点每次做 dfs 显然是不现实的。而我们只能求出以点 $1$ 为根时的情形。

> 我们很容易从下向上合并得到树根为 $1$ 时问题的答案，而题目要求以所有点为根的情况下的答案。这时候我们就要从上向下考虑：将根从 $pa$ 转移到 $u$ 的时候，答案会如何变化。
>
> 代表题目
>
> [P3478 [POI2008] STA-Station](https://www.luogu.com.cn/problem/P3478)
>
> [ [USACO10MAR] Great Cow Gathering G ](https://www.luogu.com.cn/problem/P2986)



##### [CF161D - Distance in Tree](https://codeforces.com/problemset/problem/161/D)

【题意】

给大小为 $n$ 的无根树，求树上距离为 $k$ 的点对数量。$n \le 50000,\ k \le 500$。

```cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long
#define PII pair<int, int>
#define endl "\n"
/**********************  Core code begins  **********************/

void SolveTest() {
	int n, k;
	cin >> n >> k;

	vector<vector<int>> adj(n + 1);
	for (int i = 1; i < n; i++) {
		int u, v;
		cin >> u >> v;
		adj[u].push_back(v);
		adj[v].push_back(u);
	}

	vector<array<int, 505>> f(n + 1), g(n + 1);

	function<void(int, int)> dfs1 = [&](int u, int pa) {
		f[u][0] = 1;
		for (int v : adj[u]) {
			if (v == pa) {
				continue;
			}
			dfs1(v, u);
			for (int i = 1; i <= k; i++) {
				f[u][i] += f[v][i - 1];
			}
		}
	};

	function<void(int, int)> dfs2 = [&](int u, int pa) {
		g[u][0] = 1;
		if (u != 1) {
			g[u][1] = 1;
			for (int i = 2; i <= k; i++) {
				g[u][i] = g[pa][i - 1] + f[pa][i - 1] - f[u][i - 2];
			}
		}

		for (int v : adj[u]) {
			if (v == pa) {
				continue;
			}
			dfs2(v, u);
		}
	};

	dfs1(1, 0);
	dfs2(1, 0);
	int ans = 0;
	for (int i = 1; i <= n; i++) {
		ans += f[i][k] + g[i][k];
	}
	cout << ans / 2 << endl;
}

/**********************  Core code ends  ***********************/
signed main() {
	ios::sync_with_stdio(false);
	cin.tie(0);
	cout.tie(0);
	int T = 1;
	// cin >> T;
	for (int i = 1; i <= T; i++) {
		SolveTest();
	}
	return 0;
}
```





##### [P3478 [POI2008] STA-Station](https://www.luogu.com.cn/problem/P3478)

【题意】

给定一棵树，无边权。求出一个结点，使得以这个结点为根时，所有结点的深度之和最大。

```cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long
#define PII pair<int, int>
#define endl "\n"
/**********************  Core code begins  **********************/

void SolveTest() {
	int n;
	cin >> n;

	vector<vector<int>> adj(n + 1);
	for (int i = 1; i < n; i++) {
		int u, v;
		cin >> u >> v;
		adj[u].push_back(v);
		adj[v].push_back(u);
	}

	vector<int> dep(n + 1), siz(n + 1), f(n + 1), ans(n + 1);
	
	function<void(int, int)> dfs1 = [&](int u, int pa) {
		dep[u] = dep[pa] + 1;
		f[u] = dep[u];
		siz[u] = 1;
		for (int v : adj[u]) {
			if (v == pa) {
				continue;
			}
			dfs1(v, u);
			siz[u] += siz[v];
			f[u] += f[v];
		}
	};

	dfs1(1, 0);
	ans[1] = f[1];

	function<void(int, int)> dfs2 = [&](int u, int pa) {
		if (u != 1) {
			ans[u] = ans[pa] - siz[u] + (n - siz[u]);
		}
		for (int v : adj[u]) {
			if (v == pa) {
				continue;
			}
			dfs2(v, u);
		}
	};
	
	dfs2(1, 0);
	int res = max_element(ans.begin(), ans.end()) - ans.begin();
	cout << res << endl;
}

/**********************  Core code ends  ***********************/
signed main() {
	ios::sync_with_stdio(false);
	cin.tie(0);
	cout.tie(0);
	int T = 1;
	// cin >> T;
	for (int i = 1; i <= T; i++) {
		SolveTest();
	}
	return 0;
}
```



##### [ [USACO10MAR] Great Cow Gathering G ](https://www.luogu.com.cn/problem/P2986)

【题意】


$n$ 元树，有点权和边权，求点 $P$，使得以 $P$ 为根时所有节点的价值和最小。

其中点 $S$ 的价值定义为：$S$ 的点权 $\times$ $S$ 到根节点的距离。

```cpp
/**
 * 求所有点的：
 * 1. siz[u]: 子树大小
 * 2. cost[u]: 子树中所有节点到u的花费之和
*/
void dfs1(int u, int fa) {
    siz[u] = a[u];

    for (int i = head[u]; i; i = nxt[i]) {
        int v = to[i];

        if (v == fa) {
            continue;
        }

        dfs1(v, u);
        siz[u] += siz[v];
        cost[u] += cost[v] + siz[v] * edge[i];
    }
}

/**
 * 换根dp转移
*/
void dfs2(int u, int fa) {
    for (int i = head[u]; i; i = nxt[i]) {
        int v = to[i];

        if (v == fa) {
            continue;
        }

        d[v] = d[u] - siz[v] * edge[i] + (tot - siz[v]) * edge[i];
        ans = min(ans, d[v]);
        dfs2(v, u);
    }
}
```



##### [[USACO12FEB]Nearby Cows G](https://www.luogu.com.cn/problem/P3047)

【题意】

 $n$ 元树，有点权，对于每个点 $i$ 求出距离它不超过 $k$  的所有节点权值和 $m_i $。





##### [CF708C Centroids](https://www.luogu.com.cn/problem/CF708C)

【题意】

给 $n$ 元树，定义改造树：删去一条边，再加入一条边，保证改造后还是一棵树。求有多少点可以通过改造树，成为这颗树的重心？

注：如果以某个点为根，每个子树的大小都不大于 $\dfrac{n}{2}$，则称某个点为重心。









##### [[COCI2014-2015#1] Kamp](https://www.luogu.com.cn/problem/P6419)

【题意】

$n$ 元树，有边权。指定 $k$ 个特殊点。
对于树上每个点 $i$，求从点 $i$ 出发经过所有特殊点至少一次的路径的最小值。



##### [CF855C -  Helga Hufflepuff's Cup](https://codeforces.com/problemset/problem/855/C)







### 基环树DP

##### [城市环路- 洛谷](https://www.luogu.com.cn/problem/P1453)

求基环树最大点权独立集



##### [[IOI2008] Island - 洛谷](https://www.luogu.com.cn/problem/P4381)

求基环树直径





### 虚树

##### [[SDOI2011] 消耗战 - 洛谷](https://www.luogu.com.cn/problem/P2495)

$n$ 元有根树，有边权，$m$ 次询问：

- 指定 $k$ 个特殊点，要求选择一些边，使得断掉这些边后所有特殊点均无法到达根节点；

求选择的最小边权和。





##### [[HNOI2014]世界树 - 洛谷](https://www.luogu.com.cn/problem/P3233)

给n元有根树，边权为 $1$，$m$ 次询问：

- 指定 $k$ 个特殊点，对于树上的每一个点 $P$，距离 $P$ 最近（距离相同则取编号最小）的特殊点记为 $Q$，称 $Q$ 管辖 $P$；

输出每一个特殊点所管辖的点的数量。








### 长链剖分

当 dp 状态只和深度有关的时候可以通过长链剖分优化到均摊 $\Theta (1)$ 的复杂度



##### [CF1009F Dominant Indices](https://www.luogu.com.cn/problem/CF1009F)

有根 $n$ 元树。设 $d(u,x)$ 为 $u$ 子树中到 $u$  距离为 $x$ 的节点数。
对于每个点 $u$，求一个最小的 $k$，使得 $d(u,k)$ 最大。





##### [[POI2014]HOT-Hotels 加强版](https://www.luogu.com.cn/problem/P5904)

$n$ 元树，求有多少组点  $(i,j,k)$  满足 $i,j,k$ 两两之间的距离都相等（$i,j,k$ 打乱顺序算同一组）
