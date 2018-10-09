# Text Fitting Problems

### 418. Sentence Screen Fitting

Given a **rows x cols** screen and a sentence represented by a list of **non-empty** words, find **how many times** the given sentence can be fitted on the screen.

**Note:**

1. A word cannot be split into two lines.
2. The order of words in the sentence must remain unchanged.
3. Two consecutive words **in a line** must be separated by a single space.
4. Total words in the sentence won't exceed 100.
5. Length of each word is greater than 0 and won't exceed 10.
6. 1 ≤ rows, cols ≤ 20,000.

**Example 1:**

> **Input:** rows = 2, cols = 8, sentence = ["hello", "world"]
> 
> **Output:** 1
> 
> **Explanation:**
> 
> hello---
> 
> world---
> 
> The character '-' signifies an empty space on the screen.

**Example 2:**

> **Input:** rows = 3, cols = 6, sentence = ["a", "bcd", "e"]
> 
> **Output:** 2
> 
> **Explanation:**
> 
> a-bcd- 
> 
> e-a---
> 
> bcd-e-
> 
> The character '-' signifies an empty space on the screen.

**Example 3:**

> **Input:** rows = 4, cols = 5, sentence = ["I", "had", "apple", "pie"]
> 
> **Output:** 1
> 
> **Explanation:**
> 
> I-had
> 
> apple
> 
> pie-I
> 
> had--
> 
> The character '-' signifies an empty space on the screen.

```python
class Solution:
    def wordsTyping(self, sentence, rows, cols):
        """
        :type sentence: List[str]
        :type rows: int
        :type cols: int
        :rtype: int
        """
        sentence = " ".join(sentence) + " "
        length, ptr = len(sentence), 0
        for _ in range(rows):
            ptr += cols
            while sentence[ptr % length] != " ":
                ptr -= 1
            ptr += 1  # never point to space 
        return ptr // length
```

For every row, we try to advance `ptr` by `cols` units. If it ends up pointing to a character, we let it go back to the first space to its left, and then increase it by one so that it points to a chacracter. After that, all characters and spaces on its left can fit in the screen.

### 68. Text Justification

Given an array of words and a width **maxWidth**, format the text such that each line has exactly **maxWidth** characters and is fully (left and right) justified.

You should pack your words in a greedy approach; that is, pack as many words as you can in each line. Pad extra spaces **' '** when necessary so that each line has exactly **maxWidth** characters.

Extra spaces between words should be distributed as evenly as possible. If the number of spaces on a line do not divide evenly between words, the empty slots on the left will be assigned more spaces than the slots on the right.

For the last line of text, it should be left justified and no **extra** space is inserted between words.

**Note:**

- A word is defined as a character sequence consisting of non-space characters only.
- Each word's length is guaranteed to be greater than 0 and not exceed **maxWidth**.
- The input array **words** contains at least one word.

**Example 1:**

> **Input:**
> 
> words = ["This", "is", "an", "example", "of", "text", "justification."]
> 
> maxWidth = 16
> 
> **Output:**
> 
> [
> 
>    "This    is    an",
> 
>    "example  of text",
> 
>    "justification.  "
> 
> ]

**Example 2:**

> **Input:**
> 
> words = ["What","must","be","acknowledgment","shall","be"]
> 
> maxWidth = 16
> 
> **Output:**
> 
> [
> 
>   "What   must   be",
> 
>   "acknowledgment  ",
> 
>   "shall be        "
> 
> ]
> 
> **Explanation:** 
> 
> Note that the last line is "shall be    " instead of "shall     be", because the last line must be left-justified instead of fully-justified. Note that the second line is also left-justified becase it contains only one word.

**Example 3:**

> **Input:**
> 
> words = ["Science","is","what","we","understand","well","enough","to","explain", "to","a","computer.","Art","is","everything","else","we","do"]
> 
> maxWidth = 20
> 
> **Output:**
> 
> [
> 
>   "Science  is  what we",
> 
>   "understand      well",
> 
>   "enough to explain to",
> 
>   "a  computer.  Art is",
> 
>   "everything  else  we",
> 
>   "do                  "
> 
> ]

```python
class Solution:
    def fullJustify(self, words, maxWidth):
        """
        :type words: List[str]
        :type maxWidth: int
        :rtype: List[str]
        """
        ret, ptr = [], 0
        while ptr < len(words):
            # fill buffer
            buffer, length = [words[ptr]], len(words[ptr])
            ptr += 1
            while ptr < len(words) and length + 1 + len(words[ptr]) <= maxWidth:
                buffer.append(words[ptr])
                length += 1 + len(buffer[-1])
                ptr += 1
            # build line
            if ptr == len(words) or len(buffer) == 1:   # left justification
                ret.append(" ".join(buffer) + " " * (maxWidth - length))
            else:                                       # full justification
                num_blank = len(buffer) - 1
                num_space = (maxWidth - length) // num_blank
                extra = maxWidth - length - num_space * num_blank
                line = [buffer[0]]
                for i in range(num_blank):
                    line.append(" " * (num_space + 1))
                    if i < extra:
                        line.append(" ")
                    line.append(buffer[i + 1])
                ret.append("".join(line))
        return ret
```

We store words in a line in `buffer` and compute `length` of this line assuming that one space is put between adjacent words. When it is not possible to add more words, we build this line and append to `ret`. We always process the first word of the line separately, since for which we don't need to put a space on its left.