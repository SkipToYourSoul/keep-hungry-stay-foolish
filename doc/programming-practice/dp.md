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

dp[-1][k][0] = 0
解释：因为 i 是从 0 开始的，所以 i = -1 意味着还没有开始，这时候的利润当然是 0 。
dp[-1][k][1] = -infinity
解释：还没开始的时候，是不可能持有股票的，用负无穷表示这种不可能。
dp[i][0][0] = 0
解释：因为 k 是从 1 开始的，所以 k = 0 意味着根本不允许交易，这时候利润当然是 0 。
dp[i][0][1] = -infinity
解释：不允许交易的情况下，是不可能持有股票的，用负无穷表示这种不可能。

base case：
dp[-1][k][0] = dp[i][0][0] = 0
dp[-1][k][1] = dp[i][0][1] = -infinity

状态转移方程：
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
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

# 打家劫舍

#### [打家劫舍 II](https://leetcode-cn.com/problems/house-robber-ii/)

```python
# dpResult函数则为最基础的动归解
# dp[i] = max(dp[i+1], dp[i+2] + nums[i])
class Solution:
    def rob(self, nums: List[int]) -> int:
        N = len(nums)
        if N == 1:
            return nums[0]
        def dpResult(nums, start, end):
            dp = [0] * (N + 2)
            for i in range(end, start - 1, -1):
                dp[i] = max(dp[i + 1], dp[i + 2] + nums[i])
            return dp[start]
        
        return max(dpResult(nums, 0, N - 2), dpResult(nums, 1, N - 1))
```

#### [打家劫舍 III](https://leetcode-cn.com/problems/house-robber-iii/)

```python
class Solution:
    def rob(self, root: TreeNode) -> int:
        # 返回一个大小为 2 的数组 arr
        # arr[0] 表示不抢 root 的话，得到的最大钱数
        # arr[1] 表示抢 root 的话，得到的最大钱数
        def dp(root):
            if root is None:
                return (0, 0)
            left = dp(root.left)
            right = dp(root.right)
            do_it = root.val + left[0] + right[0]
            not_do = max(left[0], left[1]) + max(right[0], right[1])
            return (not_do, do_it)
        
        result = dp(root)
        return max(result[0], result[1])
```

