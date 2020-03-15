# 回溯的思想

搞清楚边界条件，满足条件即可结束递归。

```python
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return
    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```

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

## 子集

子集的关键在先用数学归纳法找出规律，subset(n) = subset(n - 1) + [for set in subset(n-1) + [n]]

#### [78. 子集](https://leetcode-cn.com/problems/subsets/)

```python
class Solution:
    def subsets(self, nums):
        result = []
        n = len(nums)

        def backtrace(position, tmp):
            # print("tmp = {0}, position = {1}".format(tmp, position))
            result.append(tmp)
            for i in range(position, n):
                backtrace(i + 1, tmp + [nums[i]])

        backtrace(0, [])
        return result
```

#### [90. 子集 II](https://leetcode-cn.com/problems/subsets-ii/)

```python
class Solution:
    def subsets(self, nums):
        result = []
        n = len(nums)
        nums.sort()

        def backtrace(position, tmp):
            if tmp not in result:
                result.append(tmp)
            for i in range(position, n):
                backtrace(i + 1, tmp + [nums[i]])

        backtrace(0, [])
        return result
```

## 组合

组合问题需要注意判断退出条件。

求组合总和时，先排序。

#### [77. 组合](https://leetcode-cn.com/problems/combinations/)

```python
class Solution:
    def combine(self, n: int, k: int) -> List[List[int]]:
        result = []

        def backtrack(tmp, pos):
            if len(tmp) == k:
                result.append(tmp)
                return
            for i in range(pos, n+1):
                backtrack(tmp + [i], i + 1)
        
        backtrack([], 1)
        return result
```

#### [39. 组合总和](https://leetcode-cn.com/problems/combination-sum/)

```python
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        result = []
        n = len(candidates)
        candidates.sort()

        def backtrack(mid_candidate, mid_target, tmp):
            print("tmp={0}".format(tmp))
            if mid_target == 0:
                result.append(tmp)
            elif mid_target < 0:
                return
            for i in range(len(mid_candidate)):
                if mid_candidate[i] > mid_target:
                    break
                backtrack(mid_candidate[i:], mid_target - mid_candidate[i], tmp + [mid_candidate[i]])

        backtrack(candidates, target, [])
        return result
```

#### [40. 组合总和 II](https://leetcode-cn.com/problems/combination-sum-ii/)

```python
class Solution:
    def combinationSum2(self, candidates: List[int], target: int) -> List[List[int]]:
        result = []
        n = len(candidates)
        candidates.sort()

        def backtrack(mid_target, tmp, position):
            print("tmp={0}, target={1}, position={2}".format(tmp, mid_target, position))
            if mid_target == 0:
                result.append(tmp)
            elif mid_target < 0:
                return
            for i in range(position, n):
                if candidates[i] > mid_target:
                    break
                if candidates[i] == candidates[i - 1] and i > position:
                    continue
                backtrack(mid_target - candidates[i], tmp + [candidates[i]], i + 1)

        backtrack(target, [], 0)
        return result
```

## 排列

排列问题在于要判断中间结果是否是已包含关系，避免重复、

#### [46. 全排列](https://leetcode-cn.com/problems/permutations/)

```python
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        result = []
        n = len(nums)

        def backtrack(tmp):
            if len(tmp) == n:
                result.append(tmp)
            for i in range(n):
                if nums[i] not in tmp:
                    backtrack(tmp + [nums[i]])
        
        backtrack([])
        return result
```

#### [47. 全排列 II](https://leetcode-cn.com/problems/permutations-ii/)

```python
class Solution:
    def permuteUnique(self, nums: List[int]) -> List[List[int]]:
        result = []
        n = len(nums)
        visit = [0] * n
        nums.sort()

        def backtrack(tmp):
            if len(tmp) == n:
                result.append(tmp)
            for i in range(n):
                # 两个数相同的情况下，规定有序，避免重复结果出现
                # 这个步骤相当于节省了 tmp not in result 的判断，可以提高运行速度
                if visit[i] == 1 or (i > 0 and nums[i] == nums[i - 1] and not visit[i - 1]):
                    continue
                visit[i] = 1
                backtrack(tmp + [nums[i]])
                visit[i] = 0
        
        backtrack([])
        return result
```

