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

#### [寻找两个有序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)

使用二分的思想，将每次排除的数尽量的多，而不是一个个的排除。

```python
class Solution:
    def findMedianSortedArrays(self, nums1: List[int], nums2: List[int]) -> float:
        M = len(nums1)
        N = len(nums2)
        # 保证M < N，去进行之后的操作
        if M > N:
            return self.findMedianSortedArrays(nums2, nums1)
        
        iMin, iMax = 0, M
        while iMin <= iMax:
            # nums1的切分点
            i = (iMax + iMin) // 2
            # nums2的切分点, i + j = m - i + n - j + 1
            j = (M + N + 1) // 2 - i
            if j != 0 and i != M and nums2[j - 1] > nums1[i]:
                # nums2切分点比nums1要大，说明i要增大，j要减小
                # nums1的前面i个数不会为结果
                iMin = i + 1
            elif i != 0 and j != N and nums1[i - 1] > nums2[j]:
                # 反之nums1切分点大于nums2，说明i要减小，j增大
                iMax = i - 1
            else:
                # 判断边界
                maxLeft = 0
                if i == 0:
                    maxLeft = nums2[j - 1]
                elif j == 0:
                    maxLeft = nums1[i - 1]
                else:
                    maxLeft = max(nums1[i - 1], nums2[j - 1])
                if (M + N) % 2 == 1:
                    # 奇数，直接返回结果
                    return maxLeft
                
                minRight = 0
                if i == M:
                    minRight = nums2[j]
                elif j == N:
                    minRight = nums1[i]
                else:
                    minRight = min(nums1[i], nums2[j])
                return (maxLeft + minRight) / 2
        
        return 0
```

