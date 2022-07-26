# 数位DP



数位dp的题目一般会问，**某个区间内，满足某种性质的数的个数**。

对应的数位dp问题有相应的解题技巧：

1. 利用前缀和求某个区间内满足条件的数的个数
2. 利用树的结构来考虑（按位分类讨论）



## 度的数量



### 原题链接

> https://www.acwing.com/problem/content/description/1083/



### 思路

通过前缀和思想计算出从 0 ~ n的所有方案数，给定区间做差即可

将n分解为B进制数，从高位开始依次考虑每个数。

按照树形结构划分n，左分支为 0~a-1, 右分支为 a

### C++ 代码

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
#include <vector>

using namespace std;

const int N = 40;

int K, B;
int f[N][N];

void init() {
    for (int i = 0; i < N; i ++)
        for (int j = 0; j <= i; j ++)
            if (!j) f[i][j] = 1;
            else f[i][j] = f[i-1][j] + f[i-1][j-1];

}

int dp(int n) { // 从 0 ~ n 符合条件的数的个数
    if (!n) return 0;

    vector<int>nums;
    while(n) {
        nums.push_back(n % B);
        n /= B;
    }
    // for (int i = 0; i < nums.size(); i ++) cout << nums[i] << ' ';
    // puts("");
    int res = 0, last = 0;
    for (int i = nums.size()-1; i >= 0; i --) {
        int x = nums[i]; // x = 0, x = 1, x > 1
        // 对x分类讨论
        // cout << "x : " << x << endl;
        if (x) { // x != 0 即 x > 0
            res += f[i][K-last]; // 该位填0, 剩下从i位中选择K-last个位置填1
            // cout << res << endl;
            // cout << i << ' ' << K-last << endl;
            // cout << f[i][K-last] << endl;
            if (x > 1) { // 该位可以填1
                if (K - last - 1 >= 0)
                    res += f[i][K-last-1]; // 剩下从i位中选择K-last-1个位置填1
                // 对于符合条件的已经处理完成，直接break
                break;
            } else {
                // 对应 x == 1
                last ++; // 该位置填 1
                if (last > K) break;
            }
        }

        // 最后一个分支，即最后一位，如果为1则已经被上方处理过，如果为0，则方案数加1
        if (i == 0 && last == K) res ++;
    }
    return res;
}
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
    init();
    int x = read(), y =read();
    K = read(), B = read();
    printf("%d\n", dp(y) - dp(x-1));
    return 0;   
}
```


## 数字游戏



### 原题链接

> https://www.acwing.com/problem/content/1084/



### 思路

#### 状态表示

```
f[i][j] 表示共有i位，最高位为j的个数
```

#### 状态计算

```
f[i][j] = f[i-1][k] (k = j, j+1, j+2, j+3, ... 9)
```



### C++ 代码

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
#include <vector>
using namespace std;

const int N = 40;

int f[N][N];

void init() {
    for (int i = 0; i < 10; i ++) f[1][i] = 1;
    for (int i = 2; i < N; i ++)
        for (int j = 0; j <= 9; j ++)  
            for (int k = j; k < N; k ++)
                f[i][j] += f[i-1][k];
}

int dp(int n) {
    if (!n) return 1;
    vector<int> nums;
    while(n) nums.push_back(n%10), n/=10;

    int res = 0, last = 0; // last表示上一位的数字
    for (int i = nums.size() - 1; i >= 0; i --) {
        int x = nums[i];
        // 分支一：考虑从last~x-1
        for (int j = last; j < x; j ++) { 
            res += f[i+1][j];
        }
		// 不满足条件后退出
        if (x < last) break;
        // 分支二：x分支
        last = x;
        if (!i) res ++;
    }
    return res;
}

int main() {
    
    init();
    int a, b;
    while(cin >> a >> b) cout << dp(b) - dp(a - 1) << endl;
    return 0;

}
```



## Windy数

### 原题链接

> https://www.acwing.com/problem/content/1085/



### 思路

#### 状态表示

```
f[i][j] 表示考虑前i位，最高位为j 的所有方案数量
```

#### 状态计算

```
f[i][j] += f[i-1][k] (k = 0, 1, ..., 9)
abs(j-k) >= 2
```

#### 前导0问题

枚举的第 `i+1` 位是最高位，不能填0。如果填0，那么答案就会加上`f[i+1][0]`举这样一个例子，
对于数字13，是满足windy数定义的，那么加上前导0之后的 `013` 就不会被 `f[3][0]` 加进去，原因就是abs(0-1)<2，这样就导致答案漏掉

因此分两部分来计算答案

* 第一部分考虑长度与 `n` 等长的数

  ```cpp
  for (int i = len-1; i >= 0; i --)
      {
          int x = nums[i];
          for (int j = (i == len-1); j < x; j ++) {
              if (abs(last - j) >= 2) {
                  res += f[i+1][j]; 
              }
          }
  
          if (abs(last-x) < 2) break;
          else last = x;
          if (!i) res ++;
      }
  ```

* 第二部分考虑长度小于 `n` 的数

  ```cpp
  for (int i = 1; i < len; i ++)
          for (int j = 1; j <= 9; j ++)
              res += f[i][j];
  ```

  

### C++ 代码

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
#include <vector>
#include <cmath>

using namespace std;

const int N = 11;

int f[N][N];

void init() {
    for (int i = 0; i < 10; i ++) f[1][i] = 1;
    for (int i = 2; i < N; i ++)
        for (int j = 0; j < 10; j ++)
            for (int k = 0; k < 10; k ++)
                if (abs(j-k) >= 2) f[i][j] += f[i-1][k];
}

int dp(int n) {
    if(!n) return 0;

    vector<int> nums;

    while(n) nums.push_back(n%10), n/=10;

    int res = 0;
    int last = -1000;
    int len = nums.size();
    for (int i = len-1; i >= 0; i --)
    {
        int x = nums[i];
        for (int j = (i == len-1); j < x; j ++) {
            if (abs(last - j) >= 2) {
                res += f[i+1][j]; 
            }
        }

        if (abs(last-x) < 2) break;
        else last = x;
        if (!i) res ++;
    }

    for (int i = 1; i < len; i ++)
        for (int j = 1; j <= 9; j ++)
            res += f[i][j];

    return res;
}

int main() {
    init();
    int a, b;
    cin >> a >> b;
    cout << dp(b) - dp(a-1) << endl;
    return 0;
}
```





## 数字游戏 II


### 原题链接

> https://www.acwing.com/problem/content/description/1086/

### 思路

#### 状态转移

```
f[i][j][k] 表示考虑i位数，最高位为j，k所有位数之和模N后的余数 所有方案数
```

#### 状态计算

```
f[i][j][k] += f[i-1][x][mod(k-j, P)] 前i-1位和模P的余数为mod(k-j, P)
```



### C++ 代码

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
#include <vector>

using namespace std;

const int N = 40, M = 110;

int f[N][10][M];
int P;

int mod(int x, int y) {
    return (x % y + y) % y; // 将得到的负余数转化为0~y-1之间
}

void init() {
    memset(f, 0, sizeof f);
    for (int i = 0; i < 10; i ++) f[1][i][i%P] = 1;

    for (int i = 2; i < N; i ++) //位数
        for (int j = 0; j < 10; j ++) // 最高位
            for (int k = 0; k < P; k ++) // 所有位上的和
                for (int x = 0; x < 10; x ++) // 下一位
                    f[i][j][k] += f[i-1][x][mod(k-j, P)];
    
}

int dp(int n) {
    if (!n) return 1;

    vector<int> nums;

    while(n) nums.push_back(n%10), n/=10;

    int res = 0;
    int last = 0;
    for (int i = nums.size() - 1; i >= 0; i --) {
        int x = nums[i];
        for (int j = 0; j < x; j ++)
            res += f[i+1][j][mod(-last, P)]; // 前面位之和为 last, 后面数字的和加上last % N 应为0

        last += x;
        if (!i && last % P == 0) res ++;
    }

    return res;
}   

int main() {
    int a, b;
    while(cin >> a >> b >> P) {
        init();
        cout << dp(b) - dp(a-1) << endl;
    }
    return 0;
}
```





## 不要62

### 原题链接

> https://www.acwing.com/problem/content/1087/



### 思路

#### 状态表示

```
f[i][j] 表示考虑前i位, 最高位为j的fang
```

#### 状态转移

```
f[i][j] = f[i-1][k]
j != 6 && k != 2 
j != 4
k != 4
```



### C++ 代码

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
#include <vector>

using namespace std;

const int N = 11;

int f[N][10];

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

void init() {
    for (int i = 0; i < 10; i ++)
        if (i != 4) f[1][i] = 1;
    for (int i = 2; i < N; i ++)
        for (int j = 0; j < 10; j ++)
        {
            if (j == 4) continue;
            for (int k = 0; k < 10; k ++)
                if (k == 4 || j == 6 && k == 2) continue;
                else f[i][j] += f[i-1][k];
        }
}

int dp(int n) {
    if (!n) return 1;
    vector<int> nums;
    while(n) nums.push_back(n%10), n /= 10;
    int res = 0, last = 0;
    for (int i = nums.size()-1; i >= 0; i --) {
        int x = nums[i];
        for (int j = 0; j < x; j ++) {
            if (j == 4 || last == 6 && j == 2) continue;
            else res += f[i+1][j];
        }

        if (x == 4 || last == 6 && x == 2) break;
        last = x;
        if (!i) res ++;
    }
    return res;
}

int main() {
    init();
    int n, m;
    while(n = read(), m = read(), n || m) {
        printf("%d\n", dp(m) - dp(n-1));
    }
    return 0;
}
```





## 经典数位DP问题



### 原题链接

> https://www.acwing.com/problem/content/1088/



### 思路

#### 迭代做法



**本题有3个限制条件**

1. 整数中某一位是 7；
2. 整数的每一位加起来的和是 7 的整数倍；
3. 这个整数是 7 的整数倍。

需要记录整数大小和整数的各位数字之和

因此状态除了位数，最高位还需要两维

**本题数据范围较大，处处需注意取模**

##### 状态表示

`f[i][j][a][b]` : `a` 表示数的大小模 `7` 的余数，`b` 表示数的各位数字之和模 `7` 的余数

##### 状态计算

![image-20220725221543872](D:/%E7%AC%94%E8%AE%B0/%E7%AE%97%E6%B3%95%E6%8F%90%E9%AB%98%E8%AF%BE/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92/%E6%95%B0%E4%BD%8DDP.assets/image-20220725221543872.png)

```cpp
for (int i = 0; i < 10; i ++) { 
        if (i == 7) continue;
        f[1][i][i%7][i%7] = {1, i, i * i};
    }
    ll power = 10;
    for (int i = 2; i < N; i ++, power *= 10) 
        for (int j = 0; j < 10; j ++) 
            if (j == 7) continue;
            else 
                for (int a = 0; a < 7; a ++)
                    for (int b = 0; b < 7; b ++)
                        for (int k = 0; k < 10; k ++) {
                            if (k == 7) continue;
                            // a = j * 10^(i-1) + (...)  %7
                            // b = j + (...)			 %7
                            auto &v1 = f[i][j][a][b], &v2 = f[i-1][k][mod(a-j*power, 7)][mod(b-j, 7)];
                            v1.s0 = mod(v1.s0 + v2.s0, P);
                            // 一次项
                            v1.s1 = mod(v1.s1 + j * (power % P) % P * v2.s0 + v2.s1, P);
                            // 二次项
                            v1.s2 = mod(v1.s2 + j * j * (power % P) % P * (power % P) % P * v2.s0 + 
                                        2 * j * (power % P) % P * v2.s1 % P +
                                        v2.s2
                                    , P);
                        }
```



### C++ 代码

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
#include <vector>

using namespace std;
typedef long long ll;
typedef long long LL;

const int N = 20, P = 1e9 + 7;

struct F{
    ll s0, s1, s2; // s0 表示数量, s1 表示所有符合条件的数的一次项和, s2 表示所有符合条件的数的二次项和
} f[N][10][10][10]; // 表示考虑前i位，最高位为j，该数模7的余数，数字之和模7的余数

int mod(ll x, ll y) {
    return (x % y + y) % y;
}

int power7[N], powerP[N];

void init() {
    for (int i = 0; i < 10; i ++) {
        if (i == 7) continue;
        f[1][i][i%7][i%7] = {1, i, i * i};
    }
    ll power = 10;
    for (int i = 2; i < N; i ++, power *= 10) 
        for (int j = 0; j < 10; j ++) 
            if (j == 7) continue;
            else 
                for (int a = 0; a < 7; a ++)
                    for (int b = 0; b < 7; b ++)
                        for (int k = 0; k < 10; k ++) {
                            if (k == 7) continue;
                            // a = j * 10^(i-1) + ...
                            // b = j + ...
                            auto &v1 = f[i][j][a][b], &v2 = f[i-1][k][mod(a-j*power, 7)][mod(b-j, 7)];
                            v1.s0 = mod(v1.s0 + v2.s0, P);
                            v1.s1 = mod(v1.s1 + j * (power % P) % P * v2.s0 + v2.s1, P);
                            v1.s2 = mod(v1.s2 + j * j * (power % P) % P * (power % P) % P * v2.s0 + 
                                        2 * j * (power % P) % P * v2.s1 % P +
                                        v2.s2
                                    , P);
                        }
    power7[0] = 1;
    powerP[0] = 1;
    for (int i = 1; i < N; i ++) {
        power7[i] = power7[i-1] * 10 % 7;
        powerP[i] = powerP[i-1] * 10ll % P;
    }
}




F get(int i, int j, int a, int b) {
    ll s0 = 0, s1 = 0, s2 = 0;
    for (int x = 0; x < 7; x ++) 
        for (int y = 0; y < 7; y ++)
        {
            if (x == a || y == b) continue;
            auto &v = f[i][j][x][y];
            // cout << x << ' ' << y << ' ' << v.s2 << endl;
            s0 = (s0 + v.s0) % P;
            s1 = (s1 + v.s1) % P;
            s2 = (s2 + v.s2) % P;
        }
    return {s0, s1, s2};
}

ll dp(ll n) {
    if (!n) return 0;

    ll nn = n % P;
    vector<int> nums;

    while(n) nums.push_back(n%10), n/=10;
    
    ll last1 = 0, last2 = 0; // last1 存储上一个数的大小，last2存储上一个数的各位数字之和
    ll res = 0;
    
    for (int i = nums.size() - 1; i >= 0; i --) {
        int x = nums[i];
        
        for (int j = 0; j < x; j ++) {
            if (j == 7) continue;
            // 0 = last1 * 10^(i+1) + j  %7
            // 0 = j + last2 
            ll a = mod(-last1 * power7[i+1], 7);
            ll b = mod(-last2, 7);
            // cout << "a and b :  ";
            // cout << a << ' ' << b << endl;
            auto v = get(i+1, j, a, b);
            // cout << v.s0 << ' ' << v.s1 << ' ' << v.s2 << endl;
            // cout << (last1 % P) * (last1 % P) % P * powerP[i+1] % P * powerP[i+1] % P * v.s0 + 2 * last1 % P * powerP[i+1] % P * v.s1 + v.s2 << endl;
            res = mod(res + (last1 % P) * (last1 % P) % P * powerP[i+1] % P * powerP[i+1] % P * v.s0 + 2 * last1 % P * powerP[i+1] % P * v.s1 + v.s2, P);
            // cout << res << endl;
        }
        
        if (x == 7) break;
        last1 = last1 * 10 + x;
        last2 += x;
        // cout << last1 << ' ' << last2 << endl;
        if (!i && last1 % 7 && last2 % 7) res = (res + nn * nn) % P;
    }
    return res;
}

int main (){
    init();
    int T;
    // cout << f[1][0][0][0].s0 << endl;
    cin >> T;
    while(T--) {
        ll l, r;
        cin >> l >> r;
        cout << mod(dp(r) - dp(l-1), P) << endl;
    }
    return 0;
}

```



