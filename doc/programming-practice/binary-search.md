# 二分查找

二分查找的三种基本情况：查等、左边界、右边界。

二分查找可通用于有序数据的遍历复杂度优化，从O(n)有效降低至O(log(n))。

```python
class Solution:
    def binary_search(self, target, nums):
        left, right = 0, len(nums) - 1
        while left <= right:
            mid = left + (right - left) // 2
            if nums[mid] == target:
                return mid
            elif nums[mid] > target:
                right = mid - 1
            elif nums[mid] < target:
                left = mid + 1
        return -1

    def left_bound(self, target, nums):
        left, right = 0, len(nums) - 1
        while left <= right:
            mid = left + (right - left) // 2
            if nums[mid] == target:
                right = mid - 1
            elif nums[mid] > target:
                right = mid - 1
            elif nums[mid] < target:
                left = mid + 1
        if left >= len(nums) or nums[left] != target:
            return -1
        return left

    def right_bound(self, target, nums):
        left, right = 0, len(nums) - 1
        while left <= right:
            mid = left + (right - left) // 2
            if nums[mid] == target:
                left = mid + 1
            elif nums[mid] > target:
                right = mid - 1
            elif nums[mid] < target:
                left = mid + 1
        if right < 0 or nums[right] != target:
            return -1
        return right
```

