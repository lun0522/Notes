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
class Solution:
    def calculate(self, s):
        """
        :type s: str
        :rtype: int
        """
        ops = {"+": lambda x, y: x + y, "-": lambda x, y: x - y}
        def evaluate(i):
            ret, op = 0, ops["+"]
            while True:
                if i == len(s) or s[i] == ")":
                    return ret, i
                else:
                    c = s[i]
                    if c == " ":
                        pass
                    elif c == "(":
                        val, i = evaluate(i + 1)
                        ret = op(ret, val)
                    elif c in ops:
                        op = ops[c]
                    else:
                        num = 0
                        while i < len(s) and s[i].isnumeric():
                            num = num * 10 + int(s[i])
                            i += 1
                        ret = op(ret, num)
                        continue
                    i += 1
        
        return evaluate(0)[0] 
```

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
class Solution:
    def calculate(self, s):
        """
        :type s: str
        :rtype: int
        """
        ops = {"+": lambda x, y: x + y, "-": lambda x, y: x - y,
               "*": lambda x, y: x * y, "/": lambda x, y: x // y}
        
        i = 0
        stack = []
        while i < len(s):
            c = s[i]
            if c == " ":
                pass
            elif c in "+-*/":
                stack.append(c)
            else:  # number
                num = 0
                while i < len(s) and s[i].isnumeric():
                    num = num * 10 + ord(s[i]) - ord('0')
                    i += 1
                if stack and stack[-1] in "*/":
                    op = ops[stack.pop()]
                    stack.append(op(stack.pop(), num))
                else:
                    stack.append(num)
                continue
            i += 1
        
        if not stack:
            return 0
        ret = stack[0]
        for i in range(1, len(stack), 2):
            ret = ops[stack[i]](ret, stack[i + 1])
        return ret
```

We scan twice. The first scan is to pre-process `s`. If we encounter addition or subtraction, simply push it to `stack`. If we encounter multiplication or division, immediately evaluate it. In the second scan, we only deal with additions and subtractions.

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
class Solution:
    def calculate(self, s):
        """
        :type s: str
        :rtype: int
        """
        ops = {"+": lambda x, y: x + y, "-": lambda x, y: x - y,
               "*": lambda x, y: x * y, "/": lambda x, y: x // y}
        
        def evaluate(i):
            stack = []
            while i < len(s):
                c = s[i]
                if c == " ":
                    pass
                elif c in "+-*/":
                    stack.append(c)
                elif c == ")":
                    i += 1
                    break
                else:  # number or left parenthesis
                    if c == "(":
                        num, i = evaluate(i + 1)
                    else:
                        num = 0
                        while i < len(s) and s[i].isnumeric():
                            num = num * 10 + ord(s[i]) - ord('0')
                            i += 1
                    if stack and stack[-1] in "*/":
                        op = ops[stack.pop()]
                        stack.append(op(stack.pop(), num))
                    else:
                        stack.append(num)
                    continue
                i += 1

            if not stack:
                return 0, i
            ret = stack[0]
            for j in range(1, len(stack), 2):
                ret = ops[stack[j]](ret, stack[j + 1])
            return ret, i
        
        return evaluate(0)[0]
```