# Divide and Conquer Problems

### 240. Search a 2D Matrix II

Write an efficient algorithm that searches for a value in an m x n matrix. This matrix has the following properties:

- Integers in each row are sorted in ascending from left to right.
- Integers in each column are sorted in ascending from top to bottom.
Consider the following matrix:

> [
> 
>   [1,   4,  7, 11, 15],
> 
>   [2,   5,  8, 12, 19],
> 
>   [3,   6,  9, 16, 22],
> 
>   [10, 13, 14, 17, 24],
> 
>   [18, 21, 23, 26, 30]
> 
> ]

**Example 1:**

> **Input:** matrix, target = 5
> 
> **Output:** true

**Example 2:**

> **Input:** matrix, target = 20
> 
> **Output:** false

```python
class Solution:
    def searchMatrix(self, matrix, target):
        """
        :type matrix: List[List[int]]
        :type target: int
        :rtype: bool
        """
        if not matrix or not matrix[0]:
            return False
        
        def search(startY, startX, maxY, maxX):
            if startY >= maxY or startX >= maxX:
                return False
            
            # marching through diagonal
            y, x = startY, startX
            while y < maxY and x < maxX and matrix[y][x] < target:
                y, x = y + 1, x + 1
            
            # divide and conquer if target is not on this diagonal
            if y < maxY and x < maxX and matrix[y][x] == target:
                return True
            else:
                return search(startY, x, y, maxX) or search(y, startX, maxY, x)
        
        return search(0, 0, len(matrix), len(matrix[0]))
```

![](https://s3-lc-upload.s3.amazonaws.com/users/shankark/image_1523638489.png)

Numbers are guaranteed to be greater than all other numbers to their left and their top. Numbers on the diagonal are guaranteed to be the greatest in the square defined by themselves. For example, the 17 is larger than any other number in the top left 4x4 square.

We can keep searching through the diagonal until find a number not less than `target`. If it is equal to `target`, we are done; otherwise, we do divide and conquer. For example, if `target` is 16, we first find 17 on the diagonal that is not less than 16. As is shown in the image above, 16 might locate inside any of the two red rectangles, and thus we call `search` recursively.