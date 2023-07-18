# 换根DP

换根dp 的通用特点：

- 无根树
- 需要两次 dfs



## 第一类问题

一般来说，在无根树上的树形 dp 都是假定一个根节点例如点 $1$，那么这棵树就有了方向。这样我们通常可以通过一次 dfs 的树形 dp ，自下而上合并（可以理解为 dfs 后序更新答案）得到每个点**向下的子树**的信息。

作为真正的无根树，每个点除了是一棵向下的子树的根，还是一棵**向上的子树**的根。而我们考虑假设 $u$ 的父节点是 $pa$ 这棵向上的子树其实分两部分：

1. $pa$ 向上的子树（包括 $pa$ 自身）；
2. $pa$ 的子树当中，除了 $u$ 之外的其他子树。

其中第二部分决定了我们没有办法在一次 dfs 中同时获得向下和向上子树的信息。所以在进行第一次dfs 之后，我们还需要进行第二次dfs。

设 $f[u]$ 为 $u$ 向下的子树信息，$g[u]$ 为 $u$ 向上的子树信息，当前的递归状态是 `dfs(u, pa)` 。$g[u]$ 只能由 $pa$ 转移过来，根据向上的子树的两部分：

1. 用 $g[pa]$ 更新 $g[u]$。
2. 据题分析怎样处理 $pa$ 向下的子树当中除 $u$ 之外的子树。有时用 $pa$ 的信息简单减去 $u$ 的信息即可，有时需要维护一些诸如 “次大值” 的向下信息。

代表题目

[CF161D - Distance in Tree](https://codeforces.com/problemset/problem/161/D)



## 第二类问题

题目要分别求出以所有点为根的情况。

如果枚举根节点，做 $n$ 次 dfs 显然是不现实的。

而换根 dp 的做法是：先假定一个根，用一次 dfs 从下向上（可以理解为 dfs 后序更新答案）求出以某个点（例如点 $1$）为根的答案。

然后我们从点 $1$ 开始，考虑把当前的根节点挪到一个相邻的根节点，答案会怎样变化（加了多少，减了多少）。例如当前根从 $pa$ 移动到 $u$，我们可以分别思考 $u$ 向下的子树 和 $pa$ 向上的子树对答案影响的变化；也可以直接列公式对比做差，等等。

整个过程是 dfs 从上向下（dfs 先序）求出以每个点为根时的答案。



代表题目

[P3478 [POI2008] STA-Station](https://www.luogu.com.cn/problem/P3478)

[[USACO10MAR] Great Cow Gathering G ](https://www.luogu.com.cn/problem/P2986)



### [CF161D - Distance in Tree](https://codeforces.com/problemset/problem/161/D)

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





### [P3478 [POI2008] STA-Station](https://www.luogu.com.cn/problem/P3478)

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



### [ [USACO10MAR] Great Cow Gathering G ](https://www.luogu.com.cn/problem/P2986)

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



### [[USACO12FEB]Nearby Cows G](https://www.luogu.com.cn/problem/P3047)

【题意】

 $n$ 元树，有点权，对于每个点 $i$ 求出距离它不超过 $k$  的所有节点权值和 $m_i $。

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
	vector<int> a(n + 1);
	for (int i = 1; i < n; i++) {
		int u, v;
		cin >> u >> v;
		adj[u].push_back(v);
		adj[v].push_back(u);
	}
	for (int i = 1; i <= n; i++) {
		cin >> a[i];
	}

	vector<array<int, 25>> f(n + 1), g(n + 1);

	function<void(int, int)> dfs1 = [&](int u, int pa) {
		f[u][0] = a[u];
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

	dfs1(1, 0);

	function<void(int, int)> dfs2 = [&](int u, int pa) {
		g[u][0] = a[u];
		if (u != 1) {
			g[u][1] = a[pa];
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

	dfs2(1, 0);

	for (int i = 1; i <= n; i++) {
		int ans = a[i];
		for (int j = 1; j <= k; j++) {
			ans += f[i][j] + g[i][j];
		}
		cout << ans << endl;
	}
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



### [CF708C Centroids](https://www.luogu.com.cn/problem/CF708C)

【题意】

给 $n$ 元树，定义改造树：删去一条边，再加入一条边，保证改造后还是一棵树。求有多少点可以通过改造树，成为这颗树的重心？

注：如果以某个点为根，每个子树的大小都不大于 $\dfrac{n}{2}$，则称某个点为重心。



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

	vector<int> siz(n + 1), wson(n + 1), g(n + 1), ans(n + 1), from(n + 1);
	vector<array<int, 2>> f(n + 1);

	function<void(int, int)> dfs1 = [&](int u, int pa) {
		siz[u] = 1;
		for (int v : adj[u]) {
			if (v == pa) {
				continue;
			}
			dfs1(v, u);
			siz[u] += siz[v];
			int tmp = siz[v] > n / 2 ? f[v][0] : siz[v];
			if (siz[v] > n / 2) {
				wson[u] = v;
			}
			if (f[u][0] < tmp) {
				f[u][1] = f[u][0];
				f[u][0] = tmp;
				from[u] = v;
			} else if (f[u][1] < tmp) {
				f[u][1] = tmp;
			}
		}
		if (n - siz[u] > n / 2) {
			wson[u] = pa;
		}
		if (!wson[u]) {
			ans[u] = 1;
		} else if (wson[u] != pa && siz[wson[u]] - f[wson[u]][0] <= n / 2) {
			ans[u] = 1;
		}
	};

	function<void(int, int)> dfs2 = [&](int u, int pa) {
		if (n - siz[u] <= n / 2) {
			g[u] = n - siz[u];
		} else if (pa) {
			if (u == from[pa]) {
				g[u] = max(g[pa], f[pa][1]);
			} else {
				g[u] = max(g[pa], f[pa][0]);
			}
		}
		if (n - siz[u] > n / 2 && n - siz[u] - g[u] <= n / 2) {
			ans[u] = 1;
		}
		for (int v : adj[u]) {
			if (v != pa) {
				dfs2(v, u);
			}
		}
	};

	dfs1(1, 0);
	dfs2(1, 0);

	for (int i = 1; i <= n; i++) {
		cout << ans[i] << " \n"[i == n];
	}
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





### [[COCI2014-2015#1] Kamp](https://www.luogu.com.cn/problem/P6419)

【题意】

$n$ 元树，有边权。指定 $k$ 个特殊点。
对于树上每个点 $i$，求从点 $i$ 出发经过所有特殊点至少一次的路径的最小值。

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

	vector<vector<PII>> adj(n + 1);
	for (int i = 1; i < n; i++) {
		int u, v, w;
		cin >> u >> v >> w;
		adj[u].push_back({v, w});
		adj[v].push_back({u, w});
	}

	vector<int> sum(n + 1), vis(n + 1);
	for (int i = 1; i <= k; i++) {
		int x;
		cin >> x;
		vis[x] = sum[x] = 1;
	}

	vector<int> ans(n + 1), g(n + 1);
	vector<array<int, 2>> f(n + 1);
	vector<int> from(n + 1);

	function<void(int, int)> dfs1 = [&](int u, int pa) {
		for (auto [v, w] : adj[u]) {
			if (v == pa) {
				continue;
			}
			dfs1(v, u);
			sum[u] += sum[v];
			if (sum[v]) {
				ans[u] += ans[v] + 2 * w;
				int now = f[v][0] + w;
				if (now >= f[u][0]) {
					f[u][1] = f[u][0];
					f[u][0] = now;
					from[u] = v;
				} else if (now > f[u][1]) {
					f[u][1] = now;
				}
			}
		}
	};

	function<void(int, int, int)> dfs2 = [&](int u, int pa, int wfrom) {
		if (sum[u] == k) {
			g[u] = 0;
		} else {
			g[u] = g[pa] + wfrom;
			if (u == from[pa]) {
				g[u] = max(g[u], f[pa][1] + wfrom);
			} else {
				g[u] = max(g[u], f[pa][0] + wfrom);
			}
		}

		if (u != 1) {
			if (sum[u] == 0) {
				ans[u] = ans[pa] + wfrom * 2;
			} else if (sum[u] != k) {
				ans[u] = ans[pa];
			} else {
				ans[u] = ans[pa] - wfrom * 2;
			}
		}
		for (auto [v, w] : adj[u]) {
			if (v == pa) {
				continue;
			}
			dfs2(v, u, w);
		}
	};

	dfs1(1, 0);
	dfs2(1, 0, 0);
	for (int i = 1; i <= n; i++) {
		cout << ans[i] - max(f[i][0], g[i]) << endl;
	}
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



### [CF855C -  Helga Hufflepuff's Cup](https://codeforces.com/problemset/problem/855/C)



