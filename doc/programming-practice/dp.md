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

