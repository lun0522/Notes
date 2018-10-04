# Random Number Problems

### 528. Random Pick with Weight

Given an array **w** of positive integers, where **w[i]** describes the weight of index **i**, write a function **pickIndex** which randomly picks an index in proportion to its weight.

**Note:**

1. **1 <= w.length <= 10000**
2. **1 <= w[i] <= 10^5**
3. **pickIndex** will be called at most **10000** times.

**Example 1:**

> **Input:**
> 
> ["Solution","pickIndex"]
> 
> [[[1]],[]]
> 
> **Output:** [null,0]

**Example 2:**

> **Input:**
> 
> ["Solution","pickIndex","pickIndex","pickIndex","pickIndex","pickIndex"]
> 
> [[[1,3]],[],[],[],[],[]]
> 
> **Output:** [null,0,1,1,1,0]

**Explanation of Input Syntax:**

The input is two lists: the subroutines called and their arguments. **Solution**'s constructor has one argument, the array **w**. pickIndex has no arguments. Arguments are always wrapped with a list, even if there aren't any.

```python
class Solution:

    def __init__(self, w):
        """
        :type w: List[int]
        """
        for i in range(1, len(w)):
            w[i] += w[i - 1]
        self.w = w

    def pickIndex(self):
        """
        :rtype: int
        """
        return bisect.bisect_left(self.w, random.randint(1, self.w[-1]))
```

We accumulate weights and then do a binary search. For example, if input weights are **[1, 2, 3]**, we set `self.w` to **[1, 3, 6]**. Then we generate a random number in range **[1, 6]**. If it is in range **[1, 1]**, output **0**; if in **[2, 3]**, output **1**; if in **[4, 6]**, output **2**;

### 710. Random Pick with Blacklist

Given a blacklist **B** containing unique integers from **[0, N)**, write a function to return a uniform random integer from **[0, N)** which is **NOT** in **B**.

Optimize it such that it minimizes the call to system’s **Math.random()**.

**Note:**

1. **1 <= N <= 1000000000**
2. **0 <= B.length < min(100000, N)**
3. **[0, N)** does NOT include N. See interval notation.

**Example 1:**

> **Input:**
> 
> ["Solution","pick","pick","pick"]
> 
> [[1,[]],[],[],[]]
> 
> **Output:** [null,0,0,0]

**Example 2:**

> **Input:** 
> 
> ["Solution","pick","pick","pick"]
> 
> [[2,[]],[],[],[]]
> 
> **Output:** [null,1,1,1]

**Example 3:**

> **Input:**
> 
> ["Solution","pick","pick","pick"]
> 
> [[3,[1]],[],[],[]]
> 
> **Output:** [null,0,0,2]

**Example 4:**

> **Input:** 
> 
> ["Solution","pick","pick","pick"]
> 
> [[4,[2]],[],[],[]]
> 
> **Output:** [null,1,3,1]

**Explanation of Input Syntax:**

The input is two lists: the subroutines called and their arguments. **Solution**'s constructor has two arguments, **N** and the blacklist **B**. pick has no arguments. Arguments are always wrapped with a list, even if there aren't any.

```python
class Solution(object):

    def __init__(self, N, blacklist):
        """
        :type N: int
        :type blacklist: List[int]
        """
        self.N = N - len(blacklist)
        blacklist = set(blacklist)
        self.valid = {}
        next_valid = self.N
        for num in blacklist:
            if num < self.N:
                while next_valid in blacklist:
                    next_valid += 1
                self.valid[num] = next_valid
                next_valid += 1

    def pick(self):
        """
        :rtype: int
        """
        ret = random.randint(0, self.N - 1)
        return self.valid.get(ret, ret)
```

We set `self.N` to the number of possible outputs. Later we'll generate random values in range **[0, `self.N`)**. Within this range, some numbers are invalid, so we will map them to a valid number that is outside of the range **[0, `self.N`)**. We use `next_valid` to find such a number, which runs from `self.N` to `N - 1` and will **not** take a value that presents in `blacklist`. This mapping relationship is stored in `self.valid`.

For example, if `N` is **6** and `blacklist` is **[1, 5]**, possible outputs are **0, 2, 3, 4**. We set `self.N` to **4**. Within the range **[0, 4)**, **1** is invalid, so we use `next_valid` to find **4** and map **1** to it. If the generated random values are **0, 1, 2, 3**, corresponding outputs will be **0, 4, 2, 3**.
