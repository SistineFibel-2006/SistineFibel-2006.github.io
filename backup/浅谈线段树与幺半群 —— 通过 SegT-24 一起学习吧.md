# 浅谈线段树与幺半群 —— 通过 SegT-24 一起学习吧

---

---

参考资料：https://info.atcoder.jp/entry/algorithm_lectures/range_query_monoid_design

---

## 什么是线段树？

通常来讲，可以理解为“能够处理幺半群区间积”的数据结构就是线段树。

因而对于一个线段树，我们只需要定义其数据类型 `S`，二元运算 `op`，单位元 `e` 即可。

实际上这里 `(S,op,e)` 需要构成一个幺半群。这个术语在这里可能存在疑惑，但是你现在不太需要能完全理解这个概念，下面会继续讲解。

## 什么是幺半群？如何确定问题的幺半群？

例如，序列 $A$ 的子串 $X=(A_L,A_{L+1},\ldots,A_{R-1})$ 对应了某种你想要求得的信息，这样的信息的值我们称作为 $f(X)$，对于一个区间 $[L, R)$ 的查询，从线段树中获得 $f(X)$。

对于两个序列 $X$，与 $Y$，我们把将其结合写作 $XY$。

那么幺半群就是：

- `S`：$f(X)$ 的类型。
- `op`：从 $f(X),f(Y)$ 求得 $f(XY)$ 的函数。
- `e`：对于空序列的信息 $f(\set{})$

那么 `op` 实际上就是满足 $op(f(X),f(Y)) = f(XY)$ 的函数。

幺半群实际上要满足下列两种法则：

- 单位元（幺元）的性质：$op(e,X) = op(X,e) = X$
- 结合律：$op(op(X,Y),Z) = op(X,op(Y,Z))$

但是实际上，只要正确实现了 $op(f(X),f(Y)) = f(XY)$ 的话，这些性质是一定成立的。

所以实际上这里引入幺半群只是为了确定我们要设计的东西！

我们下面所涉及的定义序列信息与设计其结合方式，也就被称作“**设计幺半群**”。

---

## 情形1: 直接从序列中获得并处理出所需要的信息

设计幺半群的时候，首先要学习的基本思路就是：对于一个序列，直接将信息作为 $f(X) \in S$ 来处理出。

---

### 例子1: 区间和问题

> [!note]
>
> **【问题1】（区间和）**
>
> 给定序列 $A = (A_0,A_1,\ldots,A_{N-1})$。处理以下的 $Q$ 次查询，查询有两种。
>
> 1. 给定整数 $0\le i \lt N$ 和 $x$：$A_i := x$
> 2. 给定整数 $0\le L \lt R \le N$，求 $\sum_{L\le i \lt R} A_i$

那么我们显然可以直接定义 $f(X) = \sum_{i\in X} i$ 为状态，结合就是：
$$
f(XY) = f(X) + f(Y)
$$
空串的总和是 $0$，于是就可以设计出来了：

```cpp
using S = long long;

S op(S x, S y) {return x + y;}

S e() {return 0;}
```

### 例子2: 区间取最小值

> [!note]
>
> **【问题2】（区间取最小值）**
>
> 给定序列 $A = (A_0,A_1,\ldots,A_{N-1})$。处理以下的 $Q$ 次查询，查询有两种。
>
> 1. 给定整数 $0\le i \lt N$ 和 $x$：$A_i := x$
> 2. 给定整数 $0\le L \lt R \le N$，求 $\min_{L\le i \lt R} A_i$

这种情况下，其实是同样的设计
$$
f(X) = \min_{i \in X} i
$$
合并就是两边的最小值再取最小，可以写出：

```cpp
using S = long long;
const long long inf = 1LL << 60; // 需要给一个大于题目内所有数字的数

S op(S x, S y) {return min(x, y);}

S e() {return inf;}
```

这里其实有一些很值得思考的点，比如幺元的设计：

通常来讲，我们设计幺元只要能够满足 $op(e,X) = op(X,e) = X$ 即可，但是我们也可以通过类似代数数据结构的角度来存储这个幺元，也就是将数据类型变成：是幺元、不是幺元（存储数据） 的形式。例如：

```cpp
struct S {
  bool is_empty;
  int min;
};

S op(S x, S y) {
  if(x.is_empty) return y;
  if(y.is_empty) return x;
  S z;
  z.is_empty = false;
  z.min = min(x.min, y.min);
  return z;
}

S e() {return {true, 0};}
```

当然，这样的设计在这里看上去可能有些多余，但是在一些复杂的状态下，或许会变得很有利。

### 例子3: 多种值的组合

如果要查询多种值，可以考虑组合在一起共同维护。例如下面：
> [!note]
>
> **【问题3】（区间求和、取最小值、取最大值）**
>
> 给定序列 $A = (A_0,A_1,\ldots,A_{N-1})$。处理以下的 $Q$ 次查询，查询有两种。
>
> 1. 给定整数 $0\le i \lt N$ 和 $x$：$A_i := x$
> 2. 给定整数 $0\le L \lt R \le N$，求 $\sum_{L\le i \lt R} A_i$
> 3. 给定整数 $0\le L \lt R \le N$，求 $\min_{L\le i \lt R} A_i$
> 4. 给定整数 $0\le L \lt R \le N$，求 $\max_{L\le i \lt R} A_i$

显然的解法是，用三个线段树来维护，但是我们也可以组合这些值：

```cpp
struct S {
  long long sum, min, max;
};

S op(S x, S y) {
  S z;
  z.sum = x.sum + y.sum;
  z.min = min(x.min, y.min);
  z.max = max(x.max, y.max);
  return z;
}

S e() {return {0, inf, -inf};}
```

在这个问题中，我们将 $3$ 个各自单独也能用线段树处理的值合并成了一个。另一方面，有时也会将多个相互关联的值作为一个整体信息来处理。

> [!note]
>
> **【问题4】（区间最小值与个数）**
>
> 给定序列 $A = (A_0,A_1,\ldots,A_{N-1})$。处理以下的 $Q$ 次查询，查询有两种。
>
> 1. 给定整数 $0\le i \lt N$ 和 $x$：$A_i := x$
> 2. 给定整数 $0\le L \lt R \le N$，求 $x = \sum_{L\le i \lt R} A_i$，并求出满足 $A_i = x$ 的 $i$  的个数

那么这里，我们只需要在合并的时候，同时合并最小值的个数即可

```cpp
struct S {
  int min, cnt;
};

S op(S x, S y) {
  S z;
  z.min = min(x.min, y.min);
  if(z.min == x.min) z.cnt += x.cnt;
  if(z.min == y.min) z.cnt += y.cnt;
  return z;
}

S e() {return {inf, 0};}
```

### 要点

其实就是考虑通过 $f(X),f(Y)$ 求得 $f(XY)$ 的时候所需要的信息，通过这些信息来设计 $f(X)$ 本身即可。

> [!note]
>
> **【问题5】（区间求均值）**
>
> 给定序列 $A = (A_0,A_1,\ldots,A_{N-1})，B = (B_0,B_1,\ldots,B_{N-1})$。处理以下的 $Q$ 次查询，查询有两种。
>
> 1. 给定整数 $0\le i \lt N$ 和 $x, y$：$A_i := x，B_i := y$.
> 2. 给定整数 $0\le L \lt R \le N$，对于每一个 $i \in [L, R)$，认为有 $B_i$ 个 $A_i$ 价值的物品，求出这些物品的平均价值.

那么肯定要存储的就是**区间内物品的平均值**对吧。但是考虑下面这个问题

> [!note]
>
> 有两堆物品 $X, Y$。
>
> 第一堆的平均价格是 $x$，第二堆是 $y$，求两堆合并之后的平均价格。

聪明的你一定能想到，只有 $x, y$ 显然是不能解决这个问题的，因为不知道每一堆的物品数量。所以显然不能用下面这个来设计

```cpp
using S = double;
S op(S x, S y) {
  return (x + y) / 2.0f;
}
```

而且实际上，$op(x,y) = \frac{x + y}{2}$ 这个运算也是不满足结合律的，用笔演算一下就能发现，但是这个设计最主要的问题，在于不可以只用两堆物品的平均值来求得整体平均值。

同时，需要注意的是，一般来讲，除法运算都是天然不满足结合律的。

那么可以这个问题的思考方式，可以转化为如何求得**一堆物品的平均值**？显然就是

- $X$ 中的物品个数
- $X$ 中的物品总价

所以这样就可以了！

```cpp
struct S {
  long long cnt;
  long long sum;
};

S op(S x, S y) {
  S z;
  z.cnt = x.cnt + y.cnt;
  z.sum = x.sum + y.sum;
  return z;
}

S e() {return {0, 0};}

segtree<S, op, e> segt;

double wk(int L, int R) {
  S x = seg.prod(L, R);
  return 1.0f * x.sum / x.cnt;
}
```

在这个例子中，只需将平均值改写为“个数与总和”，就能分别独立地结合它们。另一方面，为了计算所需的信息可能需要辅助信息，而为了结合该辅助信息又可能需要其他信息。下一章将讨论在这种情况下依次添加所需信息的思路。

## 情形2: 添加求得的信息所需的必须的信息

有些时候，只凭借所需要的信息无法求出 $f(XY)$，这样的情况下，就需要考虑将所需要的辅助信息加入到 $f(X)$ 中，这一章将主要考察这一部分。

### 例子1: 第二最小值
> [!note]
>
> **【问题6】（区间第二小值）**
>
> 给定序列 $A = (A_0,A_1,\ldots,A_{N-1})$。处理以下的 $Q$ 次查询，查询有两种。
>
> 1. 给定整数 $0\le i \lt N$ 和 $x$：$A_i := x$.
> 2. 给定整数 $0\le L \lt R \le N$，输出将 $A_L,A_{L+1},\dots,A_{R-1}$ 升序排列后的第二个值.

如果我们将序列 $X$ 的第二小值设为 $f(X)$，但是只通过 $X,Y$ 的第二小值，不可以求出 $XY$ 的第二小值。

而 $XY$ 的第二小值会从下面四种中得到：

- $X$ 的最小
- $X$ 的第二小
- $Y$ 的最小
- $Y$ 的第二小

所以可以这样设计：

```cpp
struct S{
  int min, min2;
};

S op(S x, S y) {
  S z;
  if(x.min < y.min) {
    z.min = x.min;
    z.min2 = min(x.min2, y.min);
  } else {
    z.min = y.min;
    z.min2 = min(x.min, y.min2);
  }
  return z;
}

S e() {return {inf, inf};}
```

在这个例子里，想要的信息是 `min2`，但是为了让其变成可以合并的信息，所以我们添加了 `min`.

### 例子2: 超级大数字？！
> [!note]
>
> **【问题7】（字符串整数）**
>
> 给定由数字 $1,2,\ldots,9$ 序列 $A = (A_0,A_1,\ldots,A_{N-1})$。处理以下的 $Q$ 次查询，查询有两种。
>
> 1. 给定整数 $0\le i \lt N$ 和 $x$：$A_i := x$.
> 2. 给定整数 $0\le L \lt R \le N$，输出将 $A_L,A_{L+1},\dots,A_{R-1}$ 视为一个十进制整数之后所得的值对 $998244353$ 取模之后的结果.

那么肯定知道的是，链接满足下列规则：
$$
\text{val}(XY) = \text{val}(X) \times 10^{|Y|} + \text{val}(Y)
$$
这样一来就可以知道要存储的东西了：
$$
f(X) = (\text{val}(X), 10^{|X|})
$$
显然后面那个也是 $10^{|XY|} = 10^{|X|} \times 10^{|Y|}$，于是显然有转移：

```cpp
using mint = Modint;

struct S{
  mint val, pow10;
};

S op(S x, S y) {
  S z;
  z.val = x.val * y.pow10 + y.val;
  z.pow10 = x.pow10 * y.pow10;
  return z;
}

S e() {return {0, 1};}
```

#### 插曲：滚动哈希

与上面这个问题几乎相同的情况下，有一个重要的问题：字符串的滚动hash.

设 $p$ 为素数，$\text{base}$ 为随机整数，字符当作整数处理的时候，对于一个字符串 $S=S_0S_1\ldots S_{K-1}$，
$$
\text{hash}(S) = \big{(}\sum_{i=0}^{K-1} S_i \times \text{base}^{K-1-i}\big{)} \mod p
$$
观察这个滚动哈希，会发现，当 $S_i$ 是数字、$\text{base}$ 是 $10$ 的时候，滚动哈希等价于 **“把数字串看作是以十进制书写的整数对 $p$ 取余数的结果”**.

当 $p$ 和 $\text{base}$ 为常量时，可以通过滚动哈希与 $\text{base}$ 的幂次的组合进行类似的幺半群设计，从而能够用线段树处理在修改字符的同时求子串滚动哈希这类查询。

滚动hash主要用于字符串领域，用于高概率正确判断字符串是不是一致的.

### 例子3: `ABC` 子序列个数
> [!note]
>
> **【问题8】（`ABC` 个数）**
>
> 给定由字符 $A、B、C$ 序列 $A = (A_0,A_1,\ldots,A_{N-1})$。处理以下的 $Q$ 次查询，查询有两种。
>
> 1. 给定整数 $0\le i \lt N$ 和字符 $x$：$A_i := x$.
> 2. 给定整数 $0\le L \lt R \le N$，输出将 $A_L,A_{L+1},\dots,A_{R-1}$ 的子序列等于 `ABC` 的个数.  等价的，认为满足 $L \le i\lt j\lt k \lt R$ 的整数元组 $(i,j,k)$ 中，$A_i,A_j,A_k$ 按顺序为 `A`、`B`、`C` 的元组个数.

考虑左边和右边组成的时候，怎么样可以组成一个 `ABC`：

- 从 $X$ 选出 `ABC`
- 从 $X$ 选出 `AB`，$Y$ 里选 `C`
- 从 $X$ 选出 `A`，$Y$ 里选 `BC`
- 从 $Y$ 选出 `ABC`

所以显然的，一个 $f(X)$ 要存储这几个量的个数：`ABC`, `AB`, `A`, `C`, `BC`

但是，你会发现，得到 `AB` 和 `BC` 还需要 `B`，于是加入 `B` 之后就形成闭包了.

这样一来就设计完成了！

```cpp
struct S {
	AR<mm, 3> c;
	mm RG = 0, GB = 0, ans = 0;
	S() : c({0, 0, 0}), RG(0), GB(0), ans(0) {}
	S(int i, ll v) : RG(0), GB(0), ans(0) {
		int id = (T[i] == 'R' ? 0 : (T[i] == 'G' ? 1 : 2));
		c = {0, 0, 0};
		c[id] = v;
	}
};

S op(S l, S r) {
	S now;
	rep(i, 3) now.c[i] = l.c[i] + r.c[i];
	now.RG = l.c[0] * r.c[1] + l.RG + r.RG;
	now.GB = l.c[1] * r.c[2] + l.GB + r.GB;
	now.ans = l.RG * r.c[2] + l.c[0] * r.GB + l.ans + r.ans;
	return now;
}
```

注意我们添加的顺序：
$$
ABC \rightarrow A,AB,BC,C \rightarrow B
$$
按这个顺序，添加了结合所需的信息；最终，也可以认为，在从较小的子序列组装出 `ABC` 这个子序列时，所经过的所有状态都作为区间信息持有.

### 例子3: 最大子串和

> [!note]
>
> **【问题9】（最大字串和）**
>
> 给定由字符 $A、B、C$ 序列 $A = (A_0,A_1,\ldots,A_{N-1})$。处理以下的 $Q$ 次查询，查询有两种。
>
> 1. 给定整数 $0\le i \lt N$ 和字符 $x$：$A_i := x$.
> 2. 给定整数 $0\le L \lt R \le N$，输出将 $A_L,A_{L+1},\dots,A_{R-1}$ 的非空子串之和可能取得的最大值.  等价的，输出满足 $L \le l \lt r \le R$ 的 $(l, r)$ 所对应的 $\sum_{i=l}^{r-1} A_i$ 可能取得的最大值

考虑序列 $X$ 的最大字串和是 $ans(X)$，考虑 $\text{ans}(XY)$ 需要从 $X,Y$ 的什么信息求呢？

不难考虑 $XY$ 的非空子串有下面三类：

- $X$ 的非空子串
- $Y$ 的非空子串
- $X$ 的非空后缀和 $Y$ 的非空前缀的拼接

所以实际上你除了 $\text{ans}(X)$ 以外，还要维护：$\text{prefix}(X),\text{suffix}(X)$.

于是有：
$$
\text{ans}(XY) = \max(\text{ans}(X), \text{ans}(Y),\text{suffix}(X) + \text{prefix}(Y))
$$

$$
\text{prefix}(XY) = \max(\text{prefix}(X),\text{full}(X) + \text{prefix}(Y))
$$

$$
\text{suffix}(XY) = \max(\text{suffix}(X)+\text{full}(Y),\text{suffix}(Y))
$$

所以有代码：

```cpp
struct S {
  long long ans, prefix, suffix, full;
};

S op(S x, S y) {
  S z;
  z.ans = max({x.ans, y.ans, x.suffix + y.prefix});
  z.prefix = max(x.prefix, x.full + y.prefix);
  z.suffix = max(x.suffix + y.full, y.suffix);
  z.full = x.full + y.full;
  return z;
}

S e() {return {-inf, -inf, -inf, 0};}
```

在这个例子中，

$ans⁡⟶prefix⁡,suffix⁡⟶full⁡$

按这个顺序，添加了结合所需的信息.   可以认为是将构建作为答案的部分数组时从小部分开始组合过程中出现的所有状态，都作为区间信息持有.

---

---

## 情形3: 处理函数的复合

---

























