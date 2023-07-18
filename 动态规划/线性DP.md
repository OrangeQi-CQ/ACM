

### 矩阵路径模型

##### [P7074 -  方格取数 ](https://www.luogu.com.cn/problem/P7074)

【题意】

给一个 $r\times c$ 矩阵，求从左上角走到右下角最短路径（每一步只能向下或向右）的最大路径和

本题可以扩展至矩阵可为负数，或者求最小路径和



【思路】

- 状态表示

集合：定义`dp[i][j]`为从(1, 1)到达(i, j)的所有方案

属性：最大值

- 状态转移

$dp[i][j]=max\left \{  dp[i][j-1]+w[i][j],\ \ \ dp[i-1][j]+w[i][j]\right \} $

```c++
if(i>1) dp[i][j]=max(dp[i][j],dp[i-1][j]+w[i][j]);
if(j>1) dp[i][j]=max(dp[i][j],dp[i][j-1]+w[i][j]);
```



【朴素做法】

```c++
#include<iostream>
using namespace std;
int w[104][104],dp[1004][1004];
int r,c;


int main(){
    int T;cin>>T;
    while(T--){
        cin>>r>>c;
        for(int i=1;i<=r;i++){
            for(int j=1;j<=c;j++){
                cin>>w[i][j];
                dp[i][j]=0;
            }
        }
        
        dp[1][1]=w[1][1];
        for(int i=1;i<=r;i++){
            for(int j=1;j<=c;j++){
                if(i>1) dp[i][j]=max(dp[i][j],dp[i-1][j]+w[i][j]);
                if(j>1) dp[i][j]=max(dp[i][j],dp[i][j-1]+w[i][j]);
            }
        }
        
        cout<<dp[r][c]<<'\n';
    }
}
```



【滚动数组优化】

```c++
#include<cstring>
#include<iostream>
using namespace std;

const int N = 105;
int w[2][N], dp[2][N], T, n, m;

int main(){
    cin >> T;
    while(T--){
        cin >> r >> c;

        for(int i = 1; i <= n; i++){
            for(int j = 1; j <= m; j++){
                cin >> w[i&1][j];
                dp[i&1][j] = max(dp[i&1][j-1], dp[(i-1)&1][j]) + w[i&1][j];
            }
        }
        cout << dp[n&1][m] << endl;

        memset(dp, 0, sizeof dp);
    }
}

```





##### [P1004 - 方格取数 ](https://www.luogu.com.cn/problem/P1004)

【题意】

给定$n\times n$非负矩阵，寻找两条从左上角到右下角的路径，相交的位置只计算一次，求两条路径和的最大值







##### [P1006 - 传纸条 ](https://www.luogu.com.cn/problem/P1006)









### 最长上升子序列模型



##### LIS 模板题

[Acwing898 - LIS 模板题](https://www.acwing.com/problem/content/898/)



【朴素算法】

`f[i]`表示以`a[i]`结尾的最长上升子序列长度

$f[i]=max\{ f[j] +1\ |\  j\le i\ \and\ a[j]\le a[i]\}$



【树状数组优化】

原理同朴素算法

`f[i]`表示以`a[i]`结尾的最长上升子序列长度

$f[i]=max\{ f[j] +1\ \ \  |\ \ j\le i\ and\ a[j]\le a[i]\}$

```c++
struct BIT_MAX {
    int tr[N];

    void add(int x, int k) {
        for (; x < N; x += x & -x) {
            tr[x] = max(tr[x], k);
        }
    }

    int query(int x) {
        int ans = 0;

        for (; x; x -= x & -x) {
            ans = max(ans, tr[x]);
        }

        return ans;
    }
} bit1;


void SolveTest() {
    scanf("%lld", &n);
    
    for (int i = 1; i <= n; i++) {
        scanf("%lld", &a[i]);
    }

    for (int i = 1; i <= n; i++) {
        f[i] = bit1.query(a[i] - 1) + 1;
        bit1.add(a[i], f[i]);
    }

    printf("%lld", f[n]);
}
```







【二分优化】

$f[i]$ 表示长度为 $i$ 的上升子序列，最后一个元素的最小值。

```cpp
const int N = 1e5 + 7;

int n, a[N], f[N];

void SolveTest() {
    scanf("%lld", &n);
    
    for (int i = 1; i <= n; i++) {
        scanf("%lld", &a[i]);
    }

    fill(f, f + 1 + n, 1e9);
    int len = 1;

    for (int i = 1; i <= n; i++) {
        if (a[i] <= f[len]) {
            int p = lower_bound(f + 1, f + 1 + len, a[i]) - f;
            f[p] = a[i];
        } else {
            f[++len] = a[i];
        }
    }

    printf("%lld", len);
}
```



##### Dilworth定理

对于任意有限偏序集，其最大反链中元素的数目必等于最小链划分中链的数目

换言之，把一个序列划分为最少的下降子序列的个数 $\Leftrightarrow $ 求该序列最长不降子序列长度





### 最长公共子序列(LCS)

【普通序列】

$Solution:$

$f[i][j]$ ：$a$ 的前 $i$ 个数字和 $b$ 的前 $j$ 个数字的最长公共子序列长度

- 如果 $a[i]=b[j]$ ：$f[i][j]\gets f[i-1][j-1]+1$

- 否则：$f[i][j]\gets max\{f[i-1][j],f[i][j-1\}$

```c++
int f[N][N], n, m;
string a, b;

void SolveTest() {
    //输入
    scanf("%lld %lld", &n, &m);
    cin >> a >> b;
    a = " " + a;
    b = " " + b;

    //暴力dp
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            if (a[i] == b[j]) {
                f[i][j] = f[i - 1][j - 1] + 1;
            } else {
                f[i][j] = max(f[i - 1][j], f[i][j - 1]);
            }
        }
    }

    printf("%lld", f[n][m]);
}

```





【两个序列都是排列】

$Solution:$

将 $b[]$ 构建为 $c[]$ ，使得若 $b[i]=a[j]$，则$c[i]=j$

即 $c[i]$ 是 $b[i]$ 在 $a[]$ 中出现的位置下标

则：$ab$ 的最长公共子序列 $\Leftrightarrow$ $c$ 的最大上升子序列。









### 状态机模型

##### [Acwing1051 - 大盗阿福 ](https://www.acwing.com/problem/content/1051/)

【题意】

给序列 $a$，选择若干两两不相邻的数字，求被选择数字的最大总和。



【分析】

状态表示

- 设 $f[i][0]$ 表示：考虑前 $i$ 个数字，不选择第 $i$ 个数字的最大总和；
- 设 $f[i][1]$ 表示：考虑前 $i$ 个数字，选择第 $i$ 个数字的最大总和； 

状态转移

- $f[i][0] = \max(f[i - 1][0], f[i - 1][1])$；
- $f[i][1] = f[i - 1][0] + a[i]$；

答案即为 $\max(f[n][0], f[n][1])$。



【代码】

```cpp
const int N = 2e5 + 7;
int n, a[N], f[N][2];

void SolveTest() {
    scanf("%lld", &n);
    
    for (int i = 1; i <= n; i++) {
        scanf("%lld", &a[i]);
        f[i][0] = f[i][1] = 0;
    }

    for (int i = 1; i <= n; i++) {
        f[i][1] = f[i - 1][0] + a[i];
        f[i][0] = max(f[i - 1][0], f[i - 1][1]);
    }

    printf("%lld\n", max(f[n][0], f[n][1]));
}

```



##### [Acwing1057 - 买卖股票IV](https://www.acwing.com/problem/content/1059/)

【题意】

给定一个长度为 $n$ 的数组 $a$， $a[i]$ 表示一个股票在第 $i$ 天的价格。一次买入卖出合为一笔交易，最多可以完成 $k$ 笔交易。（必须在再次购买前出售掉之前的股票）。

求能获取的最大利润。

$(1\le n \le 10^5,\ 1 \le k \le 100)$



【分析】

状态表示：

- 设 $f[i][j][0/1]$ 表示：前 $i$ 天，进行了 $j$ 次交易，当天结束后手里 没有 / 有 股票的最高利润。

状态转移：

- $f[i][j][0] = \max(f[i - 1][j][0], f[i - 1][j][1] + a[i])$；
- $f[i][j][1] = \max(f[i - 1][j][1], f[i - 1][j - 1][0] - a[i])$；

答案就是 $\max_{i = 1} ^ {k} f[n][i][0]$。



【代码】

```cpp
void SolveTest() {
    int n, k;
    cin >> n >> k;

    for (int i = 1; i <= n; i++) {
        scanf("%lld", &a[i]);
    }

    memset(f, -0x3f, sizeof f);

    for (int i = 0; i <= n; i++) {
        f[i][0][0] = 0;
    }

    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= k; j++) {
            f[i][j][0] = max(f[i - 1][j][0], f[i - 1][j][1] + a[i]);
            f[i][j][1] = max(f[i - 1][j][1], f[i - 1][j - 1][0] - a[i]);
        }
    }

    int ans = 0;

    for (int i = 0; i <= k; i++) {
        ans = max(ans, f[n][i][0]);
    }

    cout << ans << endl;
}
```





##### [Acwing1060 - 买卖股票V](https://www.acwing.com/problem/content/1060/)

【题意】

给定一个长度为 $n$ 的数组 $a$，$a[i]$ 表示股票在第 $i$ 天的价格。有以下限制：

- 必须在再次购买前出售掉之前的股票；
- 卖出股票后，无法在第二天买入股票 (即冷冻期为 $1$ 天)。

求 $n$ 天的最大利润。

$n \le 10^5$。

【分析】

状态表示：

- 设$f[i][0]$ 表示：考虑前 $i$ 天，当天没有股票，且不处于冷冻期 （空仓）的最大利润
- 设$f[i][1]$ 表示：考虑前 $i$ 天，当天有股票 （持仓）的最大利润
- 设$f[i][2]$ 表示：考虑前 $i$ 天，当天没有股票，且处于冷冻期 （冷冻期）的最大利润

状态转移

- $f[i][0] = \max(f[i - 1][0], f[i - 1][2])$；
- $f[i][1] = \max (f[i - 1][1], f[i - 1][0] - a[i])$；
- $f[i][2] = \max(f[i - 1][1] + a[i])$；

答案即为 $\max(f[n][0], f[n][2])$。



【代码】

```cpp
const int N = 2e5 + 7;
int n, a[N], f[N][3];

void SolveTest() {
    cin >> n;

    for (int i = 1; i <= n; ++ i) {
        scanf("%lld", &a[i]);
    }

    memset(f, -0x3f, sizeof f);
    f[0][0] = 0;

    for (int i = 1; i <= n; ++ i) {
        f[i][0] = max(f[i - 1][0], f[i - 1][2]);
        f[i][1] = max(f[i - 1][1], f[i - 1][0] - a[i]);
        f[i][2] = f[i - 1][1] + a[i];
    }

    int ans = max(f[n][0], f[n][2]);
    printf("%lld\n", ans);
}
```



##### [Acwing1054 - 设计密码](https://www.acwing.com/problem/content/1054/)


给定整数 $n$，字符串 $T$，求满足一下要求的字符串 $S$ 的个数：

- $S$ 的长度是 $n$；
- $S$ 只包含小写英文字母；
- $S$ 不包含子串 $T$；

$(1\le |T| \le n \le 50)$，答案对 $10^9 + 7$ 取整。







