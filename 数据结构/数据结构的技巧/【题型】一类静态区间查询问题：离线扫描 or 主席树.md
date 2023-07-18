整理一类数据结构题的做法。

题目特点是，对于一个序列的多次查询，允许离线。这种问题潜在的方向有很多：用各种数据结构在线做，也可以尝试离线莫队。

这里整理一类很常见的做法：

- 将所有询问的区间按照右端点排序。可以用 `std::vector<std::vector<Query>> qu` 记录以每个点为右边界的询问。
- 从左向右扫描序列，用数据结构维护答案。



## [[SDOI2009] HH的项链](https://www.luogu.com.cn/problem/P1972) 

【题意】

给一个序列，多次查询某个区间内不同的数字的个数。

```cpp
#include <bits/stdc++.h>
#define int long long

const int N = 1e6 + 7;

struct BIT {
    int n;
    std::vector<int> a;
    BIT (int _n): n(_n), a(n + 1) {}
    void add(int x, int k) {
        for (int i = x; i <= n; i += i & -i) {
            a[i] += k;
        }
    }
    int query(int x) {
        int res = 0;
        for (int i = x; i; i -= i & -i) {
            res += a[i];
        }
        return res;
    }
};

struct Query {
    int id, l;
};

signed main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    std::cout.tie(0);
    
    int n, m;
    std::cin >> n;
    std::vector<int> a(n + 1);
    for (int i = 1; i <= n; i++) {
        std::cin >> a[i];
    }

    std::cin >> m;
    std::vector<std::vector<Query>> qu(n + 1);
    std::vector<int> ans(m + 1);
    BIT bit(n);

    for (int i = 1; i <= m; i++) {
        int l, r;
        std::cin >> l >> r;
        qu[r].push_back({i, l});
    }

    std::vector<int> lastpos(N + 1);    // 记录每种颜色上一次出现的位置
    for (int i = 1; i <= n; i++) {
        if (lastpos[a[i]]) {
            bit.add(lastpos[a[i]], -1);
        }
        bit.add(i, 1);
        lastpos[a[i]] = i;
        for (auto [id, lpos] : qu[i]) {
            ans[id] = bit.query(i) - bit.query(lpos - 1);
        }
    }
    for (int i = 1; i <= m; i++) {
        std::cout << ans[i] << "\n";
    }

    return 0;
}
```



## [[HEOI2012]采花](https://www.luogu.com.cn/problem/P4113) 

【题意】

给一个序列，多次查询某个区间内有多少个数字出现了不少于两次。

```cpp
#include <bits/stdc++.h>
#define int long long

struct BIT {
    int n;
    std::vector<int> a;
    BIT(int _n) : n(_n), a(n + 1) {};
    void add(int x, int k) {
        for (int i = x; i <= n; i += i & -i) {
            a[i] += k;
        }
    }
    int query(int x) {
        int res = 0;
        for (int i = x; i; i -= i & -i) {
            res += a[i];
        }
        return res;
    }
};

struct Query {
    int id, l;
};

signed main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    std::cout.tie(0);

    int n, c, m;
    std::cin >> n >> c >> m;
    std::vector<int> a(n + 1);
    for (int i = 1; i <= n; i++) {
        std::cin >> a[i];
    }

    std::vector<std::vector<Query>> qu(n + 1);
    std::vector<int> ans(m + 1);
    for (int i = 1; i <= m; i++) {
        int l, r;
        std::cin >> l >> r;
        qu[r].push_back({i, l});
    }

    // last[i] 记录最近的 j < i，满足 a[j] == a[i]
    std::vector<int> last(n + 1), pos(c + 1);
    BIT bit(n + 1);
    for (int i = 1; i <= n; i++) {
        last[i] = pos[a[i]];
        pos[a[i]] = i;
    }

    for (int i = 1; i <= n; i++) {
        if (last[i]) {
            bit.add(last[i], 1);
            if (last[last[i]]) {
                bit.add(last[last[i]], -1);
            }
        }
        for (auto [id, lpos] : qu[i]) {
            ans[id] = bit.query(i) - bit.query(lpos - 1);
        }
    }

    for (int i = 1; i <= m; i++) {
        std::cout << ans[i] << "\n";
    }
    return 0;
}
```



## [CF522D. Closest Equals](http://codeforces.com/problemset/problem/522/D)

【题意】

现在有一个序列 $a_1, a_2, ..., a_n$ ，还有$m$个查询 $l_j, r_j$ $(1 ≤ l_j ≤ r_j ≤n)$  。对于每一个查询，请找出距离最近的两个元素 $a_x$ 和 $a_y$$ (x ≠ y)$ ，并且满足以下条件：

- $l_j ≤ x, y ≤ r_j$; 

- $a_x = a_y$。 


【思路】

将询问按照右端点排序，从左向右扫描更新答案。

$pre[i]$ 记录位置 $i$ 左边最靠右的位置，满足 $a[pre[i]] = a[i]$。

在从左向右扫描的时候，用线段树维护另一个序列 $t$：$t[i]$ 记录当前最小的距离满足 $a[i+t[i]] = a[i]$。由于我们是从左向右扫描的，所以这个 $i+t[i]$ 一定不会超出询问的范围。所以就可以用线段树维护 $t$，每次单点修改，查询区间最小值即可。

```cpp
#include <bits/stdc++.h>
#define int long long

const int INF = 1e18;
#define ls(p) (p * 2)
#define rs(p) (p * 2 + 1)

// 单点修改，查询区间最小值
struct Segtree {
    std::vector<int> minv;
    Segtree(int n) {
        minv.assign(n * 4, INF);
    }
    void pushup(int p) {
        minv[p] = std::min(minv[ls(p)], minv[rs(p)]);
    }
    void modify(int p, int l, int r, int x, int k) {
        if (l == x && r == x) {
            minv[p] = k;
            return;
        }
        int mid = (l + r) >> 1;
        if (x <= mid) {
            modify(ls(p), l, mid, x, k);
        } else {
            modify(rs(p), mid + 1, r, x, k);
        }
        pushup(p);
    }
    int queryMin(int p, int l, int r, int x, int y) {
        if (x <= l && r <= y) {
            return minv[p];
        }
        int mid = (l + r) >> 1, res = INF;
        if (x <= mid) {
            res = std::min(res, queryMin(ls(p), l, mid, x, y));
        }
        if (y > mid) {
            res = std::min(res, queryMin(rs(p), mid + 1, r, x, y));
        }
        return res;
    }
};

struct Query {
    int id, l;
};

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
    auto tmpa = a;
    std::sort(tmpa.begin() + 1, tmpa.end());
    tmpa.erase(unique(tmpa.begin(), tmpa.end()), tmpa.end());
    for (int i = 1; i <= n; i++) {
        a[i] = std::lower_bound(tmpa.begin(), tmpa.end(), a[i]) - tmpa.begin();
    }

    std::vector<std::vector<Query>> qu(n + 1);
    std::vector<int> ans(m + 1);
    for (int i = 1; i <= m; i++) {
        int l, r;
        std::cin >> l >> r;
        qu[r].push_back({i, l});
    }

    // pre[i] 维护上一个与 a[i] 相等的位置
    std::vector<int> pre(n + 1), pos(n + 1);
    for (int i = 1; i <= n; i++) {
        pre[i] = pos[a[i]];
        pos[a[i]] = i;
    }
    // 这个线段树维护的实际上是：每个点右边最近的与它相等的点的距离
    Segtree tr(n + 1);

    for (int i = 1; i <= n; i++) {
        if (pre[i]) {
            tr.modify(1, 1, n, pre[i], i - pre[i]);
        }
        for (auto [id, lpos] : qu[i]) {
            ans[id] = tr.queryMin(1, 1, n, lpos, i);
            if (ans[id] == INF) {
                ans[id] = -1;
            }
        }
    }

    for (int i = 1; i <= m; i++) {
        std::cout << ans[i] << "\n";
    }
    return 0;
}
```





## [CF594D. REQ](https://codeforces.com/problemset/problem/594/D)

【题意】

给定序列，多次询问：

- 区间 $[l,r]$ 求 $\varphi(\prod_{i=l}^r a_i) \bmod 10^9+7$。

【思路】

根据上面的套路，离线 + 扫描线。

根据欧拉函数的计算公式，答案可以分成两部分的乘积：

- 区间乘积

- 枚举所有质数，质数 $p$ 对答案的贡献为 $\dfrac{p-1}{p}$。
    - 每个质数只在最后一次出现的位置对答案做出贡献，所以在扫描的过程中动态地维护每个质数最后一次出现的位置下标。
    - 扫描的时候将 $a_i$ 分解质因数，对 $a_i$ 的所有质因数在它们最后一次出现的位置取消贡献，在 $i$ 处加入它们的贡献，并更新最后一次出现的位置为 $i$。

```cpp
#include <bits/stdc++.h>
#define int long long

const int MOD = 1e9 + 7;

template<class T>
T qpow(T x, int k) {
    T res = 1;
    while (k) {
        if (k % 2) {
            res = res * x;
        }
        x = x * x;
        k /= 2;
    }
    return res;
}

int norm(int x) {
    return (x % MOD + MOD) % MOD;
}

struct mint {
    int x;
    int val() const {
        return x;
    }
    mint(int x = 0) : x(norm(x)) {}
    mint operator-() const {
        return mint(norm(MOD - x));
    }
    mint inv() const {
        assert(x != 0);
        return qpow(*this, MOD - 2);
    }
    mint &operator*=(const mint &rhs) {
        x = x * rhs.x % MOD;
        return *this;
    }
    mint &operator+=(const mint &rhs) {
        x = norm(x + rhs.x);
        return *this;
    }
    mint &operator-=(const mint &rhs) {
        x = norm(x - rhs.x);
        return *this;
    }
    mint &operator/=(const mint &rhs) {
        return *this *= rhs.inv();
    }
    friend mint operator*(const mint &lhs, const mint &rhs) {
        mint res = lhs;
        res *= rhs;
        return res;
    }
    friend mint operator+(const mint &lhs, const mint &rhs) {
        mint res = lhs;
        res += rhs;
        return res;
    }
    friend mint operator-(const mint &lhs, const mint &rhs) {
        mint res = lhs;
        res -= rhs;
        return res;
    }
    friend mint operator/(const mint &lhs, const mint &rhs) {
        mint res = lhs;
        res /= rhs;
        return res;
    }
};

const int MAXN = 1e6 + 7;
int pcnt = 0, primes[MAXN + 10];
std::bitset<MAXN + 10> isp;

void init() {
    isp.set();
    pcnt = isp[0] = isp[1] = 0;
    for (int i = 2; i <= MAXN; i++) {
        if (isp[i]) {
            primes[++pcnt] = i;
        }
        for (int j = 1; j <= pcnt && i * primes[j] <= MAXN; j++) {
            int x = primes[j];
            isp[i * x] = 0;
            if (i % x == 0) {
                break;
            }
        }
    }
}

template<class T>
struct BIT {
    int n;
    std::vector<T> tr;
    BIT(int _n) {
        n = _n;
        tr.assign(n + 1, 1);
    }
    void add(int x, T k) {
        for (int i = x; i <= n; i += i & -i) {
            tr[i] = tr[i] * k;
        }
    }
    T query(int x) {
        T res = 1;
        for (int i = x; i; i -= i & -i) {
            res = res * tr[i];
        }
        return res;
    }
};

struct Query {
    int id, l, r;
};

signed main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    std::cout.tie(0);

    init();

    int n, q;
    std::cin >> n;
    std::vector<mint> a(n + 1), pre(n + 1, 1);
    for (int i = 1; i <= n; i++) {
        int x;
        std::cin >> x;
        a[i] = x;
        pre[i] = pre[i - 1] * x;
    }
    std::cin >> q;
    std::vector<std::vector<Query>> qu(n + 1);
    std::vector<int> ans(q + 1);
    for (int i = 1; i <= q; i++) {
        int l, r;
        std::cin >> l >> r;
        qu[r].push_back({i, l, r});
    }

    BIT<mint> bit(n + 1);
    std::vector<int> last(MAXN + 1);

    for (int i = 1; i <= n; i++) {
        std::vector<int> tmp;
        int t = a[i].x;
        for (int j = 1; j <= pcnt; j++) {
            int x = primes[j];
            if (x * x > t) {
                break;
            }
            if (t % x == 0) {
                tmp.push_back(x);
            }
            while (t % x == 0) {
                t /= x;
            }
        }
        if (t > 1) {
            tmp.push_back(t);
        }
        for (int x : tmp) {
            mint val = mint(x - 1) / mint(x);
            if (last[x]) {
                bit.add(last[x], val.inv());
            }
            last[x] = i;
            bit.add(i, val);
        }

        for (auto [id, ql, qr] : qu[i]) {
            mint res = (mint)pre[i] / (mint)pre[ql - 1] * bit.query(i) / bit.query(ql - 1);
            ans[id] = res.x;
        }
    }
    for (int i = 1; i <= q; i++) {
        std::cout << ans[i] << "\n";
    }

    return 0;
}
```



## [CF1000F. One Occurrence](https://codeforces.com/problemset/problem/1000/F)

【题意】

给定一个长度为 $n$ 序列 $a$，$m$ 个询问，每次询问：

- 给定一个区间 $[l,r]$，输出这个区间内只出现一次的数当中的任意一个，没有就输出 $0$。

【思路】

（待做。用莫队套值域分块很好写，但是还没理解题解区里面线段树的做法）。



## [CF1422F. Boring Queries](http://codeforces.com/problemset/problem/1422/F)

【题意】

给一个序列 $(a_i \le 2\times 10^5)$。

- 多次查询某区间 $\operatorname{lcm}$。

强制在线，答案对 $10^9+7$ 取模。

【思路】

如果答案不需要取模， $\operatorname{lcm}$ 是可以区间合并的。但是取模的话就有些麻烦，只能用下面方式计算：

> 假设 $\displaystyle a_i=\prod_{j=1}^{k}p_j^{c_{ij}}$，其中 $c_{ij}$ 是 $a_i$ 中质因子 $p_j$ 的出现次数。那么 
> $$
> \operatorname{lcm}(a_1,a_2,...,a_n) = \prod_{j=1}^{k}p_j^{\max(c_{1j}, c_{2j}, ...,c_{nj})}
> $$
> 所以当我们求区间 $\operatorname {lcm}$ 的时候，每个质因子的贡献是区间内次数的最大值，答案是所有质因子贡献的乘积。

先不考虑强制在线的约束。考虑离线扫描，批量求以 $i$ 为右端点的区间的答案。

- 用线段树维护：每个位置（下标）对答案的贡献。查询的时候返回区间乘积。

- 对于当前位置 $a_i$，我们在线段树的下标 $i$ 处插入 $a_i$，表示位置 $i$ 对答案的贡献为 $a_i$。但是这样显然会有重复的贡献。
- 用数组 $s[x]$ 记录上一个能整除 $x$ 的下标。将 $a_i$ 进行质因数分解 $\displaystyle a_i = \prod_{j=1}p_j^{c_{ij}}$。对于所有的 $k=1,2,..., c_{ij}$，我们都在线段树的下标 $s[p_j^{k}]$ 处乘上 $\dfrac{1}{p_j}$，并更新 $s[p_j^k]=i$。

- 这样就可以直接查询所有以 $i$ 为右边界的区间的答案了，在线段树上求区间 $[l,i]$ 乘积就是区间 $[l,i]$ 的 $\operatorname {lcm}$。

至于强制在线的要求：由于每扫描到一个点都是进行若干次单点修改，所以可以可持久化维护线段树的每个版本。

本题空间略微卡常（可能因为本意是根号分治），能不开 `long long` 的地方就不开 `long long`。

```cpp
#include <bits/stdc++.h>
#define LL long long

const int MOD = 1e9 + 7, N = 2e5 + 7;
int f[N], s[N];

LL qpow(LL x, int k) {
    LL res = 1;
    while (k) {
        if (k % 2) {
            res = res * x % MOD;
        }
        k /= 2;
        x = x * x % MOD;
    }
    return res;
}
LL inv(LL x) {
    return qpow(x, MOD - 2);
}

struct Info {
    int prod = 1;
    friend Info operator+ (Info a, Info b) {
        return Info{(int)(1ll * a.prod * b.prod % MOD)};
    }
};

struct PerSegmentTree {
    std::vector<Info> info = {Info()};
    std::vector<int> ls = {0}, rs = {0};
    int nodecnt = 0;
    int clone(int p) {
        info.push_back(info[p]);
        ls.push_back(ls[p]);
        rs.push_back(rs[p]);
        return ++nodecnt;
    }
    int insert(int p, int l, int r, int x, int k) {
        int rt = clone(p);
        info[rt] = info[rt] + Info{k};
        if (l >= r) {
            return rt;
        }
        int mid = (l + r) / 2;
        if (x <= mid) {
            ls[rt] = insert(ls[p], l, mid, x, k);
        } else {
            rs[rt] = insert(rs[p], mid + 1, r, x, k);
        }
        return rt;
    }
    Info query(int p, int l, int r, int x, int y) {
        if (l >= x && r <= y) {
            return info[p];
        }
        int mid = (l + r) / 2;
        Info res;
        if (x <= mid) {
            res = res + query(ls[p], l, mid, x, y);
        }
        if (y > mid) {
            res = res + query(rs[p], mid + 1, r, x, y);
        }
        return res;
    }
};

signed main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    std::cout.tie(0);

    for (int i = 2; i < N; i++) {
        if (!f[i]) {
            for (int j = i; j < N; j += i) {
                f[j] = i;
            }
        }
    }

    int n, q;
    std::cin >> n;
    std::vector<int> a(n + 1), root(n + 1);
    PerSegmentTree tr;
    for (int i = 1, x; i <= n; i++) {
        std::cin >> x;
        root[i] = tr.insert(root[i - 1], 1, n, i, x);
        while (x > 1) {
            int p = f[x], t = 1;
            while (x % p == 0) {
                x /= p;
                t *= p;
                if (s[t]) {
                    root[i] = tr.insert(root[i], 1, n, s[t], inv(p));
                }
                s[t] = i;
            }
        }
    }

    int l, r;
    LL ans = 0;
    std::cin >> q;
    while (q--) {
        std::cin >> l >> r;
        l = (l + ans) % n + 1;
        r = (r + ans) % n + 1;
        if (l > r) {
            std::swap(l, r);
        }
        Info res = tr.query(root[r], 1, n, l, r);
        ans = res.prod;
        std::cout << ans << "\n";
    }

    return 0;
}
```



