# Valid Parentheses Problems

If we need to generate (or validate) a string of parentheses (*i.e.* consisting of only **(** and **)**), there is a simple rule: on the left hand side, the number of **(** should always be greater than or equal to the number of **)**. Later we will use `leftCnt` and `rightCnt` to keep track of the count of them.

### 22. Generate Parentheses

Given **n** pairs of parentheses, write a function to generate all combinations of well-formed parentheses.

For example, given **n** = 3, a solution set is:

> [
> 
>   "((()))",
> 
>   "(()())",
> 
>   "(())()",
> 
>   "()(())",
> 
>   "()()()"
> 
> ]

```python
class Solution:
    def generateParenthesis(self, n):
        """
        :type n: int
        :rtype: List[str]
        """
        def put(remain, leftCnt, rightCnt, prev, res):
            if not remain:
                res.append("".join(prev))
            else:
                if leftCnt < n:  # can put a (
                    prev.append("(")
                    put(remain - 1, leftCnt + 1, rightCnt, prev, res)
                    prev.pop()
                if leftCnt > rightCnt:  # can put a )
                    prev.append(")")
                    put(remain - 1, leftCnt, rightCnt + 1, prev, res)
                    prev.pop()
            return res
        
        return put(n * 2, 0, 0, [], [])
```

The entire string should contain n **(** and n **)**, so we can do a depth first search, try to append a **(** whenever `leftCnt < n`, and try to append a **)** whenever `leftCnt > rightCnt`.

### 32. Longest Valid Parentheses

Given a string containing just the characters **'('** and **')'**, find the length of the longest valid (well-formed) parentheses substring.

**Example 1:**

> **Input:** "(()"
> 
> **Output:** 2
> 
> **Explanation:** The longest valid parentheses substring is "()"

**Example 2:**

> **Input:** ")()())"
> 
> **Output:** 4
> 
> **Explanation:** The longest valid parentheses substring is "()()"
 
```python
class Solution:
    def longestValidParentheses(self, s):
        """
        :type s: str
        :rtype: int
        """
        def longest(string, left, right):
            leftCnt = rightCnt = maxLen = 0
            for char in string:
                if char == left:
                    leftCnt += 1
                else:
                    rightCnt += 1
                    if leftCnt == rightCnt:  # all lefts and rights are paired
                        maxLen = max(maxLen, leftCnt * 2)
                    elif leftCnt < rightCnt:  # invalid: more rights than lefts
                        leftCnt = rightCnt = 0
            return maxLen
        
        return max(longest(s, "(", ")"), longest(s[::-1], ")", "("))
```

When `leftCnt == rightCnt`, a valid substring has been found, and it is simple to know its length. When `leftCnt < rightCnt`, the string becomes invalid because of the currently encountered parenthese, so we reset `leftCnt` and `rightCnt` to 0.

We scan from the left once (to handle inputs like **")()())"**, whose substring is invalid because of **)**) and from the right once (to handle "(()", whose substring is invalid because of **(**).