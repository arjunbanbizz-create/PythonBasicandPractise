# Python Dynamic Programming — Interview Snippets & Pitfalls

## 1. DP Framework
```python
# Every DP problem = these 3 things:
# 1. State definition — what does dp[i] represent?
# 2. Transition — how does dp[i] relate to previous states?
# 3. Base case — starting values

# Top-down (memoization)
from functools import lru_cache
@lru_cache(maxsize=None)
def dp(state): ...

# Bottom-up (tabulation)
dp = [base] * (n+1)
for i in range(1, n+1):
    dp[i] = transition(dp[i-1], ...)

# ⚠️ Pitfall: off-by-one — dp array usually size n+1 to handle index 0 as base
# ⚠️ Pitfall: define state clearly before coding — wrong state = wrong solution
```

---

## 2. Fibonacci / Linear DP
```python
# Classic Fibonacci — O(n) time O(1) space
def fib(n):
    if n <= 1: return n
    a, b = 0, 1
    for _ in range(2, n+1):
        a, b = b, a+b
    return b

# Climbing stairs (1 or 2 steps)
def climb_stairs(n):
    if n <= 2: return n
    a, b = 1, 2
    for _ in range(3, n+1):
        a, b = b, a+b
    return b

# ⚠️ Pitfall: space-optimize by keeping only last 2 values when dp[i] only needs dp[i-1], dp[i-2]
```

---

## 3. 0/1 Knapsack
```python
# dp[i][w] = max value using first i items with capacity w
def knapsack(weights, values, capacity):
    n = len(weights)
    dp = [[0]*(capacity+1) for _ in range(n+1)]
    for i in range(1, n+1):
        for w in range(capacity+1):
            dp[i][w] = dp[i-1][w]                      # skip item i
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i][w],
                               dp[i-1][w-weights[i-1]] + values[i-1])  # take item i
    return dp[n][capacity]

# Space-optimized (1D) — iterate capacity in reverse!
def knapsack_1d(weights, values, capacity):
    dp = [0] * (capacity+1)
    for i in range(len(weights)):
        for w in range(capacity, weights[i]-1, -1):     # ✅ reverse to avoid reuse
            dp[w] = max(dp[w], dp[w-weights[i]] + values[i])
    return dp[capacity]

# ⚠️ Pitfall: 1D knapsack MUST iterate capacity in reverse — forward would reuse item
# ⚠️ Pitfall: unbounded knapsack (reuse allowed) iterates FORWARD
```

---

## 4. Unbounded Knapsack / Coin Change
```python
# Min coins to make amount (each coin reusable)
def coin_change(coins, amount):
    dp = [float('inf')] * (amount+1)
    dp[0] = 0
    for a in range(1, amount+1):
        for c in coins:
            if c <= a:
                dp[a] = min(dp[a], dp[a-c] + 1)
    return dp[amount] if dp[amount] != float('inf') else -1

# Count ways to make amount
def coin_change_ways(coins, amount):
    dp = [0] * (amount+1)
    dp[0] = 1                           # one way to make 0
    for c in coins:
        for a in range(c, amount+1):    # ✅ forward — reuse allowed
            dp[a] += dp[a-c]
    return dp[amount]

# ⚠️ Pitfall: min coins — init with inf, base dp[0]=0
# ⚠️ Pitfall: count ways — outer loop is COINS, inner is AMOUNT (avoids permutation counting)
# ⚠️ Pitfall: swap loops → counts permutations not combinations
```

---

## 5. Longest Common Subsequence (LCS)
```python
def lcs(s1, s2):
    m, n = len(s1), len(s2)
    dp = [[0]*(n+1) for _ in range(m+1)]
    for i in range(1, m+1):
        for j in range(1, n+1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]

# ⚠️ Pitfall: dp indices are 1-based but string indices are 0-based → s1[i-1]
# ⚠️ Pitfall: table is (m+1) x (n+1) for empty string base cases
```

---

## 6. Longest Increasing Subsequence (LIS)
```python
# O(n²) DP
def lis_n2(nums):
    dp = [1] * len(nums)
    for i in range(1, len(nums)):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j]+1)
    return max(dp)

# O(n log n) — patience sort with bisect
def lis_nlogn(nums):
    tails = []
    for n in nums:
        import bisect
        idx = bisect.bisect_left(tails, n)
        if idx == len(tails): tails.append(n)
        else:                  tails[idx] = n
    return len(tails)

# ⚠️ Pitfall: tails array is NOT the actual LIS — only its length is correct
# ⚠️ Pitfall: bisect_left for strictly increasing; bisect_right for non-decreasing
```

---

## 7. Edit Distance
```python
def edit_distance(w1, w2):
    m, n = len(w1), len(w2)
    dp = [[0]*(n+1) for _ in range(m+1)]
    for i in range(m+1): dp[i][0] = i      # delete all chars
    for j in range(n+1): dp[0][j] = j      # insert all chars
    for i in range(1, m+1):
        for j in range(1, n+1):
            if w1[i-1] == w2[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(
                    dp[i-1][j],     # delete from w1
                    dp[i][j-1],     # insert into w1
                    dp[i-1][j-1]    # replace
                )
    return dp[m][n]

# ⚠️ Pitfall: base cases — dp[i][0]=i and dp[0][j]=j (cost to reach empty string)
```

---

## 8. House Robber Pattern
```python
# No two adjacent houses
def rob(nums):
    prev2 = prev1 = 0
    for n in nums:
        prev2, prev1 = prev1, max(prev1, prev2+n)
    return prev1

# Circular array (first and last are adjacent)
def rob_circular(nums):
    def rob_linear(arr):
        p2 = p1 = 0
        for n in arr:
            p2, p1 = p1, max(p1, p2+n)
        return p1
    return max(nums[0], rob_linear(nums[1:]), rob_linear(nums[:-1]))

# ⚠️ Pitfall: circular — run rob_linear twice excluding first OR last element
```

---

## 9. 2D Grid DP
```python
# Unique paths
def unique_paths(m, n):
    dp = [[1]*n for _ in range(m)]
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i-1][j] + dp[i][j-1]
    return dp[m-1][n-1]

# Minimum path sum
def min_path_sum(grid):
    m, n = len(grid), len(grid[0])
    for i in range(m):
        for j in range(n):
            if i == 0 and j == 0: continue
            top  = grid[i-1][j] if i > 0 else float('inf')
            left = grid[i][j-1] if j > 0 else float('inf')
            grid[i][j] += min(top, left)    # modify in-place ✅
    return grid[m-1][n-1]

# ⚠️ Pitfall: boundary check for top row (i==0) and left col (j==0) separately
```

---

## 10. Interval DP
```python
# Burst Balloons — dp[l][r] = max coins for subarray nums[l..r]
def max_coins(nums):
    nums = [1] + nums + [1]             # pad with 1s
    n = len(nums)
    dp = [[0]*n for _ in range(n)]
    for length in range(2, n):          # window size
        for l in range(n-length):
            r = l + length
            for k in range(l+1, r):     # k = last balloon to burst in [l,r]
                dp[l][r] = max(dp[l][r],
                               nums[l]*nums[k]*nums[r] + dp[l][k] + dp[k][r])
    return dp[0][n-1]

# ⚠️ Pitfall: iterate by LENGTH then left boundary — not row by row
# ⚠️ Pitfall: k is the LAST balloon burst — not first
```

---

## 11. State Machine DP (Stock Problems)
```python
# Buy/sell stock with cooldown
def max_profit_cooldown(prices):
    hold = float('-inf')    # max profit holding stock
    sold = 0                # max profit just sold (cooldown next)
    rest = 0                # max profit resting
    for p in prices:
        hold, sold, rest = max(hold, rest-p), hold+p, max(rest, sold)
    return max(sold, rest)

# ⚠️ Pitfall: update ALL states simultaneously — use previous values on right side
# ⚠️ Pitfall: hold initialized to -inf (haven't bought yet)
```

---

## 12. Key Pitfalls Cheatsheet

| ⚠️ Pitfall | Fix |
|---|---|
| dp array size n not n+1 | use `n+1` for 0-indexed base case |
| 0/1 knapsack forward 1D loop | iterate capacity in **reverse** |
| Coin ways — permutations vs combinations | outer=coins, inner=amount for combinations |
| LIS tails ≠ actual subsequence | only length is valid |
| `bisect_left` vs `bisect_right` for LIS | left=strict, right=non-decreasing |
| Edit distance missing base cases | `dp[i][0]=i`, `dp[0][j]=j` |
| Circular rob — using full array | run rob twice: `nums[1:]` and `nums[:-1]` |
| State machine — using updated values | assign all states simultaneously |
| Interval DP — iterating row by row | iterate by **window length** first |
| LCS string index 1-based | access string with `s[i-1]` not `s[i]` |
