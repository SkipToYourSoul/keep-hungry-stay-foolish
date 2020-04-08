找到状态方程式，很重要！

注意边界判断！

#### [322. 零钱兑换](https://leetcode-cn.com/problems/coin-change/)

```python
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        # f(n) = min(f(n - coins[i]) for i in range(len(coins))) + 1
        result = [float("inf")] * (amount + 1)
        result[0] = 0
        for coin in coins:
            for i in range(coin, amount + 1):
                result[i] = min(result[i - coin] + 1, result[i])
        print(result)
        return result[amount] if result[amount] != float("inf") else -1
```

#### [300. 最长上升子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

```python
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        # dp[i] = max(dp[j]) + 1, 0<=j<i, nums[i] > nums[j]
        if len(nums) == 0:
            return 0
        dp = []
        for i in range(len(nums)):
            dp.append(1)
            for j in range(i):
                if nums[i] > nums[j]:
                    dp[i] = max(dp[i], dp[j] + 1)

        return max(dp)
```

# 股票问题

```python
# dp[i][k][0 or 1]: 第i天，至多进行k次交易，手上是否持有股票

dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
              max(   选择 rest  ,             选择 sell      )

解释：今天我没有持有股票，有两种可能：
要么是我昨天就没有持有，然后今天选择 rest，所以我今天还是没有持有；
要么是我昨天持有股票，但是今天我 sell 了，所以我今天没有持有股票了。

dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
              max(   选择 rest  ,           选择 buy         )

解释：今天我持有着股票，有两种可能：
要么我昨天就持有着股票，然后今天选择 rest，所以我今天还持有着股票；
要么我昨天本没有持有，但今天我选择 buy，所以今天我就持有股票了。
```

#### [股票的最大利润](https://leetcode-cn.com/problems/gu-piao-de-zui-da-li-run-lcof/)

```python
# k = 1简化
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        # dp[i][0 or 1]: 在第i天持有/非持有股票的最大利润
        # dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
        # dp[i][1] = max(dp[i-1][1], 0 - prices[i])

        N = len(prices)
        if N == 0:
            return 0

        dp = [[0, 0] for _ in range(N)]
        dp[-1][0] = 0
        dp[-1][1] = float("-INF")
        
        for i in range(N):
            dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
            dp[i][1] = max(dp[i-1][1], 0 - prices[i])
        
        return dp[N - 1][0]
```

更多参考：[https://github.com/labuladong/fucking-algorithm/blob/master/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E7%B3%BB%E5%88%97/%E5%9B%A2%E7%81%AD%E8%82%A1%E7%A5%A8%E9%97%AE%E9%A2%98.md](https://github.com/labuladong/fucking-algorithm/blob/master/动态规划系列/团灭股票问题.md)

