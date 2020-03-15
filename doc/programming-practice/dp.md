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

