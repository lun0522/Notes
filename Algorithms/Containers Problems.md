# Containers Problems

### 11. Container With Most Water

Given **n** non-negative integers **a1, a2, ..., an**, where each represents a point at coordinate (**i, ai**). **n** vertical lines are drawn such that the two endpoints of line **i** is at (**i, ai**) and (**i, 0**). Find two lines, which together with x-axis forms a container, such that the container contains the most water.

**Note:** You may not slant the container and **n** is at least 2.

![](https://s3-lc-upload.s3.amazonaws.com/uploads/2018/07/17/question_11.jpg)

The above vertical lines are represented by array [1,8,6,2,5,4,8,3,7]. In this case, the max area of water (blue section) the container can contain is 49.

 

**Example:**

> **Input:** [1,8,6,2,5,4,8,3,7]
> 
> **Output:** 49

```python
class Solution:
    def maxArea(self, height):
        """
        :type height: List[int]
        :rtype: int
        """
        ret = 0
        left, right = 0, len(height) - 1
        while left < right:
            hl, hr = height[left], height[right]
            ret = max(ret, (right - left) * min(hl, hr))
            if hl > hr:
                right -= 1
            else:
                left += 1
        return ret
```

If we move the pointer to the higher wall, the width of trapped water will definitely decrease. Even if that pointer then points to a higher wall, the height of trapped water is still determined by the lower wall pointed by another pointer, and the area will only be smaller. So, we always try to move the pointer to the lower wall. 

### 42. Trapping Rain Water

Given **n** non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.

![](http://www.leetcode.com/static/images/problemset/rainwatertrap.png)

The above elevation map is represented by array [0,1,0,2,1,0,1,3,2,1,2,1]. In this case, 6 units of rain water (blue section) are being trapped.

**Example:**

> **Input:** [0,1,0,2,1,0,1,3,2,1,2,1]
> 
> **Output:** 6

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