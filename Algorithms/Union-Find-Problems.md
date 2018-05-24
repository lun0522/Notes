# Union Find Problems

### 305. Number of Islands II

A 2d grid map of **m** rows and **n** columns is initially filled with water. We may perform an addLand operation which turns the water at position (row, col) into a land. Given a list of positions to operate, **count the number of islands after each *addLand* operation**. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically. You may assume all four edges of the grid are all surrounded by water.

Example:

Given **m = 3, n = 3, positions = [[0,0], [0,1], [1,2], [2,1]]**.
Initially, the 2d grid **grid** is filled with water. (Assume 0 represents water and 1 represents land).

> 0 0 0
>
> 0 0 0
>
> 0 0 0

Operation #1: addLand(0, 0) turns the water at grid[0][0] into a land.

> 1 0 0
>
> 0 0 0   Number of islands = 1
>
> 0 0 0

Operation #2: addLand(0, 1) turns the water at grid[0][1] into a land.

> 1 1 0
>
> 0 0 0   Number of islands = 1
>
> 0 0 0

Operation #3: addLand(1, 2) turns the water at grid[1][2] into a land.

> 1 1 0
>
> 0 0 1   Number of islands = 2
>
> 0 0 0

Operation #4: addLand(2, 1) turns the water at grid[2][1] into a land.

> 1 1 0
>
> 0 0 1   Number of islands = 3
>
> 0 1 0

We return the result as an array: **[1, 1, 2, 3]**

Challenge:

Can you do it in time complexity O(k log mn), where k is the length of the **positions**?

```python
class Solution:
    def numIslands2(self, m, n, positions):
        """
        :type m: int
        :type n: int
        :type positions: List[List[int]]
        :rtype: List[int]
        """
        parent, size = {}, {}
        numLands = [0]
        ret = []
        
        def add(pos):
            parent[pos] = pos
            size[pos] = 1
            numLands[0] += 1  # do not forget this
        
        def root(pos):
            while pos != parent[pos]:
                # path compression
                # do NOT use pos = parent[pos] = parent[parent[pos]]
                # where pos is updated at first, then parent[pos] get updated
                parent[pos] = parent[parent[pos]]
                pos = parent[pos]
            return pos
        
        def find(pos1, pos2):
            return root(pos1) == root(pos2)
        
        def union(pos1, pos2):
            # do NOT directly merge pos1 and pos2
            # they may be leaf nodes
            # merge their roots instead
            root1, root2 = root(pos1), root(pos2)
            if size[root1] > size[root2]:
                root1, root2 = root2, root1
            # size[root1] <= size[root2], merge 1 into 2
            parent[root1] = root2
            size[root2] += size[root1]
            numLands[0] -= 1  # do not forget this
        
        for row, col in positions:
            if (row, col) not in parent:
                # simply add an island at first
                add((row, col))
                # iterate over neighbors to see whether they are lands
                # and belong to the same island
                for yOffset, xOffset in [(0, -1), (0, 1), (-1, 0), (1, 0)]:
                    y, x = row + yOffset, col + xOffset
                    if (y, x) in parent and not find((row, col), (y, x)):
                        union((row, col), (y, x))
            ret.append(numLands[0])
            
        return ret
```