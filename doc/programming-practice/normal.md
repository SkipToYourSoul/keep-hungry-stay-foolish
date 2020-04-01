#### [将数据变成0的操作次数](https://leetcode-cn.com/problems/number-of-steps-to-reduce-a-number-to-zero/)

```python
class Solution:
    def numberOfSteps (self, num: int) -> int:
        cnt = 0
        while num > 0:
            if num % 2 == 0:
                num = num/2
            elif num % 2 == 1:
                num = num - 1
            cnt = cnt + 1
        return cnt
```

#### [面试题 08.04. 幂集](https://leetcode-cn.com/problems/power-set-lcci/)

幂集。编写一种方法，返回某集合的所有子集。集合中**不包含重复的元素**。

```python
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        result = [[]]
        for i in range(0, len(nums)):
            temp = []
            for value in result:
                temp.append(value + [nums[i]])
            result.extend(temp)
        return result
```

#### [交换数字](https://leetcode-cn.com/problems/swap-numbers-lcci/)

```python
# 使用异或运算进行数据交换，可以不借助中间变量
class Solution:
    def swapNumbers(self, numbers: List[int]) -> List[int]:
        # a ^ b ^ b = a
        # a ^ b ^ a = b
        
        numbers[0] = numbers[0] ^ numbers[1]
        numbers[1] = numbers[1] ^ numbers[0]
        numbers[0] = numbers[0] ^ numbers[1]

        return numbers
```

#### [面试题 05.07. 配对交换](https://leetcode-cn.com/problems/exchange-lcci/)

```python
class Solution:
    def exchangeBits(self, num: int) -> int:
        numb = '{:b}'.format(num)   #将十进制转换为二进制
        print(numb)
        if len(numb) % 2: numb = '0'+numb
        ans = ''; i = 1
        while i <= len(numb):
            ans += numb[-(i+1)]
            ans += numb[-i]
            i += 2
        ans = ans[::-1]   #将字符串倒置
        return int(ans, 2)
```

#### [数组中数字出现的次数](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

一个整型数组 `nums` 里除两个数字之外，其他数字都出现了两次。请写程序找出这两个只出现一次的数字。要求时间复杂度是O(n)，空间复杂度是O(1)。

```python
class Solution:
    def singleNumbers(self, nums: List[int]) -> List[int]:
        num = 0
        for i in range(len(nums)):
            num = num ^ nums[i]
        # 从右往左第一位是1
        pos = num & (-1 * num)
        result = [0] * 2
        for n in nums:
            # 这个位置是1的数字，放到第一个数组里做异或运算
            if n & pos ==pos:
                result[0] ^= n
            else:
                result[1] ^= n
        return result
```

#### [数组中出现次数超过一半的数字](https://leetcode-cn.com/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/)

```python
class Solution:
    def majorityElement(self, nums: List[int]) -> int:
        vote = 0
        result = nums[0]
        for num in nums:
            if vote == 0:
                result = num
            if result == num:
                vote += 1
            else:
                vote -= 1
        return result
```

