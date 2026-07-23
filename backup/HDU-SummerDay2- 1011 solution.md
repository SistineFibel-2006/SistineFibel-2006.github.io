# HDU-SummerDay2- 1011

## [1011 键盘杀手](https://acm.hdu.edu.cn/contest/problem?cid=1230&pid=1011)

> [!note]
>
> 长度为 $N$ 的序列 $A = (a_1, a_2, \ldots, a_N)$，你可以每次选择一个下标 $i$，然后消耗 $max(a_{i-1}, a_{i+1})$ 来让 $a_i = 0$，问将所有数字变成 $0$ 的最小消耗
>
> **「约束」**
>
> - $1 \le N \le 10^5$
> - $1 \le a_i \le 10^9$

首先找到建模，对于一个数字 $a_i$ 而言，每个数字只有一个出度，至多两个入度，于是对于一个点，在考虑拔出顺序后，有下面四种情况：

- $i-1 \rightarrow i \leftarrow i + 1$ ：左右先被拔，$cost = 0$
- $i-1 \leftarrow i \rightarrow i + 1$ ：左右后被拔，$cost = max(a_{i-1},a_{i+1})$
- $i-1 \rightarrow i \rightarrow i + 1$ ：左先右后，$cost = a_{i+1}$
- $i-1 \leftarrow i \leftarrow i + 1$ ：左后右先，$cost = a_{i-1}$

我们可以围绕第 $i$ 个箭头方向来 DP，$f_i$ 表示由 $i - 1 \leftarrow i$，$g_i$ 表示由 $i-1 \rightarrow i$。

这样以来 $f_{i+1} = i \leftarrow i + 1$ 有两种情况：分别是上面的 1 和 4

然后 $g_{i+1} = i \rightarrow i + 1$ 就是 2 和 3

就可以写出 DP 式子了
$$
f_{i+1} = min(f_i + a_{i-1}, g_i + 0)
$$

$$
g_{i+1} = min(f_i + max(a_{i-1}, a_{i+1}), g_i + a_{i+1})
$$

