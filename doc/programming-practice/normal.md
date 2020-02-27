将数据变成0的操作次数

题目：https://leetcode-cn.com/problems/number-of-steps-to-reduce-a-number-to-zero/

```python
class Solution:
    def numberOfSteps (self, num: int) -> int:
        cnt = 0
        while num > 0:
            if num % 2 == 0:
                num = num/2
            elif num % 2 == 1:
                num = num - 1
            cnt = cnt + 1
        return cnt
```

#### [面试题 08.04. 幂集](https://leetcode-cn.com/problems/power-set-lcci/)

幂集。编写一种方法，返回某集合的所有子集。集合中**不包含重复的元素**。

```python
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        result = [[]]
        for i in range(0, len(nums)):
            temp = []
            for value in result:
                temp.append(value + [nums[i]])
            result.extend(temp)
        return result
```

