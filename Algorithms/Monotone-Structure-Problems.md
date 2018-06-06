# Monotone Structure Problems

### 84. Largest Rectangle in Histogram

Given n non-negative integers representing the histogram's bar height where the width of each bar is 1, find the area of largest rectangle in the histogram.

![](https://leetcode.com/static/images/problemset/histogram.png)

Above is a histogram where width of each bar is 1, given height = **[2,1,5,6,2,3]**.

![](https://leetcode.com/static/images/problemset/histogram_area.png)

The largest rectangle is shown in the shaded area, which has area = **10** unit.

**Example:**

> **Input:** [2,1,5,6,2,3]
> 
> **Output:** 10

```python
class Solution:
    def largestRectangleArea(self, heights):
        """
        :type heights: List[int]
        :rtype: int
        """
        stack = [[-1, 0]]  # [index, height]
        heights.append(0)  # so that stack get cleaned at the end
        maxArea = 0
        for i, h in enumerate(heights):
            if h > stack[-1][1]:  # if ascending
                stack.append([i, h])
            elif h < stack[-1][1]:  # if desending
                while stack[-1][1] > h:
                    index, height = stack.pop()
                    maxArea = max(maxArea, height * (i - index))
                if stack[-1][1] != h:
                    stack.append([index, h])  # inherit index
        return maxArea
```

We maintain a monotone increasing stack. If the next bar is higher than the previous bar stored in the stack, then if we draw a rectangle that has the same height as the previous bar, it can definitely **extend** to the current bar. As shown in the example histogram, when we come to **6**, which is greater than **5**, the rectangle of height **5** can be extended here.

Otherwise, if the current bar is lower the previous one, for example, we encounter a **6** after a **2**, then the rectangle of height **6** has to stop here, so we start to pop indices and heights out and calculate area, until the last height in the stack is lower than the current one.

Note that if two adjacent bars have the same height, only the first  matters, so we do nothing when `h == stack[-1][1]`. When we record a decreasing element, the rectangle of the same height can actually start from the leftmost bar that is higher than or equal to it. For example, a rectangle of height **2** can start from the bar **5**. So, when we record it, we **"inherit" the index** from the last popped out bar. That bar may have the same height as the current bar (*e.g.* **[0,2,5,6,2,0]**), in which case we don't have to record the current one.

### 239. Sliding Window Maximum

Given an array nums, there is a sliding window of size **k** which is moving from the very left of the array to the very right. You can only see the **k** numbers in the window. Each time the sliding window moves right by one position. Return the max sliding window.

**Example:**

> **Input:** nums = [1,3,-1,-3,5,3,6,7], and k = 3
> 
> **Output:** [3,3,5,5,6,7] 
> 
> **Explanation:**
> 
> Window position                Max
> 
> ---------------               -----
> 
> [1  3  -1] -3  5  3  6  7       **3**
> 
>  1 [3  -1  -3] 5  3  6  7       **3**
> 
>  1  3 [-1  -3  5] 3  6  7       **5**
> 
>  1  3  -1 [-3  5  3] 6  7       **5**
> 
>  1  3  -1  -3 [5  3  6] 7       **6**
> 
>  1  3  -1  -3  5 [3  6  7]      **7**

**Note:**
 
You may assume **k** is always valid, 1 ≤ **k** ≤ input array's size for non-empty array.

**Follow up:**

Could you solve it in linear time?

```python
from heapq import *

class Solution:
    def maxSlidingWindow(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: List[int]
        """
        # monotone decreasing queue
        # queue[0] always points to the greatest element
        queue = collections.deque()
        ret = []
        
        for i in range(len(nums)):
            if queue and queue[0] < i - k + 1:
                queue.popleft()
            
            while queue and nums[queue[-1]] <= nums[i]:
                queue.pop()
            
            queue.append(i)
            if i >= k - 1:
                ret.append(nums[queue[0]])
        
        return ret
```

If we simply traverse the window to find the maximum every time after the window moves, the time complexity will be **O(N·K)**. We can use a max-heap to keep track of the maximum, but removing elements from the heap is **O(K)**, so the time complexity doesn't change. We can also use a backlist and do lazy removal, which makes the time complexity **O(N·logK)**.

To solve it in linear time, we maintain a monotone decreasing queue, so that the maximum of the window will always be the first element of the queue. When the window moves:

1. pop out element at the beginning of the queue
2. pop out all elements at the end of the queue that are less than or equal to the new element, since they have no chance to be the maximum because of it
3. add the new element to the queue, after which the queue is guaranteed to have at least one element
4. now `queue[0]` is the index of the maximum in the window