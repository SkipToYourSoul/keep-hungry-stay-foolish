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

#### 堆排序

1) 构建堆；2）堆顶元素放置最后，重新构建堆

```python
class Solution:
    def heapSort(self, arr):
        def adjustHeap(arr, pos, length):
            tmp = arr[pos]
            print("adjust heap, pos={0}, tmp={1}".format(pos, tmp))
            # 左叶子节点
            node = pos * 2 + 1
            while node < length:
                # 左子节点小于右子节点，指向右子节点
                if node + 1 < length and arr[node] < arr[node + 1]:
                    node += 1
                if arr[node] > tmp:
                    arr[pos] = arr[node]
                    pos = node
                else:
                    break
                node = node * 2 + 1
            arr[pos] = tmp
            print(arr)

        N = len(arr)
        # 构建大顶堆, 从第一个非叶子节点开始
        for i in range(N//2 - 1, -1, -1):
            adjustHeap(arr, i, N)
        print("Complete")
        # 调整堆结构，把堆顶元素放置末尾
        for i in range(N - 1, 0, -1):
            tmp = arr[0]
            arr[0] = arr[i]
            arr[i] = tmp
            adjustHeap(arr, 0, i)
        return arr

if __name__ == '__main__':
    s = Solution()
    arr = [4,6,8,5,9]
    print(s.heapSort(arr))
```

https://www.cnblogs.com/chengxiao/p/6129630.html

#### 归并排序

https://www.cnblogs.com/chengxiao/p/6194356.html