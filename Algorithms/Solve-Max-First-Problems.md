# Solve Max First Problems

Always eliminate the ones with highest count at first.

### 621. Task Scheduler

Given a char array representing tasks CPU need to do. It contains capital letters A to Z where different letters represent different tasks.Tasks could be done without original order. Each task could be done in one interval. For each interval, CPU could finish one task or just be idle.

However, there is a non-negative cooling interval **n** that means between two **same tasks**, there must be at least n intervals that CPU are doing different tasks or just be idle.

You need to return the **least** number of intervals the CPU will take to finish all the given tasks.

**Example 1:**

> **Input:** tasks = ["A","A","A","B","B","B"], n = 2
> 
> **Output:** 8
> 
> **Explanation:** A -> B -> idle -> A -> B -> idle -> A -> B.

**Note:**

1. The number of tasks is in the range [1, 10000].
2. The integer n is in the range [0, 100].

```python
from heapq import *

class Solution:
    def leastInterval(self, tasks, n):
        """
        :type tasks: List[str]
        :type n: int
        :rtype: int
        """
        ret, n = 0, n + 1
        heap = [-count for _, count in collections.Counter(tasks).items()]
        heapify(heap)
        while heap:
            length = len(heap)
            if length >= n:
                todo = [heappop(heap) for _ in range(n)]
                [heappush(heap, t + 1) for t in todo if t + 1]
            else:
                heap = [t + 1 for t in heap if t + 1]
            if heap:
                ret += n
            else:
                ret += length
        return ret
```

Re-difine `n` as the **length of a period**, which means we will do `n` tasks, and then do another `n` tasks, and so on (may include "idle states"). It is one unit greater than input `n`.

Use `heap` to keep track of tasks with highest count, we always do them at first. When the count of a certain task reaches 0, don't push it back to `heap` any more.

There are three stages:

1. We have enough kinds of tasks, and there is no need to insert "idle" states => `ret += n`
2. We don't have enough kinds of tasks, but we will insert "idle" states to fill a period => `ret += n`
3. We don't have enough kinds of tasks, and after this batch of tasks are done, there is no more tasks to do (*i.e.* `heap` becomes empty), and thus no need to insert "idle" states => `ret += previous length of heap`

### 774. Minimize Max Distance to Gas Station

On a horizontal number line, we have gas stations at positions **stations[0], stations[1], ..., stations[N-1]**, where **N = stations.length**.

Now, we add **K** more gas stations so that **D**, the maximum distance between adjacent gas stations, is minimized.

Return the smallest possible value of **D**.

**Example:**

> **Input:** stations = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10], K = 9
> 
> **Output:** 0.500000

**Note:**

1. **stations.length** will be an integer in range **[10, 2000]**.
2. **stations[i]** will be an integer in range **[0, 10^8]**.
3. **K** will be an integer in range **[1, 10^6]**.
4. Answers within **10^-6** of the true value will be accepted as correct.

```python
from heapq import *

class Solution:
    def minmaxGasDist(self, stations, K):
        """
        :type stations: List[int]
        :type K: int
        :rtype: float
        """
        boundary = (stations[-1] - stations[0]) / K
        dists = [float(stations[i + 1] - stations[i]) for i in range(len(stations) - 1)]
        
        heap = []
        for dist in dists:
            numStation = max(1, int(dist / boundary)) - 1
            K -= numStation
            heap.append([-dist / (numStation + 1), dist, numStation])
        heapify(heap)
        
        for _ in range(K):
            _, originDist, numStation = heap[0]
            numStation += 1
            currentDist = originDist / (numStation + 1)
            heapreplace(heap, [-currentDist, originDist, numStation])
            
        return -heap[0][0]
```

Use `heap` to keep track of maximum distances, which stores [negated current distance, original distance, number of inserted gas station]. We always insert stations to the greatest distance.

An important improvement: the lower bound of current distances will never be better (less than) `(stations[-1] - stations[0]) / K`, where all gas stations are distributed perfectly evenly. So, we can intially put gas stations, as long as they don't make the current distance lower than this boundary.