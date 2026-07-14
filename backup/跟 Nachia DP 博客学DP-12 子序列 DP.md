### 跟 Nachia DP 博客学DP-12 子序列 DP

**原文**：[DP 的俗称 | Mathenachia --- DP の俗称 | Mathenachia](https://www.mathenachia.blog/dp/#toc15)

对于某个序列的（不一定连续的）所有子序列，在去除重复后进行处理的典型 DP 被称为**子序列 DP**。

> [!note]
>
> **「例题1」**:
>
> 考虑序列 $A = (a_1,a_2,\ldots,a_N)$ 有多少中子序列

朴素的想就是一个位置上选或者不选，那么是 $2^N$ 种组合，复杂度是 $O(2^N)$ 的，那么考虑如何不去重复计算即可

#### 法一

**通过限制“前驱”的位置来避免重复计数**

如果我们要判断字符串 $S$ 是不是 $T$ 的子序列，我们总会贪心的在 $T$ 种寻找 $S$ 的各个字符**最靠前**的位置匹配。

这样一来，当选择第 $i$ 个元素的时候，考虑其可能的前一个元素，也就是第 $i$ 个元素之前，且值与 $a_i$ 相同的元素：

- 如果不存在，那么所有元素都可以是前一个元素
- 如果存在，那么设存在的元素中最后一个的编号是 $j$，那么前一个元素的编号要大于等于 $j$

所以这里就有我们的状态方程了：$dp[j][i]=$ 最后取出的元素在原序列的区间 $[j,i]$ 内的子序列问题的解

而为什么是上面那样的逻辑呢？

- 假设我们正在考虑把第 $i$ 个元素 $a_i$ 加入子序列。如果 $a_i$ 这个值在前面出现过，且上一次出现的最晚位置是 $j$。如果我们允许 $a_i$ 接在位置 $\lt   j$ 的元素后面，那么这部分结果和“把 $a_j$ 接在位置 $\lt  j$ 的元素后面”是完全一样的，这就产生了重复。 为了强制同一长相的子序列只被计算一次，我们规定：**如果要选当前的 $a_i$，它的前一个元素的下标必须 $\ge j$**。

#### 法二

**容斥定理**

$dp[i]=$ 由前缀 $(a_k)_{k=1}^i$ 能够构成的**本质不同**的子序列数量

转移如下：当考虑第 $i$ 个字符时：

- 不选 $a_i$：继承前 $i -1$ 个字符所构成的所有子序列，数量就是 $dp[i-1]$
- 选 $a_i$：前面所有的子序列后面都加上 $a_i$，数量就是 $dp[i] = dp[i - 1] \times 2$，但是这部分是有**重复**的。考虑**如何去重**？假如 $a_i$ 在之前的位置 $k$ 曾经出现过（即 $a_i = a_k$）。当初处理到位置 $k$ 时，我们已经把 $a_k$ 接在了 $(a_1 \dots a_{k-1})$ 的所有子序列后面。现在如果再把 $a_i$ 接在 $(a_1 \dots a_{k-1})$ 的序列后面，产生的新序列就会和当时完全重复！ 因此，需要减去这部分重复的量，即前 $k-1$ 个字符构成的子序列数量。

那么转移方程就是： $dp[i] = dp[i - 1] \times 2 - dp[k - 1]$ （$k$ 是 $a_i$ 上一次出现的位置，没出现过就不减这部分）

#### 法三

**自动机 DP**

$dp[i][c]=$ 在 $(a_k)_{k=1}^i$ 的子序列中，以 $c$ 结尾的种类数

$sum[i] =$ $(a_k)_{k=1}^i$ 的**本质不同**子序列数（含空序列） 

转移就是：

$\begin{align}
  \text{dp} \lbrack i + 1 \rbrack \lbrack c \rbrack &= \begin{cases} \text{sum} \lbrack i \rbrack & \quad ( S _ i = c ) \cr \text{dp} \lbrack  i \rbrack \lbrack c \rbrack & \quad ( S _ i \neq c ) \end{cases} \cr
  \text{sum} \lbrack i + 1 \rbrack &= 1 + \sum _ {c} \text{dp} \lbrack  i + 1 \rbrack \lbrack c \rbrack = 2 ~ \text{sum} \lbrack i \rbrack - \text{dp} \lbrack i \rbrack \lbrack S _ i \rbrack
\end{align}$

```cpp
int main() {
  int N;
  cin >> N;
  map<int, mint> dp;
  mint sum = 1;
  for (int i = 0; i < N; i++) {
    int A;
    cin >> A;
    mint temp = dp[A];
    dp[A] = sum;
    sum += sum - temp;
  }
  cout << (sum - 1).val();
}
```

类似字典树的想法，其实就是每加入一个新节点的时候，就去与之前没出现过的位置加入该节点，然后答案是节点数 $- 1$（去除根节点）

#### 例题

##### 模板题

[AtCoder ABC214 F - Substrings](https://atcoder.jp/contests/abc214/tasks/abc214_f)

##### MID ++

[AtCoder TDPC G - 字典序](https://atcoder.jp/contests/tdpc/tasks/tdpc_lexicographical)

子序列自动机(?)

```cpp
auto Mainsol = [](){
  STR(S); LL(K);
  ll N = sz(S);
  v<array<int, 26>> nxt(N + 1, {0});
  v<ll> f(N + 1, 0); // 以 S[i] 为首字母开始的本质不同子序列数

  rep(c, 26) nxt[N][c] = N;
  rrep(i, N) {
  	rep(c, 26) nxt[i][c] = nxt[i + 1][c];
  	nxt[i][S[i] - 'a'] = i; // 从 i 之后，字符c 第一次出现的位置
  }

  rrep(i, N) {
  	f[i] = 1;
  	rep(c, 26) {
  		int next_idx = nxt[i + 1][c];
  		if(next_idx < N) 
  			f[i] = min(LINF, f[i] + f[next_idx]);
  	}
  }

  ll Sum = 0;
  rep(c, 26) {
  	if(nxt[0][c] < N) Sum = min(LINF, Sum + f[nxt[0][c]]);
  }

  if(K > Sum) return out("Eel");

  string ans = "";
  int now = 0;
  while(K > 0) {
  	rep(c, 26) {
  		int next_idx = nxt[now][c];
  		if(next_idx < N) {
  			if(K <= f[next_idx]) {
  				ans += (char)('a' + c);
  				K -= 1;
  				now = next_idx + 1;
  				break;
  			} else {
  				K -= f[next_idx];
  			}
  		}
  	}
  }
  out(ans);

};
```

