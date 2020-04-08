#### [机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

尝试使用BFS或DFS解决问题

```python
# bfs方法，使用queue来记录搜索点
# si + 1 if (i + 1) % 10 else si - 8，该条件仅适合0<n<100，通用函数应该是：
def sums(x):
    s = 0
    while x:
        s += x % 10
        x = x // 10
    return s

class Solution:
    def movingCount(self, m: int, n: int, k: int) -> int:
        queue, visited,  = [(0, 0, 0, 0)], set()
        # bfs with queue
        while queue:
            i, j, si, sj = queue.pop(0)
            if not i < m or not j < n or k < si + sj or (i, j) in visited: continue
            visited.add((i,j))
            queue.append((i + 1, j, si + 1 if (i + 1) % 10 else si - 8, sj))
            queue.append((i, j + 1, si, sj + 1 if (j + 1) % 10 else sj - 8))
        return len(visited)
   
    # dfs
    def movingCount(self, m: int, n: int, k: int) -> int:
        def dfs(i, j, si, sj):
            if not 0 <= i < m or not 0 <= j < n or k < si + sj or (i, j) in visited: return 0
            visited.add((i,j))
            return 1 + dfs(i + 1, j, si + 1 if (i + 1) % 10 else si - 8, sj) + dfs(i, j + 1, si, sj + 1 if (j + 1) % 10 else sj - 8)

        visited = set()
        return dfs(0, 0, 0, 0)
```

