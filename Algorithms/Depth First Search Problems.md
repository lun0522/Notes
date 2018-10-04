# Depth First Search Problems

Dynamic programming uses memorization to eliminate unnecessary intermediate states, while in depth/bredth first search, every state matters.

### 351. Android Unlock Patterns

Given an Android **3x3** key lock screen and two integers **m** and **n**, where 1 ≤ m ≤ n ≤ 9, count the total number of unlock patterns of the Android lock screen, which consist of minimum of **m** keys and maximum **n** keys.

Rules for a valid pattern:

1. Each pattern must connect at least **m** keys and at most **n** keys.
2. All the keys must be distinct.
3. If the line connecting two consecutive keys in the pattern passes through any other keys, the other keys must have previously selected in the pattern. No jumps through non selected key is allowed.
4. The order of keys used matters.

**Explanation:**

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

**Example:**

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

### 294. Flip Game II

You are playing the following Flip Game with your friend: Given a string that contains only these two characters: **+** and **-**, you and your friend take turns to flip two **consecutive** **"++"** into **"--"**. The game ends when a person can no longer make a move and therefore the other person will be the winner.

Write a function to determine if the starting player can guarantee a win.

**Example:**

> **Input:** s = "++++"
> 
> **Output:** true 
> 
> **Explanation:** The starting player can guarantee a win by flipping the middle "++" to become "+--+".

```python
class Solution:
    def canWin(self, s):
        """
        :type s: str
        :rtype: bool
        """
        anyWin, allLose = set(), set()
        
        def flip(chars):  # generate all possible moves
            ret = []
            for i in range(1, len(chars)):
                if chars[i - 1] == "+" and chars[i] == "+":
                    tmp = chars[:]
                    tmp[i - 1] = tmp[i] = "-"
                    ret.append(tmp)
            return ret
        
        def play(chars):  # return True if any win, False if all lose
            for c in flip(chars):
                tc = tuple(c)
                if tc in anyWin:
                    return True
                elif tc in allLose:
                    continue
                else:  # let second player play
                    if play(c):
                        allLose.add(tc)
                    else:
                        anyWin.add(tc)
                        return True
            return False
        
        return play(list(s))
```

The helper function `flip` generates all possible moves. If it returns nothing, then the current player will lose (no possible move can be made).

Suppose A is the first player, and B is the second player. For A, if after a certain move made by A, any move made by B will guarantee B a lose, then A will win. We may have two functions: `firstPlayer()` and `secondPlayer()`, each of which returns `True` if that player has chance to win, or `False` if that player will certainly lose.

They will call each other, and if after any move made by the current player, another player can never win, then the current player can guarantee a win. These two functions have exactly the reverse logic, so they are merged into `play()`. `anyWin` and `allLose` are used to avoid duplicated computations.

### 10. Regular Expression Matching

Given an input string (**s**) and a pattern (**p**), implement regular expression matching with support for **'.'** and **'\*'**.

> '.' Matches any single character.
> 
> '*' Matches zero or more of the preceding element.

The matching should cover the **entire** input string (not partial).

**Note:**

- **s** could be empty and contains only lowercase letters **a-z**.
- **p** could be empty and contains only lowercase letters **a-z**, and characters like **.** or **\***.

**Example 1:**

> **Input:**
> 
> s = "aa"
> 
> p = "a"
> 
> **Output:** false
> 
> **Explanation:** "a" does not match the entire string "aa".

**Example 2:**

> **Input:**
> 
> s = "aa"
> 
> p = "a\*"
> 
> **Output:** true
> 
> **Explanation:** '*' means zero or more of the precedeng element, 'a'. Therefore, by repeating 'a' once, it becomes "aa".

**Example 3:**

> **Input:**
> 
> s = "ab"
> 
> p = ".\*"
> 
> **Output:** true
> 
> **Explanation:** ".\*" means "zero or more (*) of any character (.)".

**Example 4:**

> **Input:**
> 
> s = "aab"
> 
> p = "c\*a\*b"
> 
> **Output:** true
> 
> **Explanation:** c can be repeated 0 times, a can be repeated 1 time. Therefore it matches "aab".

**Example 5:**

> **Input:**
> 
> s = "mississippi"
> 
> p = "mis\*is\*p\*."
> 
> **Output:** false

```python
class Solution:
    def isMatch(self, s, p):
        """
        :type s: str
        :type p: str
        :rtype: bool
        """
        memory = {}
        def dfs(sPtr, pPtr):
            if (sPtr, pPtr) not in memory:
                if pPtr == len(p):  # 1
                    ret = sPtr == len(s)
                else:
                    match = sPtr < len(s) and p[pPtr] in {".", s[sPtr]}
                    if pPtr + 1 < len(p) and p[pPtr + 1] == "*":  # 2
                        ret = dfs(sPtr, pPtr + 2) or (match and dfs(sPtr + 1, pPtr))
                    else:  # 3
                        ret = match and dfs(sPtr + 1, pPtr + 1)
                memory[sPtr, pPtr] = ret
            return memory[sPtr, pPtr]
        
        return dfs(0, 0)
```

1. If `p` is used up, `s` may not have ended: `ret = sPtr == len(s)`.
2. If there is a **\***, we **may** be able to skip chars in `s`. We can choose not to skip anything in `s`: `dfs(sPtr, pPtr + 2)`. If that didn't success, and `s[sPtr]` matches `p[pPtr]`, we can skip one char in `s`: `match and dfs(sPtr + 1, pPtr)`. Later recursive calls will skip more chars.
3. If there isn't a **\***, we may move both `sPtr` and `pPtr` forward if `s[sPtr]` matches `p[pPtr]`: `ret = match and dfs(sPtr + 1, pPtr + 1)`.