#### [面试题 08.09. 括号](https://leetcode-cn.com/problems/bracket-lcci/)

```python
class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        result = []
        def recursion(left, right, tmp_str):
            if left == right == n:
                result.append(tmp_str)
            if left < n:
                recursion(left + 1, right, tmp_str + "(")
            if right < left:
                recursion(left, right + 1, tmp_str + ")")
        recursion(0, 0, "")
        return result
```

总结：

递归结束条件：左括号数 = 右括号数 = n

括号添加条件：左括号数 < n时，可以添加左括号；右括号数 < 左括号数时，可以添加右括号