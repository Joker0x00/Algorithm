# 区间DP



## 石子合并问题



### [题目链接](https://www.acwing.com/problem/content/description/284/)

### 状态表示和状态转移

```c++
f[i][j] 表示合并区间 [i, j] 所有方案中花费代价的最小值
f[i][j] = min{f[i][k] + f[k+1][j] + s[j] - s[i-1]} (k=i, ..., j-1)
answer = f[1][n]
```



```c++
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;
 

/*
    f[i][j] 表示合并区间 [i, j] 所有方案中花费代价的最小值
    f[i][j] = min{f[i][k] + f[k+1][j] + s[j] - s[i-1]} (k=i, ..., j-1)
    answer = f[1][n]
*/

const int N = 310;
int f[N][N];
int s[N];
int n;

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i ++) {
        scanf("%d", &s[i]);
        s[i] += s[i-1];
    }
    memset(f, 0x3f, sizeof f);
    for (int i = 0; i <= n; i ++) f[i][i] = 0;

    for (int len = 2; len <= n; len ++) {
        for (int i = 1; i + len - 1 <= n; i ++) {
            int j = i + len - 1;
            for (int k = i; k < j; k ++) {
                f[i][j] = min(f[i][j], f[i][k] + f[k+1][j] + s[j] - s[i - 1]);
            }
        }
    }
    printf("%d\n", f[1][n]);
    return 0;
}
```





## 环形石子合并


### [题目链接](https://www.acwing.com/problem/content/description/1070/)

### 状态表示和状态转移



```
思路整体和上题类似
不同之处在于环形的处理上，采用复制法获得环形效果，减小时间复杂度
```



```c++
#include <iostream>
#include <algorithm>
#include <cstring>
 
using namespace std;

const int N = 410, INF = 0x3f3f3f3f;
int s[N], a[N];
int f[N][N], g[N][N];
int n;

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

int main() {
    n = read();
    for (int i = 1; i <= n; i ++) {
        s[i+n] = s[i] = read();
    }
    memset(f, 0x3f, sizeof f);
    memset(g, -0x3f, sizeof g);
    for (int i = 1; i <= 2*n; i ++) s[i] += s[i-1];
    for (int i = 0; i <= 2*n; i ++) {
        f[i][i] = g[i][i] = 0;
    }
    for (int len = 2; len <= n; len ++) {
        for (int i = 1; i + len - 1 <= 2*n; i ++) {
            int j = i + len - 1;
            for (int k = i; k < j; k ++) {
                f[i][j] = min(f[i][j], f[i][k] + f[k+1][j] + s[j] - s[i-1]);
                g[i][j] = max(g[i][j], g[i][k] + g[k+1][j] + s[j] - s[i-1]);
            }
        }
    }
    int maxv = -INF, minv = INF;
    for (int i = 1; i <= n; i ++) {
        maxv = max(maxv, g[i][i+n-1]);
        minv = min(minv, f[i][i+n-1]);
    }
    printf("%d\n%d\n", minv, maxv);
    return 0;
}
```





## 能量石

### [题目链接](https://www.acwing.com/problem/content/322/)

### 状态表示和状态转移



```
(2，3)(3，5)(5，10)(10，2)
环状转换为链状
2 3 5 10 2 3 5 10
最后合并为一个首位都相同的能量石, 此处最大区间长度应为 n+1
答案区间长度也为 n+1
f[i][j] = min{f[i][k] + f[k+1][j] + a[i] * a[k] * a[j]} (k = i+1, ..., j-1)
answer = max{f[i][i+n]}
```



```c++
#include <iostream>
#include <algorithm>
#include <cstring>
 
using namespace std;

const int N = 210;
int f[N][N];
int a[N];
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
int n;
int main()
{
    n = read();
    // memset(f, -0x3f, sizeof f);
    for (int i = 1; i <= n; i ++) {
        a[i] = a[i+n] = read();
    }
    // for (int i = 0; i <= n; i ++) f[i][i+1] = 0;

    for (int len = 3; len <= n+1; len ++) {
        for (int i = 1; i + len - 1 <= 2*n; i ++) {
            int j = i + len - 1;
            for (int k = i+1; k < j; k ++) {
                f[i][j] = max(f[i][j], f[i][k] + f[k][j] + a[i] * a[k] * a[j]);
            }
        }
    }
    int res = 0;
    for (int i = 1; i <= n; i ++) {
        res = max(res, f[i][i+n]);
    }
    printf("%d\n", res);
    return 0;
}

/*
    (2，3)(3，5)(5，10)(10，2)
    2 3 5 10 2 3 5 10
    最后合并为一个首位都相同的能量石, 此处最大区间长度应为 n+1
    答案区间长度也为 n+1
    f[i][j] = min{f[i][k] + f[k+1][j] + a[i] * a[k] * a[j]} (k = i+1, ..., j-1)
    answer = max{f[i][i+n]}
*/
```





## 凸多边形的划分





### [题目链接](https://www.acwing.com/problem/content/1071/)



### 状态表示和状态转移



```
f[i][j] = min{f[i][k] + f[k][j] + a[i] * a[k] * a[j]} (k = i+1, ..., j-1)
answer = f[1][n]
```

![image-20220719183543700](D:/%E7%AC%94%E8%AE%B0/%E7%AE%97%E6%B3%95%E6%8F%90%E9%AB%98%E8%AF%BE/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92/%E5%8C%BA%E9%97%B4dp.assets/image-20220719183543700.png)

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
 
using namespace std;
typedef long long ll;
const int N = 55, M = 40, INF = 0x3f3f3f3f;
int n, a[N];
int f[N][N][M];
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

void add(int a[], int b[]) {
    static int res[M];
    memset(res, 0, sizeof res);
    int t = 0;
    for (int i = 0; i < M; i ++) {
        t += a[i] + b[i];
        res[i] = t % 10;
        t /= 10;
    }
    memcpy(a, res, sizeof res);
}

void mul(int a[], int b) {
    static int res[M];
    memset(res, 0, sizeof res);
    ll t = 0;
    for (int i = 0; i < M; i ++) {
        t += (ll)a[i] * b;
        res[i] = t % 10;
        t /= 10;
        // cout << t << endl;
    }
    // for (int i = 0; i < M; i ++) cout << res[i];
    // cout << endl;
    memcpy(a, res, sizeof res);
    // puts("");
}

int cmp(int a[], int b[]) {
    for (int i = M-1; i >= 0; i --) {
        if (a[i] > b[i]) return 1;
        else if (a[i] < b[i]) return -1;
    }
    return 0;
}

void print(int a[]) {
    int k = M-1;
    while(k && !a[k]) k --;
    while(k >= 0) printf("%d", a[k--]);
    puts("");
}


int main(int argc, char const *argv[])
{
    n = read();
    for (int i = 1; i <= n; i ++) a[i] = read();
    int tmp[M];
    for (int len = 3; len <= n; len ++) {
        for (int i = 1; i + len - 1 <= n; i ++) {
            int j = i + len - 1;
            f[i][j][M-1] = 1;

            for (int k = i+1; k < j; k ++) {
                memset(tmp, 0, sizeof tmp);
                tmp[0] = 1;
                mul(tmp, a[i]);
                mul(tmp, a[k]);
               
                mul(tmp, a[j]);
           
                add(tmp, f[i][k]);

                add(tmp, f[k][j]);
 
                if (cmp(f[i][j], tmp) > 0) memcpy(f[i][j], tmp, sizeof tmp);
            }
        }
    }
    print(f[1][n]);
    return 0;
}
```



### 注意点

```
在计算高精度函数内，需要执行memcpy进行数组拷贝操作，第三个参数设置时需要注意
不能是 sizeof a, 应该是sizeof res
因为 a 作为参数是指针类型，大小为8个字节，在拷贝时，只会拷贝前两个数
```



## 加分二叉树

### [题目链接](https://www.acwing.com/problem/content/481/)



### 思路

```
对区间[l, r]按照根的位置进行划分

得到状态转移方程如下:
f[l][r] = max{f[l][k-1] * f[k+1][r] + a[k]} (k = l, l+1, ..., r)
端点处需特殊处理
```

### 代码

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
 
using namespace std;

const int N = 40;

int f[N][N], g[N][N]; //g[i][j] 记录区间[i, j]中的根结点
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
int n, a[N];

void dfs(int l, int r) {
    if (l > r) return;
    int root = g[l][r];
    printf("%d ", root);
    dfs(l, root-1);
    dfs(root+1, r);
}

int main()
{
    n = read();
    for (int i = 1; i <= n; i ++) f[i][i] = a[i] = read(), g[i][i] = i;

    for (int len = 2; len <= n; len ++) {
        for (int i = 1; i + len - 1 <= n; i ++) {
            int j = i + len - 1;
            for (int k = i; k <= j; k ++) {
                int left = i == k ? 1 : f[i][k-1];
                int right = j == k ? 1 : f[k+1][j];
                if (f[i][j] < left * right + a[k]) {
                    f[i][j] = left * right + a[k];
                    g[i][j] = k;
                }
            }
        }
    }
    printf("%d\n", f[1][n]);
    dfs(1, n);
    return 0;
}
```







## 棋盘分割

### [题目链接](https://www.acwing.com/problem/content/description/323/)

### 思路

**题意+分析**

将棋盘切 $n-1$ 刀，得到 $n$ 块小区域，使得 $n$ 块区域权重之和的均方差最小，由于n固定，均方差和方差的单调性相同，在分析时只考虑方差能简化运算，最后求和之后再算均值开方即得到答案



切分之后，左右或上下两部分是独立的，可以分别考虑。另外注意，切分后保留的一部分继续切分，而舍弃的一部分直接计算

**分割方案**

![image-20220720110805538](D:/%E7%AC%94%E8%AE%B0/%E7%AE%97%E6%B3%95%E6%8F%90%E9%AB%98%E8%AF%BE/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92/%E5%8C%BA%E9%97%B4dp.assets/image-20220720110805538.png)

**状态表示**

```
f[k][x1][y1][x2][y2] 表示进行第k-1次划分时, 选择对角坐标为[(x1, y1), (x2, y2)]的矩形的各部分与均值差值的平方之和最小值
```

**状态计算**

```
对于每一层状态，考虑上述四种分割分割方案，保留差值平方之和最小值
```

### C++ 代码

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
#include <cmath>

using namespace std;

const int N = 16;

double f[N][N][N][N][N];
int s[N][N];
int n, m = 8;
double X;

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

double get(int x1, int y1, int x2, int y2) {
    double x = s[x2][y2] - s[x1-1][y2] - s[x2][y1-1] + s[x1-1][y1-1] - X;
    return x * x;
}

double dp(int k, int x1, int y1, int x2, int y2) { // 求 (xi - X)^2
    double &v = f[k][x1][y1][x2][y2];
    if (v >= 0) return v;
    if (k == n) return v = get(x1, y1, x2, y2);
    double t = 1e9;
    for (int i = y1; i < y2; i ++) {
        t = min(t, dp(k+1, x1, y1, x2, i) + get(x1, i+1, x2, y2));
        t = min(t, dp(k+1, x1, i+1, x2, y2) + get(x1, y1, x2, i));
    }

    for (int i = x1; i < x2; i ++) {
        t = min(t, dp(k+1, x1, y1, i, y2) + get(i+1, y1, x2, y2));
        t = min(t, dp(k+1, i+1, y1, x2, y2) + get(x1, y1, i, y2));
    }
    return v = t;
}

int main() {
    memset(f, -1, sizeof f);
    n = read();
    for (int i = 1; i <= m; i ++) {
        for (int j = 1; j <= m; j ++) {
            s[i][j] = read() + s[i-1][j] + s[i][j-1] - s[i-1][j-1];
        }
    }
    X = (double)s[m][m] / n; 
    printf("%.3lf\n", sqrt(dp(1, 1, 1, m, m) / n));
    return 0;
}
```

**注意点**

```
double memset 为-1代表
```



