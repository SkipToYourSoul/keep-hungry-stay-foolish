# 一些实战经验

实践才是检验整理的唯一标准。

## No1

Hive查询实战：多行合并、JOIN、爆点计算等

code1：arr中K个元素的和为S，输出有几种解

> 思路：递归

## No2

code1：旋转递增数组的最小值

> 思路：考虑二分，寻求最优解

code2：实际问题：为了统计一天内APP每秒的在线用户数量，每次一个用户下线时会生成一条记录到文件里：记录里有三个字段（上线时间，下线时间，用户名），时间以秒为单位。现在有一个文件包含了当天生成的N条记录（N很大），请设计一个算法根据N条记录统计出当天每秒在线用户的数量。(0<=上线时间&下线时间<=24*3600)

> 思路：利用空间，简化时间

code3：两个递增数组的中位数，能否做到log(M + N)的时间复杂度

> 正常做O(N)，考虑二分解，参考leetcode

code4：给定一堆线段，如（2，5）、（3，6），求线段的覆盖面积（上述两个，覆盖面积为4）

> 思路：1）题目中若给出线段数字不大，考虑使用set；2）排序后正常算，考虑不覆盖、覆盖部分、全覆盖

code5：实际问题：一副扑克牌，判断你手牌是否有同花顺

> 四种花色、13种牌，正常写即可

## No3

code1：一个数组，arr[1] > arr[0], arr[n - 2] < arr[n - 1]，求数组的波峰值（若存在的话）

> 思路：二分

code2：在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

> 思路：关键点是考虑二维数组右上角的数字

code3：我们可以用1 * 2的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2 * 1的小矩形无重叠地覆盖一个2 * n的大矩形，总共有多少种方法？

> 思路：求2x+y=N有几组解，每一组，找出有几种排列。如y=1的情况，那就是C(N, 1)种放置办法

code4：二叉树，给出前序和中序遍历，求后序遍历

> 思路：刷题吧

code5：一个二维矩阵，从左上走到右下，有多少种走法。

> 思路：dp

