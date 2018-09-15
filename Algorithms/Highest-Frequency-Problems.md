# Highest Frequency Problems

### 424. Longest Repeating Character Replacement

Given a string that consists of only uppercase English letters, you can replace any letter in the string with another letter at most **k** times. Find the length of a longest substring containing all repeating letters you can get after performing the above operations.

**Note:**

Both the string's length and k will not exceed 10^4.

**Example 1:**

> **Input:**
> 
> s = "ABAB", k = 2
>
> **Output:**
> 
> 4
> 
> **Explanation:**
> 
> Replace the two 'A's with two 'B's or vice versa.

**Example 2:**

> **Input:**
>
> s = "AABABBA", k = 1
> 
> **Output:**
> 
> 4
> 
> **Explanation:**
> 
> Replace the one 'A' in the middle with 'B' and form "AABBBBA".
The substring "BBBB" has the longest repeating letters, which is 4.

```python
class Solution:
    def characterReplacement(self, s, k):
        """
        :type s: str
        :type k: int
        :rtype: int
        """
        counter = collections.defaultdict(int)
        freqs = [0]
        
        def include(ch):
            counter[ch] += 1
            if counter[ch] > len(freqs) - 1:
                freqs.append(0)
            freqs[counter[ch]] += 1
        
        def exclude(ch):
            freqs[counter[ch]] -= 1
            if not counter[ch]:
                freqs.pop()
            counter[ch] -= 1
        
        start, longest = 0, 0
        for end, char in enumerate(s):
            include(char)
            while (end - start + 1) - (len(freqs) - 1) > k:
                exclude(s[start])
                start += 1
            longest = max(longest, end - start + 1)
        
        return longest
```

In a valid window, if one character dominates, we can tolerate at most **k** other characters, so we must make sure that: window size - frequency of the dominating character <= **k**. We don't care which character dominates; we only need to know the highest frequency.

To keep track of that, we actually don't need to sort a counter or use a heap. When the window moves, frequencies change continuously, so the highest frequency changes by **at most 1** each time. We can use a stack to track existing frequencies, and read the maximum frequency from the length of this stack.

In the code above, `counter` maps each character to its frequency inside of the window, while `freqs` is a stack, where `stack[i]` represents how many characters have appearred for **at least `i`** times inside of the window. (Don't use `stack[i]` to represent how many cahracter have appearred for exactly `i` times, otherwise when the frequency of a character increases, we always have to remove it from the old frequency slot, which is unnecessary)

### 895. Maximum Frequency Stack

Implement **FreqStack**, a class which simulates the operation of a stack-like data structure.

**FreqStack** has two functions:

- **push(int x)**, which pushes an integer x onto the stack.
- **pop()**, which **removes** and returns the most frequent element in the stack. (If there is a tie for most frequent element, the element closest to the top of the stack is removed and returned)
 

**Example 1:**

> **Input:**
> 
> ["FreqStack","push","push","push","push","push","push","pop","pop","pop","pop"],
[[],[5],[7],[5],[7],[4],[5],[],[],[],[]]
> 
> **Output:** 
> 
> [null,null,null,null,null,null,null,5,7,5,4]
> 
> **Explanation:**
> 
> After making six .push operations, the stack is [5,7,5,7,4,5] from bottom to top.  Then:
> 
> pop() -> returns 5, as 5 is the most frequent.
> 
> The stack becomes [5,7,5,7,4].
> 
> pop() -> returns 7, as 5 and 7 is the most frequent, but 7 is closest to the top.
> 
> The stack becomes [5,7,5,4].
> 
> pop() -> returns 5.
> 
> The stack becomes [5,7,4].
> 
> pop() -> returns 4.
> 
> The stack becomes [5,7].
 

**Note:**

- Calls to **FreqStack.push(int x)** will be such that **0 <= x <= 10^9**.
- It is guaranteed that **FreqStack.pop()** won't be called if the stack has zero elements.
- The total number of **FreqStack.push** calls will not exceed **10000** in a single test case.
- The total number of **FreqStack.pop** calls will not exceed **10000** in a single test case.
- The total number of **FreqStack.push** and **FreqStack.pop** calls will not exceed **150000** across all test cases.

```python
class FreqStack:

    def __init__(self):
        self.counter = collections.defaultdict(int)
        self.freqs = [[]]

    def push(self, x):
        """
        :type x: int
        :rtype: void
        """
        self.counter[x] += 1
        if self.counter[x] > len(self.freqs) - 1:
            self.freqs.append([])
        self.freqs[self.counter[x]].append(x)

    def pop(self):
        """
        :rtype: int
        """
        x = self.freqs[-1].pop()
        self.counter[x] -= 1
        if not self.freqs[-1]:
            self.freqs.pop()
        return x
```

Similarily, in the code above, `self.counter` maps each character to its frequency, while `self.freqs` is a stack of stack, where `stack[i]` stacks all numbers of frequency **at least `i`**. When we need to get numbers of the highest frequency, we go to `self.freqs[-1]`. Since "if there is a tie for most frequent element, the element closest to the top of the stack is removed and returned", we simply pop out `stack[-1][-1]`.

Note that after a number is pushed onto the stack and its frequency increases by 1, we don't remove it from the old frequency slot, so that the history of `.push` is preserved, which will be useful for the future `.pop`.