# Top K Problems

To find the **K**th largest element in an array using a heap, there are two ways:

1. `heapify` the entire array, and do `heappop` until the **K**th largest element is on the top of the heap. Time complexity: `heapify` is an **O(N)** operation, while the popping is **O(K·logN)**, so **O(N+K·logN)** in total.
2. Maintain a **min**-heap of length **K**: fetch the first **K** elements of the array, `heapify` it; traverse the rest part of the array, if the element is larger than the minimum element in the heap, do `heapreplace`. Finally, the element on the top of the heap is the **K**th largest. Time complexity: **O(N·logK)**.

Without information about **N** and **K**, we cannot tell which method is more efficient.

### 215. Kth Largest Element in an Array

Find the **k**th largest element in an unsorted array. Note that it is the kth largest element in the sorted order, not the kth distinct element.

**Example 1:**

> **Input:** [3,2,1,5,6,4] and k = 2
> 
> **Output:** 5

**Example 2:**

> **Input:** [3,2,3,1,2,4,5,5,6] and k = 4
> 
> **Output:** 4

**Note:**
 
You may assume k is always valid, 1 ≤ k ≤ array's length.

```python
from heapq import *

class Solution1(object):
    def findKthLargest(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        heapify(nums)
        [heappop(nums) for _ in range(len(nums) - k)]
        return nums[0]

class Solution2(object):
    def findKthLargest(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        heap = nums[:k]
        heapify(heap)
        for num in nums[k:]:
            if num > heap[0]:
                heapreplace(heap, num)
        return heap[0]
```

### 692. Top K Frequent Words

Given a non-empty list of words, return the k most frequent elements.

Your answer should be sorted by frequency from highest to lowest. If two words have the same frequency, then the word with the lower alphabetical order comes first.

**Example 1:**

> **Input:** ["i", "love", "leetcode", "i", "love", "coding"], k = 2
> 
> **Output:** ["i", "love"]
> 
> **Explanation:** "i" and "love" are the two most frequent words. Note that "i" comes before "love" due to a lower alphabetical order.

**Example 2:**

> **Input:** ["the", "day", "is", "sunny", "the", "the", "the", "sunny", "is", "is"], k = 4
> 
> **Output:** ["the", "is", "sunny", "day"]
> 
> **Explanation:** "the", "is", "sunny" and "day" are the four most frequent words, with the number of occurrence being 4, 3, 2 and 1 respectively.

**Note:**

1. You may assume k is always valid, 1 ≤ k ≤ number of unique elements.
2. Input words contain only lowercase letters.

**Follow up:**

Try to solve it in O(n log k) time and O(n) extra space.

```python
from heapq import *

class Solution:
    def topKFrequent(self, words, k):
        """
        :type words: List[str]
        :type k: int
        :rtype: List[str]
        """
        counter = collections.Counter(words)
        heap = [[-count, word] for word, count in counter.items()]
        heapify(heap)
        return [heappop(heap)[1] for _ in range(k)]
```

The above code uses the first method, so the time complexity is O(N+K·logN). If we want to use the second method, the heap should be a min-heap for `count`, and max-heap for `word` (because "the word with the lower alphabetical order comes first"). The latter one requires some extra efforts (*e.g.* converting each word to a tuple of ascii, and negating).