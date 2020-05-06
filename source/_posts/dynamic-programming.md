---
title: 动态规划
toc: true
date: 2020-05-06 13:51:16
tags:
    - 动态规划
categories:
    - 算法
---

动态规划，就是利用历史记录避免重复计算。而这些历史记录，需要使用一些变量来保存，一般是一维数组或者二维数组。

动态规划的三大步骤：
1. 定义数组的元素的含义
2. 找出数组元素之间的关系式
3. 找出初始值


### <div id="_implementation">自顶向下和自底向上</div>
「自顶向下」： 从规模较大的问题 `f(n)` 开始，向下逐渐分解规模，直到 `f(1)` 和 `f(2)` 触底，然后逐层返回答案，这就叫「自顶向下」。

「自底向上」： 从最底下、最简单、问题规模最小的 `f(1)` 和 `f(2)` 开始往上推，直到推到我们想要的答案 `f(n)`。这就是动态规划的思路，也是为什么动态规划一般都脱离递归，而是使用循环迭代完成计算。


## <div id="_examples">示例</div>

### <div id="_coin_exchange">零钱兑换</div>
> 给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 `-1`。

1、定义数组元素的含义

`dp(n)` 表示凑出目标金额所需要的最少硬币数。

2、找出数组元素之间的关系式

由于我们不知道最后一枚硬币的面值，所以需要枚举每个硬币面值并选择其中最小值：`dp(n) = min(dp(amount - n)) + 1`，其中 `n ∈ coins`。


3、找出初始值

当金额为 0 时，表示需要 0 枚硬币即可满足情况；当金额为负数时，显然不存在硬币满足这种这种情况。所以初始值如下：
- dp[0] = 0
- dp[n] = -1 (n < 0)

按照思路，代码如下：
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

## <div id="_references">参考</div>
- [什么是动态规划？](https://www.zhihu.com/question/23995189)
