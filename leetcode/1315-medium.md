祖父节点值为偶数的节点和

题目：https://leetcode-cn.com/problems/sum-of-nodes-with-even-valued-grandparent/

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

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

