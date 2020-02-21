#### [特定深度节点链表](https://leetcode-cn.com/problems/list-of-depth-lcci/)

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def listOfDepth(self, tree: TreeNode) -> List[ListNode]:
        result = []
        queue = []
        queue.append(tree)
        while len(queue) > 0:
            size = len(queue)
            first_node = ListNode(0)
            temp_node = first_node
            for i in range(size):
                node = queue.pop(0)
                temp_node.next = ListNode(node.val)
                temp_node = temp_node.next
                if node.left is not None:
                    queue.append(node.left)
                if node.right is not None:
                    queue.append(node.right)
            result.append(first_node.next)
        return result
```

总结：

1. 二叉树和列表的基础结构定义要掌握
2. BFS（通过queue实现）
3. DFS（通过stack或递归）

#### [祖父节点值为偶数的节点和](https://leetcode-cn.com/problems/sum-of-nodes-with-even-valued-grandparent/)

```python
class Solution:
    def sumEvenGrandparent(self, root: TreeNode) -> int:
        return self.layerScan(root)
    
    def layerScan(self, root):
        if root is None:
            return 0
        else:
            return self.subChildrenValue(root) + self.layerScan(root.left) + self.layerScan(root.right)
    
    def subChildrenValue(self, root):
        val = 0
        if root is None or root.val % 2 == 1:
            return val
        if root.left is not None:
            if root.left.left is not None:
                val = val + root.left.left.val
            if root.left.right is not None:
                val = val + root.left.right.val
        if root.right is not None:
            if root.right.left is not None:
                val = val + root.right.left.val
            if root.right.right is not None:
                val = val + root.right.right.val
        return val
```
