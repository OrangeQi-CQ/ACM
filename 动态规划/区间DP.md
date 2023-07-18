

### 区间DP

##### 字符串编辑距离

https://www.acwing.com/problem/content/904/

【题意】

给定两个字符串 $A,B$，每次操作可以将 $A$ 中的某个字符删除、或插入某个字符、或替换某个字符。

求把 $A$ 变成 $B$ 的最少操作次数。

【分析】

设 $f[i][j]$：$A$ 的前 $i$ 个字符编辑成 $B$ 的前 $j$ 个字符的最少操作次数。

- $f[i][j]\gets min\{f[i-1][j]+1,\ f[i][j-1]+1\}$

- 如果 $a[i]=b[j]$：$ f[i][j]\gets min\{f[i][j], \ f[i-1][j-1]\}$

- 否则：$f[i][j]\gets min\{f[i][j],f[i-1][j-1]+1\}$



```c++
int lena, lenb, f[N][N];
char a[N], b[N];

void SolveTest() {
    //输入
    scanf("%lld%s", &lena, a + 1);
    scanf("%lld%s", &lenb, b + 1);

    //初始化
    for (int i = 1; i <= lena; i++)
        for (int j = 1; j <= lenb; j++) {
            f[i][j] = INF;
        }

    for (int i = 1; i <= lena; i++) {
        f[i][0] = i;
    }

    for (int i = 1; i <= lenb; i++) {
        f[0][i] = i;
    }

    //暴力dp
    for (int i = 1; i <= lena; i++) {
        for (int j = 1; j <= lenb; j++) {
            f[i][j] = min(f[i - 1][j] + 1, f[i][j - 1] + 1);

            if (a[i] == b[j]) {
                f[i][j] = min(f[i][j], f[i - 1][j - 1]);
            } else {
                f[i][j] = min(f[i][j], f[i - 1][j - 1] + 1);
            }
        }
    }

    printf("%lld\n", f[lena][lenb]);
    return 0;
}
```





##### 最大连续区间和

[最大子串和 - 牛客竞赛](https://ac.nowcoder.com/acm/problem/235948)

【题意】
给序列 $a[n]$，求最大连续区间和。



【分析】
设$f[i]$：以 $a[i]$ 为结尾的区间的和的最大值

- $f[i]=a[i]+max\{0,f[i-1]\}$

```cpp
void solve(){
    for(int i = 1; i <= n; i++){
        f[i] = max(f[i-1] + a[i], a[i]);
    }
}
```





2. 扩展问题

https://codeforces.com/contest/1644/problem/C

