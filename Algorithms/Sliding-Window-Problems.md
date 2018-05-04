# Sliding Window Problems

### 340. Longest Substring with At Most K Distinct Characters

Given a string, find the length of the longest substring T that contains at most k distinct characters.

For example, Given s = **“eceba”** and k = 2,

T is "ece" which its length is 3.

```python
class Solution:
    def lengthOfLongestSubstringKDistinct(self, s, k):
        """
        :type s: str
        :type k: int
        :rtype: int
        """
        ret, start = 0, 0  # start: starting index of the current window
        window = collections.defaultdict(int)
        
        for i in range(len(s)):
            window[s[i]] += 1
            
            # if the window contains more than k unique chars, emit 
            # may need to emit more than 1 char, so use while
            while len(window) > k:
                emitMe, start = s[start], start + 1
                window[emitMe] -= 1
                if window[emitMe] == 0:
                    del window[emitMe]
                
            ret = max(i - start + 1, ret)
        
        return ret
```

### 683. K Empty Slots

There is a garden with **N** slots. In each slot, there is a flower. The **N** flowers will bloom one by one in **N** days. In each day, there will be **exactly** one flower blooming and it will be in the status of blooming since then.

Given an array **flowers** consists of number from **1** to **N**. Each number in the array represents the place where the flower will open in that day.

For example, **flowers[i] = x** means that the unique flower that blooms at day **i** will be at position **x**, where **i** and **x** will be in the range from **1** to **N**.

Also given an integer **k**, you need to output in which day there exists two flowers in the status of blooming, and also the number of flowers between them is **k** and these flowers are not blooming.

If there isn't such day, output -1.

Example 1:

> Input: 
>
> flowers: [1,3,2]
>
> k: 1
>
> Output: 2

Explanation: In the second day, the first and the third flower have become blooming.

Example 2:

> Input: 
>
> flowers: [1,2,3]
>
> k: 1
>
> Output: -1

Note:

The given array will be in the range [1, 20000].

```python
class Solution:
    def kEmptySlots(self, flowers, k):
        """
        :type flowers: List[int]
        :type k: int
        :rtype: int
        """
        days = [0] * len(flowers)
        for idx, flower in enumerate(flowers):
            days[flower - 1] = idx + 1
        
        minDay = float("inf")
        left, right = 0, k + 1
        
        for i in range(1, len(days)):
            # if requirement fulfilled, continue
            if days[i] > days[left] and days[i] > days[right]:
                continue
            
            # if successfully reach right edge of window, record it
            if i == right:
                minDay = min(minDay, max(days[left], days[right]))
            
            # move window
            left, right = i, i + k + 1
            if right >= len(days):
                break
        
        return minDay == float("inf") and -1 or minDay
```

Input array **flowers** is first converted to array **days**, whose indices are positions of flowers, and values are blooming days of them. We use a fixed-size sliding window (bounded by **left** and **right**), and we want to find out windows in which all blooming days are **later than days[left] and days[right]**, so that we can say when both flowers at **left** and **right** bloom, all flowers inside of the window haven't bloomed yet.

A trick is used when a valid window is found. At that time we would have to slide the entire window by **1** unit, and start examing from **old left + 1**, but this will make time complexity O(n^2). Actually if **new left** is still to the **left** of **old right**, we will never find another valid window, because **old right** has been proved to be smaller than all those **new left**. Instead of moving the window by **1** unit, we simply set the **new left** to be **old right**.

Besides, if a window is found invalid, we should not slide the window by **1** unit either. If the position being examined is **i**, then we know **days[i]** must be smaller than **either old left or old right**, however, all **days[j]** where **old left < j < i** have been proved to be larger than **both old left and old right**, and thus must be larger than **days[i]**, so we cannot find another valid window if **old left < new left < i**.

Fortunately, these two cases can be merged into one: set **new left** to be **i**.