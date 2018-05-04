# Depth First Search Problems

Dynamic programming uses memorization to eliminate unnecessary intermediate states, while in depth/bredth first search, every state matters.

### 351. Android Unlock Patterns

Given an Android **3x3** key lock screen and two integers **m** and **n**, where 1 ≤ m ≤ n ≤ 9, count the total number of unlock patterns of the Android lock screen, which consist of minimum of **m** keys and maximum **n** keys.

Rules for a valid pattern:

1. Each pattern must connect at least **m** keys and at most **n** keys.
2. All the keys must be distinct.
3. If the line connecting two consecutive keys in the pattern passes through any other keys, the other keys must have previously selected in the pattern. No jumps through non selected key is allowed.
4. The order of keys used matters.

Explanation:

> | 1 | 2 | 3 |
> 
> | 4 | 5 | 6 |
> 
> | 7 | 8 | 9 |

Invalid move: **4 - 1 - 3 - 6**

Line 1 - 3 passes through key 2 which had not been selected in the pattern.

Invalid move: **4 - 1 - 9 - 2**

Line 1 - 9 passes through key 5 which had not been selected in the pattern.

Valid move: **2 - 4 - 1 - 3 - 6**

Line 1 - 3 is valid because it passes through key 2, which had been selected in the pattern

Valid move: **6 - 5 - 4 - 1 - 9 - 2**

Line 1 - 9 is valid because it passes through key 5, which had been selected in the pattern.

Example:

Given **m** = 1, **n** = 1, return 9.

```python
class Solution:
    def numberOfPatterns(self, m, n):
        """
        :type m: int
        :type n: int
        :rtype: int
        """
        lookup = {}
        for i in [1, 4, 7]:
            lookup[(i, i + 2)] = lookup[(i + 2, i)] = i + 1
        for i in [1, 2, 3]:
            lookup[(i, i + 6)] = lookup[(i + 6, i)] = i + 3
        lookup[(1, 9)] = lookup[(9, 1)] = lookup[(3, 7)] = lookup[(7, 3)] = 5
        
        ret = [0]
        visited = set()
        
        def visit(num):
            visited.add(num)
            if len(visited) >= m:
                ret[0] += 1
            if len(visited) < n:
                dfs(num)
            visited.remove(num)
        
        def dfs(start):
            for i in range(1, 10):
                if i not in visited:
                    if (start, i) not in lookup or lookup[(start, i)] in visited:
                        visit(i)
        
        # 1, 3, 7, 9 are symmetric; 2, 4, 6, 8 are symmetric
        [visit(i) for i in [1, 2]]
        ret[0] *= 4
        visit(5)
        
        return ret[0]
```

A lookup table is built to store prerequisites. For example, before jumping from 1 to 3, we must have visited 2 before, so `lookup[(1, 3)] = lookup[(3, 1)] = 2`. An improvement is to just start from 1, 2, and 5. Because of symmetry, starting from 3, 7, 9 is the same to starting from 1; similar for 2.