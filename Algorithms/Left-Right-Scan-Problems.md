# Left Right Scan Problems

### 152. Maximum Product Subarray

Given an integer array **nums**, find the contiguous subarray within an array (containing at least one number) which has the largest product.

Example 1:

> Input: [2,3,-2,4]
> 
> Output: 6
> 
> Explanation: [2,3] has the largest product 6.

Example 2:

> Input: [-2,0,-1]
> 
> Output: 0
> 
> Explanation: The result cannot be 2, because [-2,-1] is not a subarray.

```python
class Solution:
    def maxProduct(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        def maxPro(arr):
            product, max_ = 1, arr[0]
            for num in arr:
                product *= num
                max_ = max(max_, product)
                if num == 0:
                    product = 1
            return max_
        
        return nums and max(maxPro(nums), maxPro(nums[::-1])) or 0
```

If we only scan from left to right, the number of negative elements in the array might be odd, and thus the product will be negative (consider **[-1,2,3]**). But we can scan from right to left to fix this. Note that when we encounter a **0**, reset `product` to 1, otherwise it will always be zero from now on.

### 32. Longest Valid Parentheses

Given a string containing just the characters **'('** and **')'**, find the length of the longest valid (well-formed) parentheses substring.

Example 1:

> Input: "(()"
> 
> Output: 2
> 
> Explanation: The longest valid parentheses substring is "()"

Example 2:

> Input: ")()())"
> 
> Output: 4
> 
> Explanation: The longest valid parentheses substring is "()()"

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

Consider `(()`, scanning from right to left can fix this.