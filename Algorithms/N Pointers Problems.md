# N Pointers Problems

### 632. Smallest Range

You have **k** lists of sorted integers in ascending order. Find the **smallest** range that includes at least one number from each of the **k** lists.

We define the range [a,b] is smaller than range [c,d] if **b-a < d-c** or **a < c** if **b-a == d-c**.

**Example:**

> **Input:**[[4,10,15,24,26], [0,9,12,20], [5,18,22,30]]
> 
> **Output:** [20,24]
> 
> **Explanation:**
> 
> List 1: [4, 10, 15, 24,26], 24 is in range [20,24].
> 
> List 2: [0, 9, 12, 20], 20 is in range [20,24].
> 
> List 3: [5, 18, 22, 30], 22 is in range [20,24].

**Note:**

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
|   | **4** |   |   | 10 |    | 15 |    |    |    | 24 | 26 |    |
| **0** |   |   | 9 |    | 12 |    |    | 20 |    |    |    |    |
|   |   | **5** |   |    |    |    | 18 |    | 22 |    |    | 30 |

For example, if we want to build a range from 9, we only care about  numbers that are equal to or greater than 9:

|   |    |    |    |    |    |    |    |    |    |
|---|----|----|----|----|----|----|----|----|----|
|   | **10** |    | 15 |    |    |    | 24 | 26 |    |
| **9** |    | 12 |    |    | 20 |    |    |    |    |
|   |    |    |    | **18** |    | 22 |    |    | 30 |

9 belongs to the second list. We just need to find the smallest numbers in the first and the third lists in this cropped table, which are 10 and 18, and use the larger one as the end of the range, so finally we have range [9, 18].

To find 10 and 18, we can do binary search twice on those two lists. But that is not necessary. Suppose that we have already found 10 and 18, and use two pointers pointing to them. Later when we want to use 12 as the begin of the range, we find that 10 is less than 12, so we advance the corresponding pointer, which then points to 15 instead; on the other hand, 18 is already greater than 12, so no move. In this way we make a new range, [12, 18].

It seems that those two pointers can be reused only if we still use a number of the second list as the begin of the range, but we can make it more general, and finish everything in one scan.

We now use three pointers. Initially they point to the first elements of three lists (*i.e.* point to 0, 4 and 5). To form a range, simply find out the smallest and the largest among them (0 and 5). Then we advance the pointer pointing to the smallest one (from 0 to 9), find another range (among 4, 5 and 9), and repeat this until one of pointers reaches the end the list.

To avoid sorting, we can use a heap. A heap cannot keep track of both the smallest and the largest number, so in the solution, `heap` tracks **the next smallest number inside of a list, the index of that number, and the index of the list**, and `maximum` tracks **the largest number inside of `heap`**. When `heap` is updated, remember to update `maximum`.

### 759. Employee Free Time

We are given a list **schedule** of employees, which represents the working time for each employee.

Each employee has a list of non-overlapping **Intervals**, and these intervals are in sorted order.

Return the list of finite intervals representing **common, positive-length free time** for **all** employees, also in sorted order.

**Example 1:**

> **Input:** schedule = [[[1,2],[5,6]],[[1,3]],[[4,10]]]
> 
> **Output:** [[3,4]]
> 
> **Explanation:**
> 
There are a total of three employees, and all common
free time intervals would be [-inf, 1], [3, 4], [10, inf].
We discard any intervals that contain inf as they aren't finite.

**Example 2:**

> **Input:** schedule = [[[1,3],[6,7]],[[2,4]],[[2,5],[9,12]]]
> 
> **Output:** [[5,6],[7,9]]

(Even though we are representing **Intervals** in the form **[x, y]**, the objects inside are **Intervals**, not lists or arrays. For example, **schedule[0][0].start = 1, schedule[0][0].end = 2,** and **schedule[0][0][0]** is not defined.)

Also, we wouldn't include intervals like [5, 5] in our answer, as they have zero length.

**Note:**

1. **schedule** and **schedule[i]** are lists with lengths in range **[1, 50]**.
2. **0 <= schedule[i].start < schedule[i].end <= 10^8**.

```python
# Definition for an interval.
# class Interval:
#     def __init__(self, s=0, e=0):
#         self.start = s
#         self.end = e

from heapq import *

class Solution:
    def employeeFreeTime(self, schedule):
        """
        :type schedule: List[List[Interval]]
        :rtype: List[Interval]
        """
        heap = [(s[0].start, i, 0) for i, s in enumerate(schedule)]
        heapify(heap)
        max_start = float("-inf")
        ret = []
        while heap:
            end, idx, ptr = heappop(heap)
            if max_start != float("-inf") and max_start < end:
                ret.append(Interval(max_start, end))
            max_start = max(max_start, schedule[idx][ptr].end)
            ptr += 1
            if ptr < len(schedule[idx]):
                heappush(heap, (schedule[idx][ptr].start, idx, ptr))
        return ret
```

`heap` keeps track of three variables: the end of the free time (`end`), the index of the emplyee (`idx`), and the index of the interval within the schedule of this employee (`ptr`). We want to know which free time interval in `heap` starts latest (we track it with `max_start`), and which ends earliest (we read it from `heap[0]`). When they form a valid interval, we record them in `ret`. 

Note that the input `schedule` lables working time, so we use the `.start` of its elements as `end` of free time, and their `.end` as `start` of free time. Also note that when `ptr` reaches the end, we still update `max_start`, but we don't push it back to `heap` again. This means after the updated `max_start`, that employee is always free, and we don't have to worry about the availability of that employee any more.