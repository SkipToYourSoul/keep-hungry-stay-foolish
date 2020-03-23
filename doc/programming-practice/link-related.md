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

#### [206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

```python
class Solution:
    def reverseList(self, head: ListNode) -> ListNode:
        pre, cur, nxt = None, head, head
        while cur is not None:
            nxt = cur.next
            cur.next = pre
            pre = cur
            cur = nxt
        return pre
```

#### [92. 反转链表 II](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

```python
class Solution:
    def reverseBetween(self, head: ListNode, m: int, n: int) -> ListNode:
        if not head:
            return None
        cur, prev = head, None
        while m > 1:
            prev = cur
            cur = cur.next
            m, n = m - 1, n - 1
        
        tail, con = cur, prev
        while n:
            third = cur.next
            cur.next = prev
            prev = cur
            cur = third
            n -= 1
        if con:
            con.next = prev
        else:
            head = prev
        tail.next = cur
        return head
```

#### [25. K 个一组翻转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

```python
class Solution:
    def reverseKGroup(self, head: ListNode, k: int) -> ListNode:
        if head is None:
            return None
        a, b = head, head
        for i in range(k):
            if b is None:
                return head
            b = b.next
        newHead = self.reverse(a, b)
        a.next = self.reverseKGroup(b, k)
        
        return newHead
    
    def reverse(self, head, tail):
        pre, cur, nxt = None, head, head
        while cur != tail:
            nxt = cur.next
            cur.next = pre
            pre = cur
            cur = nxt
        return pre
```

#### [876. 链表的中间结点](https://leetcode-cn.com/problems/middle-of-the-linked-list/)

```python
class Solution:
    def middleNode(self, head: ListNode) -> ListNode:
        if head is None:
            return None
        
        slow, fast = head, head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
        return slow
```

#### [面试题 02.06. 回文链表](https://leetcode-cn.com/problems/palindrome-linked-list-lcci/)

```python
class Solution:
    def isPalindrome(self, head: ListNode) -> bool:# find center
        slow, fast = head, head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
        if fast:
            slow = slow.next
        # compare
        left = head
        right = reverse(slow)
        while left and right:
            if left.val != right.val:
                return False
            left = left.next
            right = right.next
        return True
```

