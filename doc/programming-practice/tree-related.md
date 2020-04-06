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
```

#### [特定深度节点链表](https://leetcode-cn.com/problems/list-of-depth-lcci/)

```python
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

文献：

二叉树遍历：https://blog.csdn.net/lixinyu306/article/details/83931171

#### [面试题27. 二叉树的镜像](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)

```python
class Solution:
    def mirrorTree(self, root: TreeNode) -> TreeNode:
        if root is None:
            return None
        # layer scan
        queue = []
        queue.append(root)
        while len(queue) > 0:
            node = queue.pop(0)
            tmp = node.left
            node.left = node.right
            node.right = tmp
            if node.left is not None:
                queue.append(node.left)
            if node.right is not None:
                queue.append(node.right)
        return root
    # 递归
    def mirrorTree(self, root: TreeNode) -> TreeNode:
        if root is None:
            return None
        tmp = root.left
        root.left = root.right
        root.right = tmp
        self.mirrorTree(root.left)
        self.mirrorTree(root.right)
        return root
```

#### [最小高度树](https://leetcode-cn.com/problems/minimum-height-tree-lcci/)

```python
# 参考二分查找的思想
class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> TreeNode:
        tree = self.create(0, len(nums) - 1, nums)
        return tree

    def create(self, left, right, nums):
        if left <= right:
            mid = (right + left) // 2
            print("left={}, right={}, mid={}".format(left, right, mid))
            tree_node = TreeNode(nums[mid])
            tree_node.left = self.create(left, mid-1, nums)
            tree_node.right = self.create(mid+1, right, nums) 
            return tree_node
        else:
            return None
```

#### [二叉树的深度](https://leetcode-cn.com/problems/er-cha-shu-de-shen-du-lcof/)

```python
class Solution:
    def maxDepth(self, root: TreeNode) -> int:
        if root is None:
            return 0
        return max(self.maxDepth(root.left), self.maxDepth(root.right)) + 1
```

#### [层数最深叶子节点的和](https://leetcode-cn.com/problems/deepest-leaves-sum/)

```python
class Solution:
    result = 0
    max_depth = -1
    
    def deepestLeavesSum(self, root: TreeNode) -> int:
        self.dfs(root, 1)
        return self.result
    
    def dfs(self, root, depth):
        if root is None:
            return
        if depth > self.max_depth:
            self.max_depth = depth
            self.result = root.val
        elif depth == self.max_depth:
            self.result = self.result + root.val
        self.dfs(root.left, depth + 1)
        self.dfs(root.right, depth + 1)
```

#### [最大二叉树](https://leetcode-cn.com/problems/maximum-binary-tree/)

```python
class Solution:
    def constructMaximumBinaryTree(self, nums: List[int]) -> TreeNode:
        return self.create(0, len(nums) - 1, nums)

    def create(self, left, right, nums):
        if left <= right:
            mid = self.getMidIndex(left, right, nums)
            # build tree
            root = TreeNode(nums[mid])
            root.left = self.create(left, mid - 1, nums)
            root.right = self.create(mid + 1, right, nums)
            return root
        else:
            return None
    
    def getMidIndex(self, left, right, nums):
        max_num = nums[left]
        mid = left
        for i in range(left, right + 1):
            if nums[i] > max_num:
                max_num = nums[i]
                mid = i
        return mid
```

#### [面试题54. 二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)

```python
class Solution:
    def kthLargest(self, root: TreeNode, k: int) -> int:
        if root is None:
            return None
        arr = []
        def dfs(root):
            if root is None:
                return
            dfs(root.left)
            # lcr
            arr.append(root.val)
            dfs(root.right)
        dfs(root)
        N = len(arr)
        return arr[N - k]
```

#### [二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

```python
class Solution:
    def lowestCommonAncestor(self, root: 'TreeNode', p: 'TreeNode', q: 'TreeNode') -> 'TreeNode':
        if (root.val - p.val) * (root.val - q.val) <= 0:
            return root
        if root.val > p.val:
            return self.lowestCommonAncestor(root.left, p, q)
        if root.val < p.val:
            return self.lowestCommonAncestor(root.right, p, q)
        return None
```

#### [二叉树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

```python
class Solution:
    def lowestCommonAncestor(self, root: TreeNode, p: TreeNode, q: TreeNode) -> TreeNode:
        if root is None or root == p or root == q:
            return root
        
        left = self.lowestCommonAncestor(root.left, p, q)
        right = self.lowestCommonAncestor(root.right, p, q)

        if left is None:
            return right
        if right is None:
            return left
        
        return root
```

