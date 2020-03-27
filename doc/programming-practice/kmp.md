# 字符串匹配

使用类似状态机的机制实现KMP算法，便于理解。

```python
# 模拟状态机的方式表述和实现KMP算法
class KMP:
    # 状态方程式，dp[状态][字符] = 下个状态
    # 状态表达：0 -A- 1 -B- 2 -A- 3 -B- 4 -C- 5, ABABC子串匹配时，状态从0 - 5，表示完全匹配
    dp = [[]]
    pat = ""

    def __init__(self, pat):
        M = len(pat)
        self.pat = pat
        # ASCII码, 0-256
        self.dp = [[0] * 256 for i in range(M)]
        # 初始状态，只有遇到pat的第一个字符，状态才变为1
        self.dp[0][ord(pat[0])] = 1

        X = 0
        for j in range(1, M):
            for c in range(0, 256):
                # 当前状态j对某个字符的跳动状态，等于影子状态位X对某个字符的跳动状态，其中X和j拥有相同的字符前缀
                self.dp[j][c] = self.dp[X][c]
            # 在j位置与pat的字符匹配，状态更新为j+1
            self.dp[j][ord(pat[j])] = j + 1
            # 当前是状态 X，遇到字符 pat[j]，pat 应该转移到哪个状态？
            # 状态 X 总是落后状态 j 一个状态，与 j 具有最长的相同前缀
            X = self.dp[X][ord(pat[j])]

    def search(self, txt):
        M = len(self.pat)
        N = len(txt)

        # 初始化状态0
        j = 0
        for i in range(N):
            # 计算 pat 的下一个状态
            j = self.dp[j][ord(txt[i])]
            if j == M:
                return i - M + 1
        return -1


if __name__ == '__main__':
    kmp = KMP("ABABA")
    print(kmp.search("CABAABABAC"))
```

# 参考文献

[https://github.com/labuladong/fucking-algorithm/blob/master/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E7%B3%BB%E5%88%97/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E4%B9%8BKMP%E5%AD%97%E7%AC%A6%E5%8C%B9%E9%85%8D%E7%AE%97%E6%B3%95.md](https://github.com/labuladong/fucking-algorithm/blob/master/动态规划系列/动态规划之KMP字符匹配算法.md)