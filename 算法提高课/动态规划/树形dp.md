# 树形DP



## 树的最长路径

> https://www.acwing.com/problem/content/1074/

### 思路

#### 题意

![image-20220720172652254](D:/%E7%AC%94%E8%AE%B0/%E7%AE%97%E6%B3%95%E6%8F%90%E9%AB%98%E8%AF%BE/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92/%E6%A0%91%E5%BD%A2dp.assets/image-20220720172652254.png)

由于是无向树，所以可以随意选择某个顶点作为根

以子树的根来作为划分依据，最长路径等于子树的最大路径长度和次最大路径长度之和



### C++ 代码

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 10010, M = 2 * N;

int n;
int h[N], e[M], w[M], ne[M], idx;
int ans;

inline int read(){
    int num = 0;
    char c;
    bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true;
    else num = c - '0';
    while (isdigit(c = getchar()))
    num = num * 10 + c - '0';
    return (flag ? -1 : 1) * num;
}

void add(int a, int b, int c) {
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx ++;
}

int dfs(int u, int fa) {
    int dist = 0;
    int d1 = 0, d2 = 0;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        int d = dfs(j, u) + w[i];
        dist = max(d, dist);
        if (d >= d1) d2 = d1, d1 = d;
        else if (d > d2) d2 = d;
    }
    cout << dist << endl;
    ans = max(ans, d1+d2);
    return dist;
}

int main() {
    memset(h, -1, sizeof h);
    n = read();
    for (int i = 0; i < n-1; i ++) {
        int a, b, c;
        a = read();
        b = read();
        c = read();
        add(a, b, c), add(b, a, c);
    }
    dfs(5, -1);
    printf("%d\n", ans);
    return 0;
}
```



## 树的中心

### 原题链接

> https://www.acwing.com/problem/content/1075/



### 思路

#### 题意

求某一点到树中其他结点的最远距离的最小值

对树的顶点按照上方和下方划分，对于某一个顶点而言，最远距离等于下方的最远距离和上方的最远距离取最大值

树的下方可按照上题方法来做，上方其实也是类似，做两遍 `dfs`



### C++ 代码

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 10010, M = 2 * N, INF = 0x3f3f3f3f;

int n;
int h[N], e[M], ne[M], w[M], idx;
int d1[N], d2[N], up[N], p1[N];
inline int read(){
    int num = 0;
    char c;
    bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true;
    else num = c - '0';
    while (isdigit(c = getchar()))
    num = num * 10 + c - '0';
    return (flag ? -1 : 1) * num;
}

void add(int a, int b, int c) {
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx ++;
}

int dfs_down(int u, int fa) {
    d1[u] = d2[u] = -INF;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        int d = dfs_down(j, u) + w[i];
        if (d >= d1[u]) {
            d2[u] = d1[u], d1[u] = d;
            p1[u] = j;
        }
        else if (d > d2[u]) d2[u] = d;
    }
    if (d1[u] == -INF) d1[u] = d2[u] = 0;
    if (d2[u] == -INF) d2[u] = 0;
    return d1[u];
}

void dfs_up(int u, int fa) {
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        if (j == p1[u]) {
            up[j] = max(d2[u], up[u]) + w[i];
        } else {
            up[j] = max(d1[u], up[u]) + w[i];
        }
        dfs_up(j, u);
    }
}

int main() {
    memset(h, -1, sizeof h);
    n = read();
    for (int i = 0; i < n-1; i ++) {
        int a, b, c;
        a = read(), b = read(), c = read();
        add(a, b, c), add(b, a, c);
    }
    dfs_down(1, -1);
    dfs_up(1, -1);
    int res = INF;
    for (int i = 1; i <= n; i ++) {
        res = min(res, max(d1[i], up[i]));
    }
    printf("%d\n", res);
    return 0;
}
```





## 数字转换


### 原题链接

> https://www.acwing.com/problem/content/1077/



### 思路

#### 题意

通过分析可得，每个数的 $x$ 的约数之和 $y$ 是固定的，也就是每个数只有唯一一个对应的约数之和，因此可以从 $y$ 向 $x$ 连一条有向边，利用这个性质可以按照这样的方式建树，无向边就转化为了有向边(当然无向边也是可以的，只不过需要多开两倍的数组空间)

本质上就是求树中的直径，权值为1



### C++ 代码

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 50010, M = N;

int n;
int sum[N];
int h[N], e[M], ne[M], idx;
bool st[N];
int res;
inline int read(){
    int num = 0;
    char c;
    bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true;
    else num = c - '0';
    while (isdigit(c = getchar()))
    num = num * 10 + c - '0';
    return (flag ? -1 : 1) * num;
}

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

int dfs(int u) {
    int d1 = 0, d2 = 0;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        int d = dfs(j) + 1;
        if (d >= d1) d2 = d1, d1 = d;
        else if (d > d2) d2 = d;
    }
    res = max(res, d1 + d2);
    return d1;
}

int main() {
    memset(h, -1, sizeof h);
    n = read();
    for (int i = 1; i <= n; i ++)
        for (int j = 2; j <= n / i; j ++)
            sum[i * j] += i;
    for (int i = 2; i <= n; i ++)
        if (i > sum[i])
            add(sum[i], i), st[i] = true;
    
    for (int i = 1; i <= n; i ++)
        if (!st[i]) dfs(i);
    printf("%d\n", res);
    return 0;
}  
```



### 代码解析

1. 在求约数时，利用试除法时间复杂度过高，为 $O(N\sqrt{N})$. 可以反过来考虑，对于每个数 $d$，$1\sim N$ 中以 $d$ 为约数的数就是 $d$ 的倍数，$d, 2d, 3d, ..., \lfloor N/d \rfloor*d$ ，时间复杂度为 $O(NlogN)$
2. 另外记录根结点，然后 `dfs` 时从根结点开始，因为生成的并不是完整的一棵树，而是森林，其中以 $1$ 为根结点生成的树最大，而我们的答案就在其中。实测只 `dfs(1)` ， 也能 `AC` ，但并没有严谨的证明对于所有的数据都适用，最好还是遍历所有的树





## 二叉苹果树



### 原题链接

>https://www.acwing.com/problem/content/description/1076/



### 思路

#### 题意分析

苹果的数量对应边的权值

保留边数不超过m的最大价值，是背包问题

再分析可得，对于某个以u为根结点的子树，保留其子树中的边时一定会保留u到该子树的边，每一个子树可以看作是一组，每一个子树中可选择保留k条边，因此是有依赖的背包问题

#### 状态表示

**简化前三维表示**

```
f[u][i][j]表示以u为根结点，考虑前i棵子树保留边数不超过j的所有方案中 权值之和的最大值
```



**简化后二维表示**

```
f[u][j] 表示以u为根结点的子树，所保留边数不超过j(即体积不超过j)的所有方案中 权值之和的最大值
```



#### 状态计算

**简化前三维表示**

```
f[u][i][j] = max(f[u][i][j], {f[u][i-1][j-k-1] + f[son][子树的子树数量][k] + 根结点到该子树的权重})
```



**简化后二维表示**

```
f[u][j] = max(f[i][j], {f[u][j-k-1] + f[son][k] + 根结点到该子树的权重})
```



### C++ 代码



```cpp
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 110, M = 2 * N;
int h[N], e[M], ne[M], w[M], idx;

int f[N][N];
int n, m;
inline int read(){
    int num = 0;
    char c;
    bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true;
    else num = c - '0';
    while (isdigit(c = getchar()))
    num = num * 10 + c - '0';
    return (flag ? -1 : 1) * num;
}

void add(int a, int b, int c) {
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx ++;
}

void dfs(int u, int fa) {
    for (int i = h[u]; ~i; i = ne[i]) {
        int son = e[i];
        if (son == fa) continue;
        dfs(son, u);

        for (int j = m; j; j --) 
            for (int k = 0; k < j; k ++) // 这里需要少算一条边，因为如果保留子树中的边，则一定会保留从根结点到g
                f[u][j] = max(f[u][j], f[u][j-k-1] + f[son][k] + w[i]);
    }
}

int main() {
    memset(h, -1, sizeof h);
    n = read(), m = read();
    for (int i = 0; i < n - 1; i ++) {
        int a = read(), b = read(), c = read();
        add(a, b, c), add(b, a, c);
    }

    dfs(1, -1);
    printf("%d\n", f[1][m]);
    return 0;
}
```





## 战略游戏



### 原题链接



> https://www.acwing.com/problem/content/description/325/



### 思路

#### 

#### 状态表示

```
f[u][j](j = 0, 1) 表示以u为根结点的树，状态为j的所有方案中需要士兵的最小数量
j = 0 表示此根结点不放士兵，j = 1 表示此根结点放士兵
```

#### 状态计算

```cpp
f[u][0] += f[son][1];
f[u][1] += min(f[son][0], f[son][1]); 
```



### C++ 代码

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 1510, M = N;

int n;
int h[N], e[M], ne[M], idx;
int f[N][2];
bool st[N];

inline int read(){
    int num = 0;
    char c;
    bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true;
    else num = c - '0';
    while (isdigit(c = getchar()))
    num = num * 10 + c - '0';
    return (flag ? -1 : 1) * num;
}

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

void dfs(int u) {
    f[u][1] = 1;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        // cout << u << " : " << j << endl;
        dfs(j);
        f[u][0] += f[j][1];
        f[u][1] += min(f[j][0], f[j][1]); 
        // cout << f[u][0] << ' ' << f[u][1] << endl;
    }
}

int main() {
    while(scanf("%d", &n) != EOF) {
        memset(h, -1, sizeof h);
        memset(st, 0, sizeof st);
        memset(f, 0, sizeof f);
        idx = 0;
        int a, b, num;
        for (int i = 0; i < n; i ++) {
            scanf("%d:(%d)", &a, &num);
            while(num --) {
                b = read();
                add(a, b);
                st[b] = true;
            }
        }
        
        int root;
        for (int i = 0; i < n; i ++)
            if (!st[i]) {
                root = i;
                break;
            }
        dfs(root);
        printf("%d\n", min(f[root][0], f[root][1]));
    }
    return 0;
}
```



## 皇宫看守



### 原题链接

> https://www.acwing.com/problem/content/description/1079/



### 思路

本题和上一题类似，主要区别在于，上一题每一个顶点可以看管其直接相连的边，而本题是每一个顶点可以看管其直接相连的边以及顶点，因此每个顶点的状态可以多增加一维



#### 状态表示

```
f[u][0] 表示以u为根结点的树，其根结点的父结点放护卫的方案中代价的最小值
f[u][1] 表示以u为根结点的树，其根结点自己放护卫的方案中代价的最小值
f[u][2] 表示以u为根结点的树，其根结点的子结点放护卫的方案中代价的最小值
```

#### 状态计算

```
f[u][0] = min(f[son][1], f[son][2])
f[u][1] = min(f[son][0], f[son][1], f[son][2])
f[u][2] = min{f[sk][1] + min(f[s1][1], f[s1][2]) + min(f[s1][1], f[s1][2]) + ...}
sk 为选择其中的子结点
```



### C ++代码

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 1510, M = 2*N;

int n;
int w[N];
int h[N], e[M], ne[M], idx;
int f[N][3]; // 0表示父结点放，1表示自己放，2表示子结点放

inline int read(){
    int num = 0;
    char c;
    bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true;
    else num = c - '0';
    while (isdigit(c = getchar()))
    num = num * 10 + c - '0';
    return (flag ? -1 : 1) * num;
}

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void dfs(int u, int fa) {
    int son[N];
    int son1[N];
    bool flag = false;
    f[u][1] = w[u];
    int cnt = 0;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (fa == j) continue;
        flag = true;
        // cout << u << " : " << j << endl;
        dfs(j, u);
        son[cnt] = min(f[j][1], f[j][2]);
        son1[cnt ++] = f[j][1]; 
        
        // if (u == 2) {
        //     cout << "'++++++++'" << endl;
        //     cout << j << endl;
        //     cout << min(min(f[j][0], f[j][1]), f[j][2]) << endl;; 
        //     cout << f[u][1] << ' ' << f[u][0] << endl;
        // }
        f[u][1] += min(min(f[j][0], f[j][1]), f[j][2]); 
        f[u][0] += min(f[j][1], f[j][2]);
    }
    // if (u == 1) {
    //     for (int i = 0; i < cnt; i ++)
    //     {
    //         cout << son[i] << " ";
    //     }
    //     puts("");
    //     for (int i = 0; i < cnt; i ++)
    //     {
    //         cout << son1[i] << " ";
    //     }
    //     puts("");
    // }
    if (!flag) f[u][2] = 0x3f3f3f3f;
    else {
        int res = 0x3f3f3f3f;
        for (int i = 0; i < cnt; i ++) {
            int sum = son1[i];
            for (int j = 0; j < cnt; j ++) {
                if (i != j) {
                    sum += son[j];
                }
            }
            res = min(res, sum);
        }
        f[u][2] = res;
    }
    // cout << u << " : " << f[u][0] << ' ' << f[u][1] << ' ' << f[u][2] << endl;
}


int main() {
    memset(h, -1, sizeof h);
    n = read();
    for (int i = 0; i < n; i ++) {
        int a = read(), c = read(), num = read();
        w[a] = c;
        while(num --) {
            int b = read();
            add(a, b), add(b, a);
        }
    }
    dfs(1, -1);
    printf("%d\n", min(f[1][1], f[1][2]));
    return 0;
}
```

