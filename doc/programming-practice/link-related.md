```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None
```

#### [二进制链表转整数](https://leetcode-cn.com/problems/convert-binary-number-in-a-linked-list-to-integer/)

```python
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

#### [返回倒数第 k 个节点](https://leetcode-cn.com/problems/kth-node-from-end-of-list-lcci/)

```python
# 双指针
class Solution:
    def kthToLast(self, head: ListNode, k: int) -> int:
        slow = fast = head
        i = 0
        while i < k:
            fast = fast.next
            i += 1
        while fast:
            fast = fast.next
            slow = slow.next
        return slow.val
```

#### [删除链表中的节点](https://leetcode-cn.com/problems/delete-node-in-a-linked-list/)

```python
# 把当前节点变成下一个节点
class Solution:
    def deleteNode(self, node):
        """
        :type node: ListNode
        :rtype: void Do not return anything, modify node in-place instead.
        """
        node.val = node.next.val
        node.next = node.next.next
```

#### [从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

```python
class Solution:
    def reversePrint(self, head: ListNode) -> List[int]:
        result = []
        while head:
            result.append(head.val)
            head = head.next
        result.reverse()
        return result
```

