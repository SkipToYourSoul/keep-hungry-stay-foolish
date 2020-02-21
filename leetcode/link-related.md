#### [二进制链表转整数](https://leetcode-cn.com/problems/convert-binary-number-in-a-linked-list-to-integer/)

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

# 普通解法
import math
class Solution:
    def getDecimalValue(self, head: ListNode) -> int:
        result_list = []
        while head is not None:
            result_list.append(head.val)
            head = head.next
        
        result = 0
        arr_length = len(result_list)
        for index in range(0, arr_length):
            opp_index = arr_length - index - 1
            result = result + result_list[opp_index] * int(math.pow(2, index))
        return result
      
# 位运算解法
class Solution:
    def getDecimalValue(self, head: ListNode) -> int:
        result = 0
        while head is not None:
            result = result << 1
            result = result | head.val
            head = head.next
        return result
```

