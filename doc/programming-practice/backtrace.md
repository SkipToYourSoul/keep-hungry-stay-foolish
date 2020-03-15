#### [八皇后](https://leetcode-cn.com/problems/eight-queens-lcci/)

注意回溯的使用，巧妙的使用数组来判断是否可以放置皇后。

```python
import copy

class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        result = []
        def is_not_under_attack(row, col):
            return not (rows[col] or line1[row + col] or line2[row - col])
        def put_queen(row, col):
            rows[col], line1[row + col], line2[row - col] = 1, 1, 1
        def remove_queen(row, col):
            rows[col], line1[row + col], line2[row - col] = 0, 0, 0

        def backtrack(row, one_result):
            for col in range(n):
                Q = '.' * col + 'Q' + '.' * (n - col - 1)
                if is_not_under_attack(row, col):
                    put_queen(row, col)
                    print(Q)
                    one_result.append(Q)
                    if row == n - 1:
                        result.append(copy.deepcopy(one_result))
                    else:
                        backtrack(row + 1, one_result)
                    remove_queen(row, col)
                    one_result.pop()
        
        rows = [0] * n
        line1 = [0] * (2 * n - 1)
        line2 = [0] * (2 * n - 1)
        backtrack(0, [])
        
        return result
```

#### [面试题 08.07. 无重复字符串的排列组合](https://leetcode-cn.com/problems/permutation-i-lcci/)

使用回溯的方法求解

```python
class Solution:
    def permutation(self, S: str) -> List[str]:
        result = []
        self.backtrack("", S, result)
        return result

    def backtrack(self, C: str, S: str, result: List[str]):
        if len(C) == len(S):
            result.append(C)
            return
        for i in range(len(S)):
            if C.find(S[i]) == -1:
                self.backtrack(C + S[i], S, result) 
```

# 搞懂排列、组合、子集的问题

了解回溯的思想

#### [78. 子集](https://leetcode-cn.com/problems/subsets/)

```python
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        result = [[]]
        for i in range(0, len(nums)):
            for j in range(0, len(result)):
                result.append(result[j] + [nums[i]])

        return result
```

