#Matrix DP Problems

###221. Maximal Square

Given a 2D binary matrix filled with 0's and 1's, find the largest square containing only 1's and return its area.

For example, given the following matrix:

>1 0 1 0 0

>1 0 **1 1** 1

>1 1 **1 1** 1

>1 0 0 1 0

Return 4.

```python
class Solution:
    def maximalSquare(self, matrix):
        """
        :type matrix: List[List[str]]
        :rtype: int
        """
        if not matrix:
            return 0
        # dummy row and dummy column make code clean
        dp = [[0] * (len(matrix[0]) + 1) for _ in range(len(matrix) + 1)]
        maximal = 0
        for i in range(1, len(matrix) + 1):
            for j in range(1, len(matrix[0]) + 1):
                if matrix[i - 1][j - 1] == "1":
                    dp[i][j] = min(dp[i - 1][j - 1], dp[i - 1][j], dp[i][j - 1]) + 1
                    maximal = max(maximal, dp[i][j])
        return maximal * maximal  # return area
```

dp[i][j] is the side length of the largest square whose **bottom right** is at matrix[i-1][j-1]. To renew dp[i][j], we only need dp[i-1][j-1], dp[i-1][j] and dp[i][j-1] i.e. left top, top and left element.

###304. Range Sum Query 2D - Immutable

Given a 2D matrix matrix, find the sum of the elements inside the rectangle defined by its upper left corner (row1, col1) and lower right corner (row2, col2).

![](https://leetcode.com/static/images/courses/range_sum_query_2d.png)

The above rectangle (with the red border) is defined by (row1, col1) = **(2, 1)** and (row2, col2) = **(4, 3)**, which contains sum = **8**.

Example:

>[3, 0, 1, 4, 2],
  
>[5, 6, 3, 2, 1],
  
>[1, 2, 0, 1, 5],
  
>[4, 1, 0, 1, 7],
  
>[1, 0, 3, 0, 5]

sumRegion(2, 1, 4, 3) -> 8

sumRegion(1, 1, 2, 2) -> 11

sumRegion(1, 2, 2, 4) -> 12

Note:

You may assume that the matrix does not change.

There are many calls to sumRegion function.

You may assume that row1 ≤ row2 and col1 ≤ col2.

```python
class NumMatrix:

    def __init__(self, matrix):
        """
        :type matrix: List[List[int]]
        """
        if not matrix:
            self.dp = None
        else:
            dp = [[0] * (len(matrix[0]) + 1) for _ in range(len(matrix) + 1)]
            for i in range(1, len(matrix) + 1):
                for j in range(1, len(matrix[0]) + 1):
                    dp[i][j] = dp[i][j - 1] + dp[i - 1][j] - dp[i - 1][j - 1] + matrix[i - 1][j - 1]
            self.dp = dp

    def sumRegion(self, row1, col1, row2, col2):
        """
        :type row1: int
        :type col1: int
        :type row2: int
        :type col2: int
        :rtype: int
        """
        dp = self.dp
        if not dp:
            return 0
        else:
            # pay attention to the coordinate when using dummy row / column
            return dp[row2 + 1][col2 + 1] - dp[row2 + 1][col1] - dp[row1][col2 + 1] + dp[row1][col1]
```

dp[i][j] is the sum of all elements of the submatrix whose top left is matrix[0][0], and bottom right is matrix[i-1][j-1].