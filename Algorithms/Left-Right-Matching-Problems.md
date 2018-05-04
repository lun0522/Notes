# Left-Right Matching Problems

**Be careful about when to exit the recursive call, and what to return!**

### 394. Decode String

Given an encoded string, return it's decoded string.

The encoding rule is: **k[encoded_string]**, where the encoded_string inside the square brackets is being repeated exactly k times. Note that k is guaranteed to be a positive integer.

You may assume that the input string is always valid; No extra white spaces, square brackets are well-formed, etc.

Furthermore, you may assume that the original data does not contain any digits and that digits are only for those repeat numbers, k. For example, there won't be input like **3a** or **2[4]**.

Examples:

> s = "3[a]2[bc]", return "aaabcbc".
>
> s = "3[a2[c]]", return "accaccacc".
>
> s = "2[abc]3[cd]ef", return "abcabccdcdcdef".

```python
class Solution(object):
    def decodeString(self, s):
        """
        :type s: str
        :rtype: str
        """
        def decode(ptr):
            ret = []
            while ptr < len(s) and s[ptr] != "]":  
                if s[ptr].isdigit():
                    bra = s.find("[", ptr)
                    repeat = int(s[ptr: bra])
                    segment, ptr = decode(bra + 1)
                    ret.append(segment * repeat)
                else:
                    ret.append(s[ptr])
                    ptr += 1
            return "".join(ret), ptr + 1
        
        return decode(0)[0]
```

- If the entire string has been scanned, or encounter a **]**, return what we have got in `ret`, and the pointer that points outside of the string or to the **]**. The function waiting on the stack can continue scanning after this **]** (consider **"3[a]2[bc]"**, after dealing with **3[a]**, return **aaa**, and restart from **2**)

- If encounter a numeric character, there will be a recursive function call. First find the **[**, and convert numeric characters before it to the integer **repeat**; then do recursive call, and wait for the decoded string and the start position of the following scan

- If encounter a alphabetic character, simply append to `ret` (consider the **a** in **"3[a2[c]]"**)


### 20. Valid Parentheses

Given a string containing just the characters **'('**, **')'**, **'{'**, **'}'**, **'['** and **']'**, determine if the input string is valid.

The brackets must close in the correct order, **"()"** and **"()[]{}"** are all valid but **"(]"** and **"([)]"** are not.

```python
class Solution(object):
    def validate(self, s, l):
        while l < len(s):
            if s[l] not in self.left:
                return True, l
            waitFor = self.left[s[l]]
            valid, nextL = self.validate(s, l + 1)
            if not valid or nextL >= len(s) or s[nextL] != waitFor:
                return False, -1
            l = nextL + 1
        return True, len(s)
    
    def isValid(self, s):
        """
        :type s: str
        :rtype: bool
        """
        self.left = {c1: c2 for c1, c2 in zip("([{", ")]}")}
        valid, lastL = self.validate(s, 0)
        return valid and lastL == len(s)
```

Each function call only validates one pair of parentheses. After it knows the left parenthese (**(** or **[** or **{**), it will know what should it expect for, and store it in `waitFor`. After the recursive call, the returned index must point to a character that equals to `waitFor`. Then go on to check the following part. (consider **[()()]**)

Pay attention to the outmost function call. There we don't have `waitFor`, and thus we require that the returned index must point outside of the string. (consider **]**)

An easier way:

```python
class Solution:
    def isValid(self, s):
        """
        :type s: str
        :rtype: bool
        """
        stack = []
        left, right = "([{", ")]}"
        for c in s:
            if c in left:
                stack.append(c)
            elif c in right:
                match = left[right.index(c)]
                if not stack or stack[-1] != match:
                    return False
                else:
                    stack.pop()
        return not stack
```