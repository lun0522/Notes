# Buy and Sell Stock Problems

### 121. Best Time to Buy and Sell Stock

Say you have an array for which the **i**th element is the price of a given stock on day **i**.

If you were only permitted to complete at most one transaction (i.e., buy one and sell one share of the stock), design an algorithm to find the maximum profit.

Note that you cannot sell a stock before you buy one.

**Example 1:**

> **Input:** [7,1,5,3,6,4]
> 
> **Output:** 5
> 
> **Explanation:** Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6-1 = 5. Not 7-1 = 6, as selling price needs to be larger than buying price.

**Example 2:**

> **Input:** [7,6,4,3,1]
> 
> **Output:** 0
> 
> **Explanation:** In this case, no transaction is done, i.e. max profit = 0.

```python
class Solution:
    def maxProfit(self, prices):
        """
        :type prices: List[int]
        :rtype: int
        """
        minLeft, maxPro = float("inf"), 0
        for price in prices:
            maxPro = max(maxPro, price - minLeft)
            minLeft = min(minLeft, price)
        return maxPro
```

Since only one transaction is allowed, when we are in day **i**, we only care about the minimum price occurred before day **i**, which is tracked by `minLeft`.

### 122. Best Time to Buy and Sell Stock II

Say you have an array for which the **i**th element is the price of a given stock on day **i**.

Design an algorithm to find the maximum profit. You may complete as many transactions as you like (i.e., buy one and sell one share of the stock multiple times).

**Note:** 

You may not engage in multiple transactions at the same time (i.e., you must sell the stock before you buy again).

**Example 1:**

> **Input:** [7,1,5,3,6,4]
> 
> **Output:** 7
> 
> **Explanation:** Buy on day 2 (price = 1) and sell on day 3 (price = 5), profit = 5-1 = 4. Then buy on day 4 (price = 3) and sell on day 5 (price = 6), profit = 6-3 = 3.

**Example 2:**

> **Input:** [1,2,3,4,5]
> 
> **Output:** 4
> 
> **Explanation:** Buy on day 1 (price = 1) and sell on day 5 (price = 5), profit = 5-1 = 4. Note that you cannot buy on day 1, buy on day 2 and sell them later, as you are engaging multiple transactions at the same time. You must sell before buying again.

**Example 3:**

> **Input:** [7,6,4,3,1]
> 
> **Output:** 0
> 
> **Explanation:** In this case, no transaction is done, i.e. max profit = 0.

```python
class Solution:
    def maxProfit(self, prices):
        """
        :type prices: List[int]
        :rtype: int
        """
        profit = 0
        for i in range(1, len(prices)):
            profit += max(0, prices[i] - prices[i - 1])
        return profit
```

Since we can do as many transactions as we want, we can always capture the positive profit, and sell the stock whenever the price is about to decrease.

### 188. Best Time to Buy and Sell Stock IV

Say you have an array for which the **i**th element is the price of a given stock on day **i**.

Design an algorithm to find the maximum profit. You may complete at most **k** transactions.

**Note:**

You may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).

**Example 1:**

> **Input:** [2,4,1], k = 2
> 
> **Output:** 2
> 
> **Explanation:** Buy on day 1 (price = 2) and sell on day 2 (price = 4), profit = 4-2 = 2.

**Example 2:**

> **Input:** [3,2,6,5,0,3], k = 2
> 
> **Output:** 7
> 
> **Explanation:** Buy on day 2 (price = 2) and sell on day 3 (price = 6), profit = 6-2 = 4. Then buy on day 5 (price = 0) and sell on day 6 (price = 3), profit = 3-0 = 3.

```python
class Solution:
    def maxProfit(self, k, prices):
        """
        :type k: int
        :type prices: List[int]
        :rtype: int
        """
        if k >= len(prices) - 1:
            return prices and sum([max(0, day2 - day1) for day1, day2 
                                   in zip(prices[:-1], prices[1:])]) or 0
        
        hold = [float("-inf")] * (k + 1)
        release = [float("-inf")] * (k + 1)
        release[0] = 0
        for price in prices:
            for i in range(1, k + 1):
                hold[i] = max(hold[i], release[i - 1] - price)
                release[i] = max(release[i], hold[i] + price)
        return max(release)
```

There are two states: `hold` (have stock on hand) and `release` (no stock on hand). Since we can do at most **k** transactions, we create arrays of length **k+1** to store states. `hold[i]` means the maximum profit if we have bought the stock for `i` times, and we hold the stock currently; `release[i]` means the maximum profit if we have sold the stock for `i` times, and have no stock on hand. 

`hold[0]` is not achievable since we cannot hold the stock if no buying ever happened. `release[0]` is the only possible state at the beginning. If we already held the stock, we can choose to keep it (`hold[i]` -> `hold[i]`) or sell it (`hold[i] + price` -> `release[i]`). If we didn't hold the stock, we can buy it (`release[i] - price` -> `hold[i + 1]`) or do nothing (`release[i]` -> `release[i]`).

Note that if there are **n** days in total, we can do at most **n - 1** transactions (*i.e.* buy-sell pairs). So, if **k >= n - 1**,  this problem degenerates to *122. Best Time to Buy and Sell Stock II*, where we are allowed to make unlimited number of transactions.

### 309. Best Time to Buy and Sell Stock with Cooldown

Say you have an array for which the **i**th element is the price of a given stock on day **i**.

Design an algorithm to find the maximum profit. You may complete as many transactions as you like (ie, buy one and sell one share of the stock multiple times) with the following restrictions:

- You may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).
- After you sell your stock, you cannot buy stock on next day. (ie, cooldown 1 day).

**Example:**

> **Input:** [1,2,3,0,2]
> 
> **Output:** 3 
> 
> **Explanation:** transactions = [buy, sell, cooldown, buy, sell]

```python
class Solution:
    def maxProfit(self, prices):
        """
        :type prices: List[int]
        :rtype: int
        """
        hold, release, cooldown = float("-inf"), 0, float("-inf")
        for price in prices:
            prevHold = hold
            hold = max(hold, release - price)
            release = max(release, cooldown)
            cooldown = prevHold + price
        return max(release, cooldown)
```

We are allowed to do as many transactions as we want. Now there are three states: `hold` (have stock on hand), `release` (sold stock before yesterday) and `cooldown` (sold stock yesterday). Rules of update:

- For `hold`, if we already held the stock yesterday, we can keep holding it; otherwise, if we were not in the `cooldown` state, we can buy the stock today.
- For `release`, if we didn't hold the stock yesterday, we can be in the `release` states today.
- For `cooldown`, only if we held the stock yesterday, and sell it today, we can get to this state.