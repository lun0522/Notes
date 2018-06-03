# Binary Search Problems

### 69. Sqrt(x)

Implement **int sqrt(int x)**.

Compute and return the square root of x, where x is guaranteed to be a non-negative integer.

Since the return type is an integer, the decimal digits are truncated and only the integer part of the result is returned.

**Example 1:**

> **Input:** 4
> 
> **Output:** 2

**Example 2:**

> **Input:** 8
> 
> **Output:** 2
> 
> **Explanation:** The square root of 8 is 2.82842..., and since the decimal part is truncated, 2 is returned.

```python
class Solution:
    def mySqrt(self, x):
        """
        :type x: int
        :rtype: int
        """
        if x <= 1:
            return x
        
        left, right = 0, x
        while left < right:
            middle = (left + right) // 2
            midSqr = middle**2
            if midSqr == x:
                return middle
            elif midSqr < x:
                if left == middle:
                    break
                left = middle
            else:
                right = middle
        return left
```

What we want is a `middle` whose square is **equal to or less than** x. If the square of `middle` is not exactly `x`, it still might be the answer, which is why we don't say `left = middle + 1`. Note that `left` and `middle` might be equal, in which case `left = middle` will lead to an infinite loop, so we can `break` there. 

We always return `left` because of "**equal to or less than**".

### 35. Search Insert Position

Given a sorted array and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order.

You may assume no duplicates in the array.

**Example 1:**

> **Input:** [1,3,5,6], 5
> 
> **Output:** 2

**Example 2:**

> **Input:** [1,3,5,6], 2
> 
> **Output:** 1

**Example 3:**

> **Input:** [1,3,5,6], 7
> 
> **Output:** 4

**Example 4:**

> **Input:** [1,3,5,6], 0
> 
> **Output:** 0

```python
class Solution:
    def searchInsert(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        if target > nums[-1]:
            return len(nums)
        
        left, right = 0, len(nums) - 1
        while left < right:
            middle = (left + right) // 2
            if nums[middle] == target:
                return middle
            elif nums[middle] < target:
                if left == middle:
                    break
                left = middle
            else:
                right = middle
        return right
```

Here we want to find an element **equal to or greater than** `target`, so we always return `right`. We also `break` when `left == middle`.