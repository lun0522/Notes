# N Pointers Problems

### 42. Trapping Rain Water

Given **n** non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.

![](http://www.leetcode.com/static/images/problemset/rainwatertrap.png)

The above elevation map is represented by array [0,1,0,2,1,0,1,3,2,1,2,1]. In this case, 6 units of rain water (blue section) are being trapped.

Example:

> Input: [0,1,0,2,1,0,1,3,2,1,2,1]
> 
> Output: 6

```python
class Solution:
    def trap(self, height):
        """
        :type height: List[int]
        :rtype: int
        """
        left, right, water = 0, len(height) - 1, 0
        while left < right:
            curH = 0
            if height[left] <= height[right]:
                while left < right and height[left] <= height[right]:
                    if height[left] > curH:  # renew curH
                        curH = height[left]
                    else:  # count water
                        water += curH - height[left]
                    left += 1
            else:
                while left < right and height[left] >= height[right]:
                    if height[right] > curH:  # renew curH
                        curH = height[right]
                    else:  # count water
                        water += curH - height[right]
                    right -= 1
        return water
```

The amount of the water that can be trapped in a certain grid is determined by the highest wall to the left of this grid and the highest wall to the right. We can build arrays `l` and `r` with two traversals, where `l[i]` tracks the largest height to the left of `i`, and `r[i]` tracks the right, so that the water trapped in `i` is: `min(l[i], r[i]) - height[i]`. The space complexity is O(n).

Actually we don't really need to know the heighest wall in both directions. If we know the highest wall to the left is of height `l[i]` and there **exists** a wall to the right **equal to or taller than `l[i]`**, then we can conclude the water trapped in `i` is `l[i] - height[i]`. If `height[i] > l[i]`, then `i` will become the new wall, and we renew the height of the highest wall to the left.

We keep scanning from the left to the right, until the highest wall to the right is not high enough. Then we start to scan from the right, until the wall to the left is not enough, and so on.

### 632. Smallest Range

You have **k** lists of sorted integers in ascending order. Find the **smallest** range that includes at least one number from each of the **k** lists.

We define the range [a,b] is smaller than range [c,d] if **b-a < d-c** or **a < c** if **b-a == d-c**.

Example 1:

> Input:[[4,10,15,24,26], [0,9,12,20], [5,18,22,30]]
> 
> Output: [20,24]
> 
> Explanation: 
> 
> List 1: [4, 10, 15, 24,26], 24 is in range [20,24].
> 
> List 2: [0, 9, 12, 20], 20 is in range [20,24].
> 
> List 3: [5, 18, 22, 30], 22 is in range [20,24].

Note:

1. The given list may contain duplicates, so ascending order means >= here.
2. 1 <= k <= 3500
3. -10e5 <= **value of elements** <= 10e5.

```python
from heapq import *

class Solution:
    def smallestRange(self, nums):
        """
        :type nums: List[List[int]]
        :rtype: List[int]
        """
        maximum = float("-inf")
        heap = []
        for i in range(len(nums)):
            heap.append([nums[i][0], 0, i])
            maximum = max(maximum, nums[i][0])
        heapify(heap)
        
        ret = None
        while True:
            val, ptr, lst = heap[0]
            if not ret or maximum - val < ret[1] - ret[0]:
                ret = [val, maximum]
            ptr += 1
            if ptr == len(nums[lst]):
                break
            maximum = max(maximum, nums[lst][ptr])
            heapreplace(heap, [nums[lst][ptr], ptr, lst])
            
        return ret
```

The example input is **[[4,10,15,24,26], [0,9,12,20], [5,18,22,30]]**. Rewrite it in this way:

|   |   |   |   |    |    |    |    |    |    |    |    |    |
|---|---|---|---|----|----|----|----|----|----|----|----|----|
|   | 4 |   |   | 10 |    | 15 |    |    |    | 24 | 26 |    |
| 0 |   |   | 9 |    | 12 |    |    | 20 |    |    |    |    |
|   |   | 5 |   |    |    |    | 18 |    | 22 |    |    | 30 |

For example, if we want to build a range from 9, we only care about  numbers that are equal to or greater than 9:

|   |    |    |    |    |    |    |    |    |    |
|---|----|----|----|----|----|----|----|----|----|
|   | 10 |    | 15 |    |    |    | 24 | 26 |    |
| 9 |    | 12 |    |    | 20 |    |    |    |    |
|   |    |    |    | 18 |    | 22 |    |    | 30 |

9 belongs to the second list. We just need to find the smallest numbers in the first and the third lists in this cropped table, which are 10 and 18, and use the larger one as the end of the range, so finally we have range [9, 18].

To find 10 and 18, we can do binary search twice on those two lists. But that is not necessary. Suppose that we have already found 10 and 18, and use two pointers pointing to them. Later when we want to use 12 as the begin of the range, we find that 10 is less than 12, so we advance the corresponding pointer, which then points to 15 instead; on the other hand, 18 is already greater than 12, so no move. In this way we make a new range, [12, 18].

It seems that those two pointers can be reused only if we still use a number of the second list as the begin of the range, but we can make it more general, and finish everything in one scan.

We now use three pointers. Initially they point to the first elements of three lists (*i.e.* point to 0, 4 and 5). To form a range, simply find out the smallest and the largest among them (0 and 5). Then we advance the pointer pointing to the smallest one (from 0 to 9), find another range (among 4, 5 and 9), and repeat this until one of pointers reaches the end the list.

To avoid sorting, we can use a heap. A heap cannot keep track of both the smallest and the largest number, so in the solution, `heap` tracks **the next smallest number inside of a list, the index of that number, and the index of the list**, and `maximum` tracks **the largest number inside of `heap`**. When `heap` is updated, remember to update `maximum`.