# Counting Ongoing Problems

### 253. Meeting Rooms II

Given an array of meeting time intervals consisting of start and end times **[[s1,e1],[s2,e2],...]** (si < ei), find the minimum number of conference rooms required.

For example, Given **[[0, 30],[5, 10],[15, 20]]**, return **2**.

```python
from heapq import *

class Solution(object):
    def minMeetingRooms(self, intervals):
        """
        :type intervals: List[Interval]
        :rtype: int
        """
        # sort intervals according to the starting time
        intervals.sort(key=lambda x: x.start)
        curMeeting = []  # only store the ending time
        curRoom = maxRoom = 0
        
        for interval in intervals:
            # remove meetings that end before this meeting starts
            while curMeeting and curMeeting[0] <= interval.start:
                heappop(curMeeting)
                curRoom -= 1
            
            # push the ending time of the current meeting to the heap
            heappush(curMeeting, interval.end)
            curRoom += 1
            if curRoom > maxRoom:
                maxRoom = curRoom
            
        return maxRoom
```


### 218. The Skyline Problem

A city's skyline is the outer contour of the silhouette formed by all the buildings in that city when viewed from a distance. Now suppose you are given the locations and height of all the buildings as shown on a cityscape photo (Figure A), write a program to output the skyline formed by these buildings collectively (Figure B).

![](https://leetcode.com/static/images/problemset/skyline1.jpg)![](https://leetcode.com/static/images/problemset/skyline2.jpg)

The geometric information of each building is represented by a triplet of integers **[Li, Ri, Hi]**, where **Li** and **Ri** are the x coordinates of the left and right edge of the ith building, respectively, and **Hi** is its height. It is guaranteed that **0 ≤ Li**, **Ri ≤ INT_MAX**, **0 < Hi ≤ INT_MAX**, and **Ri - Li > 0**. You may assume all buildings are perfect rectangles grounded on an absolutely flat surface at height 0.

For instance, the dimensions of all buildings in Figure A are recorded as: **[ [2 9 10], [3 7 15], [5 12 12], [15 20 10], [19 24 8] ]**.

The output is a list of "key points" (red dots in Figure B) in the format of **[ [x1,y1], [x2, y2], [x3, y3], ... ]** that uniquely defines a skyline. A key point is the left endpoint of a horizontal line segment. Note that the last key point, where the rightmost building ends, is merely used to mark the termination of the skyline, and always has zero height. Also, the ground in between any two adjacent buildings should be considered part of the skyline contour.

For instance, the skyline in Figure B should be represented as:**[ [2 10], [3 15], [7 12], [12 0], [15 10], [20 8], [24, 0] ]**.

**Notes:**

- The number of buildings in any input list is guaranteed to be in the range **[0, 10000]**.
- The input list is already sorted in ascending order by the left x position **Li**.
- The output list must be sorted by the x position.
- There must be no consecutive horizontal lines of equal height in the output skyline. For instance, **[...[2 3], [4 5], [7 5], [11 5], [12 7]...]** is not acceptable; the three lines of height 5 should be merged into one in the final output as such: **[...[2 3], [4 5], [12 7], ...]**

```python
from heapq import *

class Solution(object):
    def getSkyline(self, buildings):
        """
        :type buildings: List[List[int]]
        :rtype: List[List[int]]
        """
        points = list(set([building[0] for building in buildings] + [building[1] for building in buildings]))
        points.sort()  # all key points where the largest height may change, in ascending order
        
        ret, alive = [], []  # alive is a min heap: (negative height, end point)
        ptr, prevH = 0, 0
        for point in points:
            # push all buildings that start before or at this point onto the heap
            while ptr < len(buildings) and buildings[ptr][0] <= point:
                heappush(alive, (-buildings[ptr][2], buildings[ptr][1]))
                ptr += 1
            
            # pop buildings that end before or at this point off the heap (note this is a lazy removal)
            while alive and alive[0][1] <= point:
                heappop(alive)
            
            curH = alive and -alive[0][0] or 0
            if curH != prevH:
                ret.append([point, curH])
                prevH = curH
        
        return ret
```

Lazy removal is very important. When we use a heap, we don't have to remove all elements in the heap that do not satisfy our need, but only repetitively remove the one at index 0 that does not satisfy the requirement.