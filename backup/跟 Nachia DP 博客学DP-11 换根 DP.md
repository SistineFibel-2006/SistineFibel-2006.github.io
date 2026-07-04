### 跟 Nachia DP 博客学DP-11 ：换根 DP

将自底向上的树形 DP 的结果对任意根节点一次性计算的 DP 法，叫做**全方位树形 DP（换根 DP）**

从叶子向根计算树 DP 后，对于每个顶点 $v$，将 $v$ 的子节点排成一列并双向计算累积值，这样按照从根到叶子的顺序，就能以每个顶点常数次合并的方式实现与树 DP 相同的聚合。

#### 什么是全方位树形 DP？

> [!tip]
>
> **[定义1]**
>
> 对于某些可s以通过从每个点进行搜索来解决的问题，如果满足特定条件，则可以通过该算法以相同的计算量求出所有顶点的答案.   

首先让我们来看个例题

> [!note]
>
> 给定一个包含 $N$ 个顶点的树，对于每个顶点，求出从该顶点到最远顶点的距离（经过的边数）
>
> **[约束]**：要求复杂度小于 $O(N^2)$

朴素的 $O(N^2)$ 的解法很好思考，$O(N)$ 遍历所有顶点，每个顶点 $O(N)$ 搜索整棵树即可

考虑优化的过程其实与这个朴素的解法有紧密的联系：如果将搜索理解是**「求解树结构的值」**，那么我们可以进一步将其理解为**「计算以自身子节点为根的子树的值，并合并这些结果」**.

下图用方框表示子树，可以看出，每个子树都是由「以自身为子节点为根的子树」与「根」合并而成的.

![img](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F158134%2F660a407e-24c7-d7db-6434-060fb6047f23.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d1f205667319bdae2d416fe880783d45)

在此问题中，我们考虑对每个子树求解「从子树根节点到最远顶点的距离」这一问题。这与整体问题相同。在合并子树时，可以通过「将子树的解的最大值加上根节点的深度」这一操作来求解。实现该功能的代码如下所示。

```cpp
int dfs(int u, int fa = -1) {
    int ans = 0;
    each(v, e[u]) if(v != fa) {
        chmax(ans, dfs(v, u) + 1);
    }
    return ans;
}
```

但是这个显然不是 $O(N)$ 的对吧，加上记忆化才是完整的

```cpp
void dfs(int u, int fa = -1) {
    if(dp[u][fa] != -1) return dp[u][fa];
    int ans = 0;
    each(v, e[u]) if(v != fa) {
        ans = max(ans, dfs(v, u) + 1);
    }
    return dp[u][fa] = ans;
}
```

看上去很优美，对吧

![img](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F158134%2Fff829749-6e35-ecca-6d1e-1119c49b4135.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=73582d17757fd101763aabe17123e3ef)

如果对于这样一个菊花图来说，你会发现，如果我们只提取对`ans`有影响的部分，就是：

```cpp
int ans = 0;
chmax(ans, dp[root][0]);
chmax(ans, dp[root][1]);
chmax(ans, dp[root][2]);
chmax(ans, dp[root][3]);
chmax(ans, dp[root][4]);
chmax(ans, dp[root][5]);
chmax(ans, dp[root][6]);
```

实际上就是在求： $\max_{v \in e[u]} dp[u][v]$

而其余的也可以如下图所示：

![img](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F158134%2F791145b6-44f6-1609-2ec4-c129e541b99d.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f1e77e86850bf5712fb31e9054aba746)

形式化的讲：「先计算其子子树从左往右/从右往左的累计`MAX`」，然后对于所有父方向，合并对应的值即可

边总数是 $N - 1$ 条，每个边两段顶点各被计算一次，所有邻接顶点数的和就是 $2(N - 1) \in O(N)$ 个

**实际上不难发现的是，当操作是一个幺半群的时候，就可以使用 换根 DP**

幺半群：有单位元，每个元素有逆元且可逆，满足交换律

你可以理解为，正常的 树形 DP 是 「按照顶点」对应子树，因为本身 DP 时包含方向这个信息；而 换根 DP 需要「按有向边」的角度来思考

#### 模板

用上面取 `MAX` 的例子来举例模板代码吧：

```cpp
template <class E, class V, E (*merge)(E, E), E (*e)(), E (*add_edge)(V, int), V (*add_root)(E, int)>
struct RerootingDP {
    struct Edge { int to, idx, rev; };
    
    int n;
    vector<vector<Edge>> g;
    vector<vector<E>> dp;
    vector<V> ans;
    
    RerootingDP(int n) : n(n) {
        g.resize(n);
        dp.resize(n);
        ans.resize(n);
    }
    
    void add_edge(int u, int v) {
        g[u].push_back({v, 0, (int)g[v].size()});
        g[v].push_back({u, 0, (int)g[u].size() - 1});
    }
    
    vector<V> build(int root = 0) {
        vector<int> parent(n, -1), order;
        order.reserve(n);
        
        stack<int> st;
        st.push(root);
        parent[root] = -2;
        while (!st.empty()) {
            int u = st.top(); st.pop();
            order.push_back(u);
            for (auto& e : g[u]) {
                if (e.to == parent[u]) continue;
                parent[e.to] = u;
                st.push(e.to);
            }
        }
        
        for (int i = n - 1; i >= 1; i--) {
            int u = order[i];
            int p = parent[u];
            E acc = e();
            for (auto& e : g[u]) {
                if (e.to == p) continue;
                acc = merge(acc, dp[u][e.rev]);
            }
            V val = add_root(acc, u);
            for (auto& e : g[p]) {
                if (e.to == u) {
                    dp[p][e.rev] = add_edge(val, e.idx);
                    break;
                }
            }
        }
        
        ans[root] = add_root(e(), root);
        return ans;
    }
    
    vector<V> reroot(int root = 0) {
        build(root);
        
        for (int u = 0; u < n; u++) {
            int m = g[u].size();
            vector<E> suffix(m + 1, e());
            for (int i = m - 1; i >= 0; i--) {
                suffix[i] = merge(dp[u][i], suffix[i + 1]);
            }
            
            E prefix = e();
            for (int i = 0; i < m; i++) {
                int v = g[u][i].to;
                E other = merge(prefix, suffix[i + 1]);
                V val = add_root(other, u);
                dp[v][g[u][i].rev] = add_edge(val, g[u][i].rev);
                prefix = merge(prefix, dp[u][i]);
            }
            
            ans[u] = add_root(prefix, u);
        }
        
        return ans;
    }
};
```

求每个顶点到最远点的距离：

```cpp
int merge(int a, int b) { return max(a, b); }  // 二元运算 (E \cdot E)
int e() { return 0; } // 单位元 (E id)
int add_edge(int v, int idx) { return v + 1; } // 与新边聚合 (V, int) V: Dp数组的数值类型
int add_root(int e, int u) { return e; } //与当前节点聚合  (E, int)

int main() {
    int n;
    cin >> n;
    RerootingDP<int, int, merge, e, add_edge, add_root> reroot(n);
    for (int i = 0; i < n - 1; i++) {
        int u, v;
        cin >> u >> v;
        reroot.add_edge(u - 1, v - 1, i, i);
    }
    auto ans = reroot.reroot();
    for (int i = 0; i < n; i++) {
        cout << ans[i] << endl;
    }
    return 0;
}
```

#### 例题与其他阅读

##### 例题

[AtCoder EDPC-V](https://atcoder.jp/contests/dp/tasks/dp_v)

[AtCoder TDPC-N](https://atcoder.jp/contests/tdpc/tasks/tdpc_tree)



##### 其他阅读

[木DPと全方位木DPを基礎から抽象化まで解説【競技プログラミング】 | アルゴリズムロジック](https://algo-logic.info/tree-dp/)

[【全方位树 DP】明天就能用的便利树结构算法 #竞技编程 - Qiita --- 【全方位木DP】明日使える便利な木構造のアルゴリズム #競技プログラミング - Qiita](https://qiita.com/keymoon/items/2a52f1b0fb7ef67fb89e)

