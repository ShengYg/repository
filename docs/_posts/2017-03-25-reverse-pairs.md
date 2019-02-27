---
layout: post
title:  "类似reverse pairs的一系列问题"
date:   2017-03-25 12:00:00 +0800
categories: [algorithms]
tags: [tree, sort]
description: LeetCode 493 Reverse Pairs相关问题
---

**Question:**

[LeetCode 493. Reverse Pairs](https://leetcode.com/problems/reverse-pairs/#/description)

Given an array nums, we call `(i, j)` an important reverse pair if `i < j` and `nums[i] > 2*nums[j]`.

You need to return the number of important reverse pairs in the given array.

**Example1:**

~~~
Input: [1,3,2,3,1]
Output: 2
~~~

## 基本思想：

拆分数组并解决子问题

假设输入数组是`nums[i]`，一共有n个元素。nums[i,j]表示子数组，T[i,j]表示相关的子问题。
常见的构建子问题的方式由两种：

1. `T(i, j) = T(i, j - 1) + C`
1. `T(i, j) = T(i, m) + T(m + 1, j) + C`，其中`m = (i+j)/2`

其中，子问题`C`是关键，常常决定了问题的时间复杂度。通常采用动态规划的方式。

## 本题思路

### I -- 序列递归

递归式可写成：

`T(0, j) = T(0, j - 1) + C`

子问题`C`表示**“在数组nums[i, j-1]中选一个元素，和元素nums[j]构成符合条件的reverse pair”**。既然(p, q)表示一个满足条件的pair， 那么它需要满足两个条件：

1. `p < q`
1. `nums[p] > 2 * nums[q]`

对于子问题`C`，第一个条件是显然满足的，只需要考虑第二个条件。

最直接的方法是对子数组现行扫描，找到满足条件的pair。当然，对于递归问题，它的时间复杂度是`O(n^2)`。

为了提高搜索的效率，关键是要发现子数组的顺序其实是不重要的，因为我们只在乎符合条件的pair。这启发我们在线性扫描的时候对子数组进行排序，然后采用二分搜索。然而，问题在于数组是不断增加的，因此后来插入新的元素是，耗费的时间会增加。因此，我们要兼顾搜索和查找代价。因此优先考虑`BIT`和`BST`。

#### BST

BST要满足平衡树（`Red-black`，`AVL`...），否则最坏情况下为`O(n^2)`，很厚可能`TLE`。

#### BIT

> 这类问题采用`BIT`往往速度最快。

~~~java
private int search(int[] bit, int i) {
    int sum = 0;
    
    while (i < bit.length) {
        sum += bit[i];
        i += i & -i;
    }

    return sum;
}

private void insert(int[] bit, int i) {
    while (i > 0) {
        bit[i] += 1;
        i -= i & -i;
    }
}

public int reversePairs(int[] nums) {
    int res = 0;
    int[] copy = Arrays.copyOf(nums, nums.length);
    int[] bit = new int[copy.length + 1];
    
    Arrays.sort(copy);
    
    for (int ele : nums) {
        res += search(bit, index(copy, 2L * ele + 1));
        insert(bit, index(copy, ele));
    }
    
    return res;
}

private int index(int[] arr, long val) {
    int l = 0, r = arr.length - 1, m = 0;
    	
    while (l <= r) {
    	m = l + ((r - l) >> 1);
    		
    	if (arr[m] >= val) {
    	    r = m - 1;
    	} else {
    	    l = m + 1;
    	}
    }
    
    return l + 1;
}
~~~

### II -- 拆分递归

`T(0, n - 1) = T(0, m) + T(m + 1, n - 1) + C, m = (n-1)/2`

子问题`C`表示**“在数组nums[0, m]中选一个元素，在数组nums[m+1, n-1]中选另一个元素，构成符合条件的reverse pair”**。

对于子问题`C`，第一个条件是显然满足的，只需要考虑第二个条件。

最直接的方法是对两个子数组现行扫描，它的时间复杂度是`O(n^2)`。

同样，由于子数组的顺序是不重要的，我们可以对子数组进行排序，然后采用双指针法搜索。归并排序是另一种方法，在排序的同时统计满足条件的reverse pair。

~~~java
public int reversePairs(int[] nums) {
    return reversePairsSub(nums, 0, nums.length - 1);
}
    
private int reversePairsSub(int[] nums, int l, int r) {
    if (l >= r) return 0;
        
    int m = l + ((r - l) >> 1);
    int res = reversePairsSub(nums, l, m) + reversePairsSub(nums, m + 1, r);
        
    int i = l, j = m + 1, k = 0, p = m + 1;
    int[] merge = new int[r - l + 1];
        
    while (i <= m) {
        while (p <= r && nums[i] > 2L * nums[p]) p++;
        res += p - (m + 1);
        	
        while (j <= r && nums[i] >= nums[j]) merge[k++] = nums[j++];
        merge[k++] = nums[i++];
    }
        
    while (j <= r) merge[k++] = nums[j++];
        
    System.arraycopy(merge, 0, nums, l, merge.length);
        
    return res;
}
~~~

## 其他问题

#### [LeetCode 315. Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/#/description)

**Question:**

You are given an integer array nums and you have to return a new counts array. The counts array has the property where `counts[i]` is the number of smaller elements to the right of `nums[i]`.

**Example1:**

~~~
Given nums = [5, 2, 6, 1]

To the right of 5 there are 2 smaller elements (2 and 1).
To the right of 2 there is only 1 smaller element (1).
To the right of 6 there is 1 smaller element (1).
To the right of 1 there is 0 smaller element.

Return the array [2, 1, 1, 0]
~~~

这题比reverse pair更简单

~~~python
from bisect import bisect_left
class Solution(object):
    def countSmaller(self, nums):
        """
        :type nums: List[int]
        :rtype: List[int]
        """
        res = []
        copy = sorted(nums)
        bit = [0] * (len(nums) + 1)
        for item in nums[::-1]:
            idx = bisect_left(copy, item)
            res.append(self.search(bit, idx))
            self.insert(bit, idx + 1)
        return res[::-1]

    def search(self, bit, i):
        sum = 0
        while i > 0:
            sum += bit[i]
            i -= i & -i
        return sum

    def insert(self, bit, i):
        while i < len(bit):
            bit[i] += 1
            i += i & -i
~~~

#### [LeetCode 327. Count of Range Sum](https://leetcode.com/problems/count-of-range-sum/#/description)

**Question:**

Given an integer array nums, return the number of range sums that lie in `[lower, upper]` inclusive.

Range sum `S(i, j)` is defined as the sum of the elements in `nums` between indices `i` and `j` (`i` ≤ `j`), inclusive.

**Example1:**

~~~
Given nums = [-2, 5, -1], lower = -2, upper = 2
Return 3.
The three ranges are : [0, 0], [2, 2], [0, 2] and their respective sums are: -2, -1, 2.
~~~

range sum不适合用`BIT`求解，因为条件限制了上下界，`BIT`求解范围较繁琐。可以仿照归并排序在排序时插入对满足上下界范围的数的查找。

~~~python
from bisect import bisect_left
class Solution(object):
    def countRangeSum(self, nums, lower, upper):
        """
        :type nums: List[int]
        :type lower: int
        :type upper: int
        :rtype: int
        """
        if not nums or lower > upper:
            return 0
        n = len(nums)
        sums = [0] * n
        sum_num = 0
        for i in range(n):
            sum_num += nums[i]
            sums[i] = sum_num
        return self.countRangeSumSub(sums, 0, n-1, lower, upper)

    def countRangeSumSub(self, sums, start, end, lower, upper):
        if start == end:
            return 1 if sums[start] >= lower and sums[end] <= upper else 0
        mid = (start + end) / 2
        a = self.countRangeSumSub(sums, start, mid, lower, upper)
        b = self.countRangeSumSub(sums, mid+1, end, lower, upper)
        count = a + b

        j, k, t = mid+1, mid+1, mid+1
        cache = [0] * (end - start + 1)
        r = 0
        for i in range(start, mid+1):
            while k <= end and sums[k] - sums[i] < lower:
                k += 1
            while j <= end and sums[j] - sums[i] <= upper:
                j += 1
            while t <= end and sums[t] < sums[i]:
                cache[r] = sums[t]
                r += 1
                t += 1
            cache[r] = sums[i]
            count += j - k
            r += 1
        sums[start:t] = cache[0:t-start]
        return count
~~~

~~~java
public int countRangeSum(int[] nums, int lower, int upper) {
    int n = nums.length;
    long[] sums = new long[n + 1];
    for (int i = 0; i < n; ++i)
        sums[i + 1] = sums[i] + nums[i];
    return countWhileMergeSort(sums, 0, n + 1, lower, upper);
}

private int countWhileMergeSort(long[] sums, int start, int end, int lower, int upper) {
    if (end - start <= 1) return 0;
    int mid = (start + end) / 2;
    int count = countWhileMergeSort(sums, start, mid, lower, upper) 
              + countWhileMergeSort(sums, mid, end, lower, upper);
    int j = mid, k = mid, t = mid;
    long[] cache = new long[end - start];
    for (int i = start, r = 0; i < mid; ++i, ++r) {
        while (k < end && sums[k] - sums[i] < lower) k++;
        while (j < end && sums[j] - sums[i] <= upper) j++;
        while (t < end && sums[t] < sums[i]) cache[r++] = sums[t++];
        cache[r] = sums[i];
        count += j - k;
    }
    System.arraycopy(cache, 0, sums, start, t - start);
    return count;
}
~~~

