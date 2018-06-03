# Calculators Problems

### 224. Basic Calculator

Implement a basic calculator to evaluate a simple expression string.

The expression string may contain open **(** and closing parentheses **)**, the plus **+** or minus sign **-**, **non-negative** integers and empty spaces.

**Example 1:**

> **Input:** "1 + 1"
> 
> **Output:** 2

**Example 2:**

> **Input:** " 2-1 + 2 "
> 
> **Output:** 3

**Example 3:**

> **Input:** "(1+(4+5+2)-3)+(6+8)"
> 
> **Output:** 23

**Note:**

- You may assume that the given expression is always valid.
- **Do not** use the `eval` built-in library function.

```python
class Solution(object):
    def calculate(self, s):
        """
        :type s: str
        :rtype: int
        """
        idx, res = 0, 0
        stack, flip, isPlus = [True], False, True
        while idx < len(s):
            c = s[idx]
            if c == " ":
                pass
            elif c == "(":
                stack.append(isPlus)
                if not isPlus:
                    flip = not flip
                # imagine "1-(5)"
                # should reset when enter parenthesis
                isPlus = True
            elif c == ")":
                if not stack.pop():
                    flip = not flip
            elif c == "+":
                isPlus = True
            elif c == "-":
                isPlus = False
            else:
                num, ptr = ord(c) - ord("0"), idx + 1
                while ptr < len(s) and s[ptr].isdigit():
                    num = num * 10 + ord(s[ptr]) - ord("0")
                    ptr += 1
                sign = isPlus
                if flip:
                    sign = not sign
                if sign:
                    res += num
                else:
                    res -= num
                idx = ptr - 1
            idx += 1
        return res
```

Expressions only include **+-** and **()**, so we can actually remove all **()** (*i.e.* flatten expression) and do all computations sequentially. The only thing we need to care about is the sign before each number.

Use `stack` to record all signs outside of each level of parenthesis. When we come to a number, all signs recorded in `stack` determine whether the sign right before this number should be flipped. To avoid traversing `stack`, use `flip` to keep track of this. Whenever `stack` changes, `flip` also changes. The rest is simple, use another flag `isPlus` to record the lastest encountered sign, and flip it if necessary.


### 227. Basic Calculator II

Implement a basic calculator to evaluate a simple expression string.

The expression string contains only **non-negative** integers, **+**, **-**, **\***, **/** operators and empty spaces. The integer division should truncate toward zero.

**Example 1:**

> **Input:** "3+2*2"
> 
> **Output:** 7

**Example 2:**

> **Input:** " 3/2 "
> 
> **Output:** 1

**Example 3:**

> **Input:** " 3+5 / 2 "
> 
> **Output:** 5

**Note:**

- You may assume that the given expression is always valid.
- **Do not** use the `eval` built-in library function.

```python
class Solution(object):
    def calculate(self, s):
        """
        :type s: str
        :rtype: int
        """
        idx, res = 0, 0
        prevNum, isPlus, isMul = None, True, None
        while idx < len(s):
            c = s[idx]
            if c == " ":
                pass
            elif c in "+-":
                if isPlus:
                    res += prevNum
                else:
                    res -= prevNum
                isPlus = c == "+"
                isMul = None
            elif c in "*/":
                isMul = c == "*"
            else:
                num, ptr = ord(c) - ord("0"), idx + 1
                while ptr < len(s) and s[ptr].isdigit():
                    num = 10 * num + ord(s[ptr]) - ord("0")
                    ptr += 1
                if isMul is None:
                    prevNum = num
                else:
                    if isMul:
                        prevNum *= num
                    else:
                        prevNum //= num
                idx = ptr - 1
            idx += 1
        
        if isPlus:
            return res + prevNum
        else:
            return res - prevNum
```

All numbers connected by **\*/** are in one component. When we are inside of a component, `res` is kept unchanged. We use `isPlus` to record the sign of this component, and `prevNum` to keep track of the value of the part of this component that has been evaluated. When we encounter **+-**, we know a component ends, and conclude it into `res` using `prevNum` and `isPlus`.

Note that when a component ends, `isMul` is set to `None`, so later when we encounter the first number of the next component, we don't do mul or div, but simply store it in `prevNum`. Also note don't forget to conclude `prevNum` at the very end.

### 772. Basic Calculator III

Implement a basic calculator to evaluate a simple expression string.

The expression string contains only non-negative integers, **+**, **-**, **\***, **/** operators , open **(** and closing parentheses **)** and empty spaces. The integer division should truncate toward zero.

You may assume that the given expression is always valid. All intermediate results will be in the range of **[-2147483648, 2147483647]**.

Some examples:

> "1 + 1" = 2
> 
> " 6-4 / 2 " = 4
> 
> "2\*(5+5\*2)/3+(6/2+8)" = 21
> 
> "(2+6\* 3+5- (3\*14/7+2)\*5)+3"=-12
 

**Note: Do not** use the `eval` built-in library function.

```python
class Solution(object):
    def calculate(self, s):
        """
        :type s: str
        :rtype: int
        """
        def evaluate(expr):
            res = 0
            prevNum, isPlus, isMul = None, True, None
            for e in expr:
                if type(e) is int:
                    if isMul is None:
                        prevNum = e
                    else:
                        if isMul:
                            prevNum *= e
                        else:
                            prevNum //= e
                elif e in "+-":
                    if isPlus:
                        res += prevNum
                    else:
                        res -= prevNum
                    isPlus = e == "+"
                    isMul = None
                elif e in "*/":
                    isMul = e == "*"
                else:
                    print("Should not reach here")
            if isPlus:
                return res + prevNum
            else:
                return res - prevNum
        
        parens, stack = [], []
        i = 0
        while i < len(s):
            c = s[i]
            if c == " ":
                pass
            elif c == "(":
                parens.append(len(stack))
            elif c == ")":
                start = parens.pop()
                res = evaluate(stack[start:])
                del stack[start:]
                stack.append(res)
            elif c.isdigit():
                num, ptr = ord(c) - ord("0"), i + 1
                while ptr < len(s) and s[ptr].isdigit():
                    num = 10 * num + ord(s[ptr]) - ord("0")
                    ptr += 1
                stack.append(num)
                i = ptr - 1
            else:
                stack.append(c)
            i += 1
        return evaluate(stack)
```

`evaluate` is roughly the same to Basic Calculator II. What is different is now we have a outer loop, where `parens` records the position of **(**, and `stack` records expressions. Numbers are converted from `str` to `int` in the outer loop, so in `evaluate` we simply use the element if its type is `int`. **+-\*/** are simply pushed to the stack. When a **(** is encountered, we record the start position, so that later when a **)** is encountered, we start to evaluate the expression between this pair of parentheses. **()** are never pushed to the stack.