---
title: 动态规划
toc: true
date: 2020-05-06 13:51:16
tags:
    - 动态规划
categories:
    - 算法
---

***动态规划的一般形式就是求最值。*** 在计算中一般表现为求最长递增子序列呀，最小编辑距离等等。

因为要求最值，所以需要穷举所有可能的答案，然后找出其中的最值。***因此动态规划的核心问题是穷举。***

但是，动态规划的穷举需要满足以下条件：
- 存在重叠子问题
- 具备最优子结构
- 正确的状态转移方程

<!-- more -->

## <div id="_implementation">自顶向下和自底向上</div>
「自顶向下」： 从规模较大的问题 $dp_{(n)}$ 开始，向下逐渐分解规模，直到到达边界条件，然后逐层返回答案，这就叫「自顶向下」。

「自底向上」： 从边界条件开始往上推，直到推到我们想要的答案 $dp_{(n)}$。这就是 ***动态规划*** 的思路，也是为什么动态规划使用循环迭代完成计算而不是使用递归。

解决动态规划问题的步骤：
1、定义子问题
2、写下与子问题相关的递归
3、识别并解决基本案例

动态规划的三大步骤：

一、定义数组元素的含义

二、找出数组元素之间的关系式

三、找出初始条件


## <div id="_1_dimensional_dp">一维 DP</div>
> 问题：给定 `n`，找出将 `n` 写为 1，3，4 和的方式的数量

1、定义子问题

设 `dp(n)` 为使用不同方式将 `n` 表示为为 1，3，4 和的数量

2、找出递归

设想一种可能的解决方案： `n = X1 + X2 + ·· + Xm`

如果 `Xm = 1`，则其余项之和必须为 `n - 1`

因此，以 `Xm = 1` 结尾的和的数量等于 `dp(n-1)`

考虑其它情形（`Xm = 3`，`Xm = 4`）

递归即为： `dp(n) = dp(n-1) + dp(n-3) + dp(n-4)`

解决基础案例：
- `dp(0) = 1`
- 对于所有负数 `n`，`dp(n) = 0`
- 或者，可以设置 `dp(0) = dp(1) = dp(2) = 1` 和 `dp(3) = 2`

到此基本完成了！
```
dp[0] = dp[1] = dp[2] = 1
dp[3] = 2
for(i = 4; i < n; i++) {
    dp[i] = dp[i - 1] + dp[i - 3] + dp[i - 4]
}
```

$$^{2}$$

非常简洁。


## <div id="_examples">示例</div>

### <div id="_climb_stairs">爬楼梯</div>
<blockquote>
假设你正在爬楼梯，需要 n 阶才能到达楼顶。每次可以爬 1 或 2 层台阶，有多少种不同的方法可以爬到楼顶？
<br><br>
注意：给定 n 是一个正整数。
</blockquote>

1、定义数组元素的含义

$dp_{(n)}$ 表示有多少种方式爬过 $n$ 阶楼层到达楼顶。

2、找出数组元素间的关系

根据情况，一次可以爬 1 层台阶，也可以爬 2 层台阶，所以爬到第 n 层台阶时有两种情形：
- 从 n - 1 层台阶爬上来
- 从 n - 2 层台阶爬上来

所以，爬到 n 层台阶的所有方式为： $dp_{(n)} = dp_{(n-1)} + dp_{(n-2)}$

3、找出初始条件

由于 n 为正整数，当 n = 2 时，$dp_{(2)} = dp_{(1)} + dp_{(0)}$，0 层台阶不符合要求，因此初始条件为：
- $dp_{(1)} = 1$
- $dp_{(2)} = 2$

#### <div id="_climb_stairs_using_memo">备忘录递归解法</div>
```kotlin
fun climbStairs(n: Int): Int {
    val memo = HashMap<Int, Int>(n / 2)
    return climbStairsWithMemo(0, n, memo)
}

/**
 * [from] 从哪一层开始开始爬
 * [steps] 台阶数
 */
fun climbStairsWithMemo(from: Int, steps: Int, memo: HashMap<Int, Int>): Int {
    return when {
        // 我已到楼顶(^.^)
        from > steps -> 0
        // 爬到 n 层台阶时，只需要再爬一次即可到达楼顶，此时只有一种爬法
        from == steps -> 1
        // 查询备忘录，达到剪枝效果，快速缩短算法时间复杂度至 O(n)
        memo.contains(from) -> memo[from]!!
        else -> {
            // 自顶向下，逐步解决
            memo[from] = climbStairsWithMemo(from + 1, steps, memo) + climbStairsWithMemo(
                from + 2,
                steps,
                memo
            )
            memo[from]!!
        }
    }
}
```

#### <div id="_climb_stairs_using_dynamic_prgramming">动态规划解法</div>
```kotlin
fun climbStairs(n: Int): Int {
    if (n < 3) return n
    val dp = IntArray(n + 1)
    dp[1] = 1
    dp[2] = 2
    for (i in 3..n) {
        // 自底向上，逐步解决
        dp[i] = dp[i - 1] + dp[i - 2]
    }
    return dp[n]
}
```

### <div id="_coin_exchange">零钱兑换</div>
> 给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 `-1`。

1、定义数组元素的含义

$dp_{(n)}$ 表示凑出目标金额所需要的最少硬币数。

2、找出数组元素之间的关系式

由于我们不知道最后一枚硬币的面值，所以需要枚举每个硬币面值并选择其中最小值：
$$dp_{(n)} = \min{dp_{amount-n}+1}\;\;(n \in coins)$$

3、找出初始条件

当金额为 0 时，无法用硬币组成金额为 `0` 的情况，所以 $dp_{(0)} = 0$。当金额为负数时，显然金额不可能为负数。所以初始值如下：
- $dp_{(0)} = 0$
- $dp_{(n)} = -1\;\;(n < 0)$

#### <div id="_coin_exchange_using_memo">备忘录递归解法</div>
```kotlin
fun coinChange(coins: IntArray, amount: Int): Int {
    val memo = HashMap<Int, Int>()
    fun dp(n: Int): Int {
        return when {
            // 金额小于 0 时无解
            n < 0 -> -1
            // 金额为 0 时需要 0 个硬币
            n == 0 -> 0
            // 查备忘录
            memo.contains(n) -> memo[n]!!
            else -> {
                var minCount = Int.MAX_VALUE
                // 枚举币面值，选择其中最小硬币数
                for (coin in coins) {
                    // 最后一枚硬币的币面值为 coin 时，计算剩余金额所需要的最少硬币数
                    val minCountForRestAmount = dp(n - coin)
                    // 是否存在最优解
                    if (minCountForRestAmount < 0) {
                        continue
                    }
                    // minCountForRestAmount + 1 是使用当前 coin 作为最后一枚币面值时的最少硬币数
                    minCount = min(minCount, minCountForRestAmount + 1)
                }
                memo[n] = if (minCount == Int.MAX_VALUE) -1 else minCount
                return memo[n]!!
            }
        }
    }
    return dp(amount)
}
```

#### <div id="_coin_exchange_using_dynamic_programming">动态规划解法</div>
定义 $F_{(i)}$ 为 组成金额 $i$ 所需要的最少硬币数，假设在计算 $F_{(i)}$ 之前，我们已经计算出 $F_{(0)}$ ~ $F_{(i-1)}$ 的答案，那么 $F_{(i)}$ 对应的转移方程为：
$$F_{(i)} = \min_{j=0..n-1}F_{(i - c_{j})} + 1$$

其中 $c_j$ 表示第 $j$ 枚硬币的面值，即我们枚举最后一枚硬币面额为 $c_j$，那么需要从 $i - c_j$ 这个金额的状态 $F_{(i-c_j)}$ 转移过来，再加上当前面额硬币占一枚数量，由于要硬币数量最少，所以 $F_{(i)}$ 为前面能转移过来的状态的最小值加上当前面额硬币数量 `1`。

```kotlin
fun coinChange(coins: IntArray, amount: Int): Int {
    if (coins.isEmpty()) return -1
    val max = amount + 1
    // amount + 1 防止数组越界
    val dp = IntArray(amount + 1) { max }
    dp[0] = 0
    for (i in dp.indices) {
        for (coin in coins) {
            // i - coin < 0 跳过，因为金额不能为负数
            if (i < coin) continue
            dp[i] = min(dp[i], dp[i - coin] + 1)
        }
    }
    return if (dp[amount] > amount) -1 else dp[amount]
}
```


## <div id="_references">参考</div>
- [什么是动态规划？](https://www.zhihu.com/question/23995189)
- [动态规划解题套路框架](https://labuladong.gitbook.io/algo/dong-tai-gui-hua-xi-lie/dong-tai-gui-hua-xiang-jie-jin-jie#er-cou-ling-qian-wen-ti)
