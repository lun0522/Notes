# Dijkstra's Algorithm Problems

To find the shortest path in weighted graphs.

### 787. Cheapest Flights Within K Stops

There are **n** cities connected by **m** flights. Each fight starts from city **u** and arrives at **v** with a price **w**.

Now given all the cities and fights, together with starting city **src** and the destination **dst**, your task is to find the cheapest price from **src** to **dst** with up to **k** stops. If there is no such route, output **-1**.

> **Example 1:**
> 
> **Input:**
> 
> n = 3, edges = [[0,1,100],[1,2,100],[0,2,500]]
> 
> src = 0, dst = 2, k = 1
> 
> **Output:** 200
> 
> **Explanation:**
> 
> The graph looks like this:
> 
> ![](https://s3-lc-upload.s3.amazonaws.com/uploads/2018/02/16/995.png)
> 
> The cheapest price from city 0 to city 2 with at most 1 stop costs 200, as marked red in the picture.

> **Example 2:**
> 
> **Input:**
> 
> n = 3, edges = [[0,1,100],[1,2,100],[0,2,500]]
> 
> src = 0, dst = 2, k = 0
> 
> **Output:** 500
> 
> **Explanation:** 
> 
> The graph looks like this:
> 
> ![](https://s3-lc-upload.s3.amazonaws.com/uploads/2018/02/16/995.png)
> 
> The cheapest price from city 0 to city 2 with at most 0 stop costs 500, as marked blue in the picture.

**Note:**

- The number of nodes **n** will be in range **[1, 100]**, with nodes labeled from **0** to **n - 1**.
- The size of **flights** will be in range **[0, n * (n - 1) / 2]**.
- The format of each flight will be **(src, dst, price)**.
- The price of each flight will be in the range **[1, 10000]**.
- **k** is in the range of **[0, n - 1]**.
- There will not be any duplicated flights or self cycles.

```python
from heapq import *

class Solution:
    def findCheapestPrice(self, n, flights, src, dst, K):
        """
        :type n: int
        :type flights: List[List[int]]
        :type src: int
        :type dst: int
        :type K: int
        :rtype: int
        """
        flyto = {i: [] for i in range(n)}
        for city1, city2, price in flights:
            flyto[city1].append([city2, price])
        
        heap = [(0, src, 1)]
        while heap:
            cost, city, count = heappop(heap)
            if city == dst:
                return cost
            count += 1
            if count <= K + 2:
                for nbr, price in flyto[city]:
                    heappush(heap, [cost + price, nbr, count])
        return -1
```

- Whenever we arrive in a city, `cost` is guaranteed to be the lowest cost, so we immediately return when `city == dst`
- `count` tracks how many cities we have been to, so if we can have `K` stops, we can have been to `K + 2` cities in total
- We should be able to visit a city more than once, since some paths may have lower costs, but more steps, and thus they may fail

### 778. Swim in Rising Water

On an N x N **grid**, each square **grid[i][j]** represents the elevation at that point **(i,j)**.

Now rain starts to fall. At time **t**, the depth of the water everywhere is **t**. You can swim from a square to another 4-directionally adjacent square if and only if the elevation of both squares individually are at most **t**. You can swim infinite distance in zero time. Of course, you must stay within the boundaries of the grid during your swim.

You start at the top left square **(0, 0)**. What is the least time until you can reach the bottom right square **(N-1, N-1)**?

**Example 1:**

> **Input:** [[0,2],[1,3]]
> 
> **Output:** 3
> 
> **Explanation:**
> 
> At time 0, you are in grid location (0, 0).
> 
> You cannot go anywhere else because 4-directionally adjacent neighbors have a higher elevation than t = 0.
> 
> You cannot reach point (1, 1) until time 3.
> 
> When the depth of water is 3, we can swim anywhere inside the grid.

**Example 2:**

> **Input:** [[0,1,2,3,4],[24,23,22,21,5],[12,13,14,15,16],[11,17,18,19,20],[10,9,8,7,6]]
> 
> **Output:** 16
> 
> **Explanation:**
> 
>  **0**  **1**  **2**  **3**  **4**
> 
> 24 23 22 21  **5**
> 
> **12** **13** **14** **15** **16**
> 
> **11** 17 18 19 20
> 
> **10**  **9**  **8**  **7**  **6**
> 
> The final route is marked in bold.
> 
> We need to wait until time 16 so that (0, 0) and (4, 4) are connected.

**Note:**

1. **2 <= N <= 50**.
2. **grid[i][j]** is a permutation of **[0, ..., N*N - 1]**.

```python
from heapq import *

class Solution:
    def swimInWater(self, grid):
        """cms
        :type grid: List[List[int]]
        :rtype: int
        """
        N = len(grid)
        visited = set()
        heap = [(grid[0][0], 0, 0)]
        while heap:
            cost, row, col = heappop(heap)
            if (row, col) == (N - 1, N - 1):
                return cost
            for i, j in [(-1, 0), (1, 0), (0, -1), (0, 1)]:
                r, c = row + i, col + j
                if 0 <= r < N and 0 <= c < N and (r, c) not in visited:
                    visited.add((r, c))
                    heappush(heap, (max(cost, grid[r][c]), r, c))
```

Differently, we should only visit one place at most once. We also use a different way to update `cost`, which actually tracks the highest cost on the path.