# Trie Problems

### 212. Word Search II

Given a 2D board and a list of words from the dictionary, find all words in the board.

Each word must be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once in a word.

**Example:**

> **Input:** 
> 
> **words** = ["oath","pea","eat","rain"]
> 
> **board** =
> 
> [
> 
>   ['o','a','a','n'],
> 
>   ['e','t','a','e'],
> 
>   ['i','h','k','r'],
> 
>   ['i','f','l','v']
> 
> ]
> 
> **Output:** ["eat","oath"]

**Note:**

You may assume that all inputs are consist of lowercase letters **a-z**.

```python
class Node:
    def __init__(self):
        self.word = None
        self.next = {}

class Solution:
    def findWords(self, board, words):
        """
        :type board: List[List[str]]
        :type words: List[str]
        :rtype: List[str]
        """
        if not board:
            return []
        
        trie = Node()
        for word in words:
            node = trie
            for char in word:
                if char not in node.next:
                    node.next[char] = Node()
                node = node.next[char]
            node.word = word
        
        ret = []
        m, n = len(board), len(board[0])
        def dfs(row, col, node, visited):
            if node.word is not None:
                ret.append(node.word)
                node.word = None
            for i, j in [(-1, 0), (1, 0), (0, -1), (0, 1)]:
                y, x = row + i, col + j
                if 0 <= y < m and 0 <= x < n and (y, x) not in visited and board[y][x] in node.next:
                    visited.add((y, x))
                    dfs(y, x, node.next[board[y][x]], visited)
                    visited.remove((y, x))
        
        for r in range(m):
            for c in range(n):
                if board[r][c] in trie.next:
                    dfs(r, c, trie.next[board[r][c]], {(r, c)})
        return ret
```

Given a word, we can start from every point on `board` to do depth-first search. For multiple queries, we can store all possible words that can be formed with characters in `board` with a trie, but that will consume a huge amount of space. Instead, we build a trie with all `words` we want to query, and then do DFS starting from each point on `board`. We also set `node.word` to `None` to avoid outputing the same word twice.

### 425. Word Squares

Given a set of words **(without duplicates)**, find all word squares you can build from them.

A sequence of words forms a valid word square if the **k**-th row and column read the exact same string, where 0 ≤ **k** < max(numRows, numColumns).

For example, the word sequence **["ball","area","lead","lady"]** forms a word square because each word reads the same both horizontally and vertically.

> b a l l
> 
> a r e a
> 
> l e a d
> 
> l a d y

**Note:**

1. There are at least 1 and at most 1000 words.
2. All words will have the exact same length.
3. Word length is at least 1 and at most 5.
4. Each word contains only lowercase English alphabet **a-z**.

**Example 1:**

> **Input:**
> 
> ["area","lead","wall","lady","ball"]
> 
> **Output:**
> 
> [
> 
>   [ 
> 
>     "wall",
> 
>     "area",
> 
>     "lead",
> 
>     "lady"
> 
>   ],
> 
>   [ 
> 
>     "ball",
> 
>     "area",
> 
>     "lead",
> 
>     "lady"
> 
>   ]
> 
> ]
> 
> **Explanation:**
> 
> The output consists of two word squares. The order of output does not matter (just the order of words in each word square matters).

**Example 2:**

> **Input:**
> 
> ["abat","baba","atan","atal"]
> 
> **Output:**
> 
> [
> 
>   [ 
> 
>     "baba",
> 
>     "abat",
> 
>     "baba",
> 
>     "atan"
> 
>   ],
> 
>   [
>  
>     "baba",
> 
>     "abat",
> 
>     "baba",
> 
>     "atal"
> 
>   ]
> 
> ]

**Explanation:**
The output consists of two word squares. The order of output does not matter (just the order of words in each word square matters).

```python
class Node:
    def __init__(self):
        self.words = []
        self.next = {}

class Solution:
    def wordSquares(self, words):
        """
        :type words: List[str]
        :rtype: List[List[str]]
        """
        trie = Node()
        trie.words = words
        for word in words:
            node = trie
            for char in word:
                if char not in node.next:
                    node.next[char] = Node()
                node = node.next[char]
                node.words.append(word)
        
        def get(prefix):
            node = trie
            for char in prefix:
                if char in node.next:
                    node = node.next[char]
                else:
                    return []
            return node.words
        
        ret = []
        def dfs(prev):
            if len(prev) == len(words[0]):
                ret.append(prev[:])
            else:
                ptr = len(prev)
                for next_word in get([word[ptr] for word in prev]):
                    prev.append(next_word)
                    dfs(prev)
                    prev.pop()
            return ret
        return dfs([])
```

Within all `node` in `trie`, we record all words that has this prefix in `node.words`. The root node of `trie` simply stores all input `words`. We can use `get()` to retrieve them given a prefix. In `dfs()`, we can find out the prefix of the next word that can be added to `prev`.

For example, if input `words` are **["area","lead","wall","lady","ball"]**, if we start with **"wall"**, `ptr` is **1**, and we will require the second word to start with **["wall"[1]]**, which is **["a"]**, so we get **["area"]** from `get()`. Then `ptr` becomes **2**, and we require the next word to start with **["wall"[2], "area"[2]]**, which is **["l", "e"]**, so we get **["lead"]**. Do this again, the last word will start with **["l", "a", "d"]**, so we get **["lady"]**.