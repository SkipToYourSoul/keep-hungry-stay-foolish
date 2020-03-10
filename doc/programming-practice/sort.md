#### QuickSort

每一次的切分，将arr切成两部分，这两部分一定是左边一半小于切分点，右边一半大于切分点

```python
class Solution:
    def quickSort(self, arr, low, high):
        if low >= high:
            return
        mid = self.partition(low, high, arr)
        self.quickSort(arr, low, mid - 1)
        self.quickSort(arr, mid + 1, high)

    def partition(self, low, high, arr):
        key = arr[low]
        while low < high:
            while low < high and arr[high] >= key:
                high -= 1
            arr[low] = arr[high]
            while low < high and arr[low] <= key:
                low += 1
            arr[high] = arr[low]
        arr[low] = key
        return low

if __name__ == "__main__":
    solution = Solution()
    arr = [3,1,2,5,4]
    solution.quickSort(arr, 0, len(arr) - 1)
    print("result, arr={0}".format(arr))
```

