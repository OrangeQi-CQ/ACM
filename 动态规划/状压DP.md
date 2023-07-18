# 综述

状压DP总体分为两类

- 基于连通性（棋盘）
- 集合的划分



### 连通性状压

##### [Acwing1066 - 小国王](https://www.acwing.com/problem/content/1066/)

【题意】

在 $n×n$ 的棋盘上放 $k$ 个国王，国王可攻击相邻的 $8$ 个格子，求使它们无法互相攻击的方案总数。



【思路】

预处理，一行的合法摆放称为一个状态

- 所有状态，包括二进制表示，用掉国王的数量；
- 每个状态的所有合法邻接状态

状态表示

- 设 $f[i][j][k]$ 表示：考虑前 $i$ 行，前 $i$ 行用掉了 $j$ 个国王，第 $i$ 行的状态为 $k$ 的方案数量。

状态转移

- 枚举每一行 $i$，枚举已经用过的国王 $j$，枚举第 $i - 1$ 行的状态 $k$，枚举第 $i$ 行的可能状态 $b$ （所有可以和 $k$ 相邻的状态）。$c$ 是状态 $k$ 中的国王数量。
- 转移方程： $f[i][j][k] \leftarrow f[i][j][k]  + f[i-1][j-c][b];$



【代码】

```cpp
const int N = 2000;
int n, m, cnt = 0;
int f[20][105][N];

struct State {
    int s, num;
} state[N];

vector<int> nxt[N];

int popcnt(int s) {
    int res = 0;

    while (s) {
        res += s & 1;
        s >>= 1;
    }

    return res;
}

void init() {
    // 预处理所有的状态
    for (int s = 0; s < (1 << n); s++) {
        if (s & (s << 1) || s & (s >> 1)) {
            continue;
        }

        state[++cnt] = {s, popcnt(s)};
    }

    // 预处理每一个状态的所有相邻状态
    for (int i = 1; i <= cnt; i++) {
        for (int j = i; j <= cnt; j++) {
            int x = state[i].s, y = state[j].s;

            if ((x & y) || (x & (y << 1)) || (x & (y >> 1))) {
                continue;
            }

            nxt[i].push_back(j);
            if (i != j) {
                nxt[j].push_back(i);
            }
        }
    }
}

void SolveTest() {
    cin >> n >> m;

    init();

    f[0][0][1] = 1;
    int res = 0;

    for (int i = 1; i <= n; i++) {
        for (int j = 0; j <= m; j++) {
            for (int k = 1; k <= cnt; k++) {
                for (int b : nxt[k]) {
                    int c = state[k].num;

                    if (j >= c) {
                        f[i][j][k] += f[i - 1][j - c][b];
                    }
                }

                if (i == n && j == m) {
                    res += f[n][m][k];
                }
            }
        }
    }

    printf("%lld", res);
}

```



##### [Acwing329 - 玉米田](https://www.acwing.com/problem/content/329/)

【题意】

在$n\times n$的棋盘上放棋子，给定某些位置不能放置，并且所有棋子不能相邻（有公共边），求方案总数



【思路】

预处理

- 一行的可能状态，包括二进制 $s$，以及二进制中 $1$ 的数量
- 每个状态的所有相邻状态

 

【代码】

```cpp
const int N = 15, M = 1e4 + 7, MOD = 1e8;

int n, m, cnt = 0;
int mp[N][N], f[N][M];

struct State {
    int s, num;
} state[M];

vector<int> nxt[M];


int popcnt(int x) {
    int res = 0;

    while (x) {
        res += x & 1;
        x /= 2;
    }

    return res;
}

void init() {
    for (int s = 0; s < (1 << m); s++) {
        if ((s & (s << 1)) or (s & (s >> 1))) {
            continue;
        }

        state[++cnt] = {s, popcnt(s)};
    }

    // printf("cnt == %lld\n", cnt);

    for (int i = 1; i <= cnt; i++) {
        for (int j = i; j <= cnt; j++) {
            if (state[i].s & state[j].s) {
                continue;
            }

            nxt[i].push_back(j);

            if (i != j) {
                nxt[j].push_back(i);
            }
        }
    }
}

// 检查 state[x] 和第 y 行能否匹配
bool check(int x, int y) {
    for (int i = 0; i < m; i++) {
        if (mp[y][m - i] == 0 and ((state[x].s >> i) & 1)) {
            return false;
        }
    }

    return true;
}

void SolveTest() {
    scanf("%lld%lld", &n, &m);

    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            scanf("%lld", &mp[i][j]);
        }
    }

    init();

    int res = 0;
    f[0][1] = 1;

    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= cnt; j++) {
            if (check(j, i) == false) {
                continue;
            }

            for (int t : nxt[j]) {
                if (check(t, i - 1) == false) {
                    continue;
                }

                f[i][j] = (f[i][j] + f[i - 1][t]) % MOD;
            }

            if (i == n) {
                res  = (res + f[n][j]) % MOD;
            }
        }
    }

    printf("%lld", res);

}
```



##### [Acwing294 - 炮兵阵地](https://www.acwing.com/problem/content/294/)

【题意】

在 $n\times n$ 的棋盘上放棋子，有以下约束：

- 给定某些位置不能放置棋子
- 每个棋子上下左右距离 $2$ 以内的位置不能有其他棋子

求最多能摆放棋子的个数。



【分析】

同前两题类似，略。



【代码】

```cpp
const int N = 15, M = 505;
int n, m, cnt = 0;
int mp[105][N];
int f[2][M][M];


struct State {
    int s, num;
} state[10007];

vector<int> nxt[500];


int popcnt(int x) {
    int res = 0;

    while (x) {
        res += x & 1;
        x /= 2;
    }

    return res;
}


void init() {
    for (int s = 0; s < (1 << m); s++) {
        if ((s & (s << 1)) or (s & (s << 2))) {
            continue;
        }

        state[++cnt] = {s, popcnt(s)};
    }

    for (int i = 1; i <= cnt; i++) {
        for (int j = 1; j <= cnt; j++) {
            if (state[i].s & state[j].s) {
                continue;
            }

            nxt[i].push_back(j);

            if (i != j) {
                nxt[j].push_back(i);
            }
        }
    }
}

bool check(int x, int y) {
    if (x < 0) {
        return true;
    }

    for (int i = 0; i < m; i++) {
        if (mp[x][m - i] == 1 and (state[y].s >> i) & 1) {
            return false;
        }
    }

    return true;
}


void SolveTest() {
    scanf("%lld%lld", &n, &m);

    for (int i = 1; i <= n; i++) {
        string s;
        cin >> s;

        for (int j = 0; j < m; j++) {
            if (s[j] == 'H') {
                mp[i][j + 1] = 1;
            }
        }
    }

    init();
    f[0][1][1] = 0;
    int res = 0;

    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= cnt; j++) {
            if (check(i, j) == false) {
                continue;
            }

            for (int k : nxt[j]) {
                // 加上这个判断就TLE了，但是不明白为什么可以删掉
                // if (check(i - 1, k) == false) {
                //     continue;
                // }

                for (int t : nxt[k]) {
                    // 同上，加上这个判断就TLE了，但是不明白为什么可以删掉
                    // if (check(i - 2, t) == false) {
                    //     continue;
                    // }

                    if (state[j].s & state[t].s) {
                        continue;
                    }

                    f[i % 2][j][k] = max(f[i % 2][j][k], f[(i - 1) % 2][k][t] + state[j].num);
                }

                if (i == n) {
                    res = max(res, f[n % 2][j][k]);
                }
            }
        }
    }

    printf("%lld", res);

}
```





##### [Acwing293 - 蒙德里安的梦想](https://www.acwing.com/problem/content/293/)

【题意】

$n \times m$ 的棋盘可以摆放不同的 $1 \times 2$ 小方格的方案数。





### 集合划分状压









### 图论状压