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

若是有重复字符串，则需要稍微改进一下

```python
class Solution:
    def permutation(self, S: str) -> List[str]:
        result = []
        has_result = set()
        self.backtrack("", S, result, has_result)
        return result

    def backtrack(self, C: str, S: str, result: List[str], has_result):
        if len(C) == len(S) and C not in has_result:
            result.append(C)
            has_result.add(C)
            return
        for i in range(len(S)):
            if C.find(S[i]) == -1:
                self.backtrack(C + S[i], S, result, has_result) 
```

