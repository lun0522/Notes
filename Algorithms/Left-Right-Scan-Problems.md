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

### 135. Candy

There are **N** children standing in a line. Each child is assigned a rating value.

You are giving candies to these children subjected to the following requirements:

- Each child must have at least one candy.
- Children with a higher rating get more candies than their neighbors.

What is the minimum candies you must give?

**Example 1:**

> **Input:** [1,0,2]
> 
> **Output:** 5
> 
> **Explanation:** You can allocate to the first, second and third child with 2, 1, 2 candies respectively.

**Example 2:**

> **Input:** [1,2,2]
> 
> **Output:** 4
> 
> **Explanation:** You can allocate to the first, second and third child with 1, 2, 1 candies respectively. The third child gets 1 candy because it satisfies the above two conditions.

```python
class Solution:
    def candy(self, ratings):
        """
        :type ratings: List[int]
        :rtype: int
        """
        ret = [1] * len(ratings)
        for i in range(1, len(ret)):
            if ratings[i] > ratings[i - 1]:
                ret[i] = ret[i - 1] + 1
        for i in range(len(ret) - 1)[::-1]:
            if ratings[i] > ratings[i + 1]:
                ret[i] = max(ret[i], ret[i + 1] + 1)
        return sum(ret)
```

Note that if the rating of a child is less than or equal to the ratings of neighbors, we can simply assign `1`. We can split the output array into several mountains, where these `1`s are valleys between mountains. Each mountain has exactly one peak and no valley, otherwise we can split it into smaller mountains by the valley. It should have one rising edge and one falling edge. When we scan from the left, we assign candies for the rising edge; when we scan from the right, we build the falling edge. Note that the peaking point is shared by both edges, so we use `max` to determine the height of it.