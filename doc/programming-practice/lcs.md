# 最长公共子序列

以动态规划的思想考虑问题。

核心是动态规划转移方程。

```python
def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        # dp[i][j] = dp[i-1][j-1] + 1 if text1[i] == text2[j] else max(dp[i-1][j], dp[i][j-1])
        M = len(text1)
        N = len(text2)
        dp = [[0] * (N + 1) for _ in range(M + 1)]

        for i in range(1, M + 1):
            for j in range(1, N + 1):
                dp[i][j] = dp[i-1][j-1] + 1 if text1[i-1] == text2[j-1] else max(dp[i-1][j], dp[i][j-1])
        
        return dp[M][N]
```

