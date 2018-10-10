# Union Find Problems

### 305. Number of Islands II

A 2d grid map of **m** rows and **n** columns is initially filled with water. We may perform an addLand operation which turns the water at position (row, col) into a land. Given a list of positions to operate, **count the number of islands after each *addLand* operation**. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically. You may assume all four edges of the grid are all surrounded by water.

**Example:**

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

**Challenge:**

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

### 684. Redundant Connection

In this problem, a tree is an **undirected** graph that is connected and has no cycles.

The given input is a graph that started as a tree with N nodes (with distinct values 1, 2, ..., N), with one additional edge added. The added edge has two different vertices chosen from 1 to N, and was not an edge that already existed.

The resulting graph is given as a 2D-array of **edges**. Each element of **edges** is a pair **[u, v]** with **u < v**, that represents an undirected edge connecting nodes **u** and **v**.

Return an edge that can be removed so that the resulting graph is a tree of N nodes. If there are multiple answers, return the answer that occurs last in the given 2D-array. The answer edge **[u, v]** should be in the same format, with **u < v**.

**Example 1:**

> **Input:** [[1,2], [1,3], [2,3]]
> 
> **Output:** [2,3]
> 
> **Explanation:** The given undirected graph will be like this:
> 
>   1
> 
>  / \
> 
> 2 - 3

**Example 2:**

> **Input:** [[1,2], [2,3], [3,4], [1,4], [1,5]]
> 
> **Output:** [1,4]
> 
> **Explanation:** The given undirected graph will be like this:
> 
> 5 - 1 - 2
> 
>     |   |
> 
>     4 - 3

**Note:**

- The size of the input 2D-array will be between 3 and 1000.
- Every integer represented in the 2D-array will be between 1 and N, where N is the size of the input array.

```python
class Solution:
    def findRedundantConnection(self, edges):
        """
        :type edges: List[List[int]]
        :rtype: List[int]
        """
        N = len(edges) + 1
        parent = list(range(N))
        size = [1] * N
        
        def root(node):
            while parent[node] != node:
                parent[node] = parent[parent[node]]
                node = parent[node]
            return node
        
        def union(root1, root2):
            if size[root1] > size[root2]:
                root1, root2 = root2, root1
            parent[root1] = root2
            size[root2] += size[root1]
        
        for u, v in edges:
            r1, r2 = root(u), root(v)
            if r1 == r2:
                return [u, v]
            union(r1, r2)
```

We consider `edges` one by one. If anyone causes a problem (forms a circle), return it.

### 685. Redundant Connection II

In this problem, a rooted tree is a **directed** graph such that, there is exactly one node (the root) for which all other nodes are descendants of this node, plus every node has exactly one parent, except for the root node which has no parents.

The given input is a directed graph that started as a rooted tree with N nodes (with distinct values 1, 2, ..., N), with one additional directed edge added. The added edge has two different vertices chosen from 1 to N, and was not an edge that already existed.

The resulting graph is given as a 2D-array of **edges**. Each element of **edges** is a pair **[u, v]** that represents a **directed** edge connecting nodes **u** and **v**, where **u** is a parent of child **v**.

Return an edge that can be removed so that the resulting graph is a rooted tree of N nodes. If there are multiple answers, return the answer that occurs last in the given 2D-array.

**Example 1:**

> **Input:** [[1,2], [1,3], [2,3]]
> 
> **Output:** [2,3]
> 
> **Explanation:** The given directed graph will be like this:
> 
>   1
> 
>  / \
> 
> v   v
> 
> 2-->3

**Example 2:**

> **Input:** [[1,2], [2,3], [3,4], [4,1], [1,5]]
> 
> **Output:** [4,1]
> 
> **Explanation:** The given directed graph will be like this:
> 
> 5 <- 1 -> 2
> 
>      ^    |
> 
>      |    v
> 
>      4 <- 3

**Note:**

- The size of the input 2D-array will be between 3 and 1000.
- Every integer represented in the 2D-array will be between 1 and N, where N is the size of the input array.

```python
class Solution:
    def findRedundantDirectedConnection(self, edges):
        """
        :type edges: List[List[int]]
        :rtype: List[int]
        """
        N = len(edges) + 1
        parent = [0] * N
        candidates = None
        for i, (u, v) in enumerate(edges):
            if parent[v]:
                candidates = [[parent[v], v], [u, v]]
                edges[i] = [-1, -1]  # disable this edge
                break
            else:
                parent[v] = u
        
        def root(node):
            while parent[node] != node:
                parent[node] = parent[parent[node]]
                node = parent[node]
            return node
        
        def union(root1, root2):
            if size[root1] > size[root2]:
                root1, root2 = root2, root1
            parent[root1] = root2
            size[root2] += size[root1]
        
        parent = list(range(N))
        size = [1] * N
        for u, v in edges:
            if u < 0:
                continue
            r1, r2 = root(u), root(v)
            if r1 == r2:
                if candidates:
                    return candidates[0]
                else:
                    return [u, v]
            else:
                union(r1, r2)
        return candidates[1]
```

There might be two issues with the graph:

1. A certain node may have two parents.
2. There might be a circle (so that no one can be the root).

- If only 1 happens, we will have two candidate edges, we return the one occurs last in the input `edges`.
- If only 2 happens, we simply remove the edge that causes the circle (just like *684. Redundant Connection*)
- If both of them happen, we will have two candidate edges. We know the answer must be among these two candidates. We remove the second candidate at first, and then do union-find. If there is no circle, that means we had removed the problematic edge, which is the second candidate; otherwise we return the first candidate.

In the code above, we first find out whether there is a node that has two parents. If so, store them in `candidate` and disable `candidate[1]`. Later when we do union-find, if there is no circle, return `candidate[1]`; if there is a circle and `candidate`, return `candidate[0]`; if there is a circle but no `candidate`, return the edge that causes the circle.