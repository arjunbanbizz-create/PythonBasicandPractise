# Python Recursion & Backtracking — Interview Snippets & Pitfalls

---

# RECURSION

## 1. Recursion Template
```python
def solve(params):
    # 1. Base case
    if base_condition:
        return base_value
    # 2. Recursive case
    return combine(solve(smaller_params))

# ⚠️ Pitfall: always define base case FIRST — missing it causes infinite recursion
# ⚠️ Pitfall: ensure params get strictly smaller — or use memoization
```

---

## 2. Memoization Template
```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    if n <= 1: return n
    return fib(n-1) + fib(n-2)

# Manual memo dict
def fib(n, memo={}):                    # ⚠️ mutable default — shared across calls!
    pass

# ✅ Safe manual memo
def fib(n, memo=None):
    if memo is None: memo = {}
    if n in memo: return memo[n]
    if n <= 1: return n
    memo[n] = fib(n-1, memo) + fib(n-2, memo)
    return memo[n]

# ⚠️ Pitfall: lru_cache args must be HASHABLE — no lists or dicts as params
# ⚠️ Pitfall: mutable default dict persists between calls — use None sentinel
```

---

## 3. Recursion → Iteration (Stack)
```python
# Any recursion can be converted to iterative with explicit stack
# Useful when Python's recursion limit (default 1000) is hit

import sys
sys.setrecursionlimit(10**6)            # ✅ increase limit for deep recursion

# Convert DFS recursion to iterative
def dfs_iterative(root):
    if not root: return
    stack = [root]
    while stack:
        node = stack.pop()
        process(node)
        if node.right: stack.append(node.right)
        if node.left:  stack.append(node.left)

# ⚠️ Pitfall: default recursion limit is 1000 — deep trees/graphs will crash
```

---
---

# BACKTRACKING

## 4. Backtracking Template
```python
def backtrack(state, choices):
    if is_solution(state):
        result.append(state[:])        # ✅ append COPY not reference
        return
    for choice in choices:
        if is_valid(choice, state):
            state.append(choice)       # choose
            backtrack(state, next_choices)
            state.pop()                # un-choose (backtrack) ✅

# ⚠️ Pitfall: append state[:] or list(state) — not state itself (it gets modified)
# ⚠️ Pitfall: always undo the choice after recursion returns
```

---

## 5. Subsets
```python
def subsets(nums):
    res, subset = [], []
    def backtrack(i):
        res.append(subset[:])           # snapshot at every node
        for j in range(i, len(nums)):
            subset.append(nums[j])
            backtrack(j+1)
            subset.pop()
    backtrack(0)
    return res

# ⚠️ Pitfall: append snapshot at EVERY call (not just base case) — subsets include all lengths
# ⚠️ Pitfall: pass j+1 (not i+1) to avoid re-using current element
```

---

## 6. Subsets with Duplicates
```python
def subsets_with_dup(nums):
    nums.sort()                         # ✅ must sort first
    res, subset = [], []
    def backtrack(i):
        res.append(subset[:])
        for j in range(i, len(nums)):
            if j > i and nums[j] == nums[j-1]: continue   # skip dup
            subset.append(nums[j])
            backtrack(j+1)
            subset.pop()
    backtrack(0)
    return res

# ⚠️ Pitfall: sort first — duplicates must be adjacent to skip correctly
# ⚠️ Pitfall: skip condition is j > i (not j > 0) — only skip within same level
```

---

## 7. Permutations
```python
def permutations(nums):
    res = []
    def backtrack(start):
        if start == len(nums):
            res.append(nums[:])         # full permutation reached
            return
        for i in range(start, len(nums)):
            nums[start], nums[i] = nums[i], nums[start]   # swap in
            backtrack(start+1)
            nums[start], nums[i] = nums[i], nums[start]   # swap back

    backtrack(0)
    return res

# ⚠️ Pitfall: swap back to restore original order after recursion
# ⚠️ Pitfall: append nums[:] copy — not nums reference
```

---

## 8. Permutations with Duplicates
```python
def perms_unique(nums):
    nums.sort()
    res, used = [], [False] * len(nums)
    def backtrack(path):
        if len(path) == len(nums):
            res.append(path[:]); return
        for i in range(len(nums)):
            if used[i]: continue
            if i > 0 and nums[i] == nums[i-1] and not used[i-1]: continue  # skip dup
            used[i] = True
            path.append(nums[i])
            backtrack(path)
            path.pop()
            used[i] = False
    backtrack([])
    return res

# ⚠️ Pitfall: skip dup only when nums[i-1] was NOT used in current path
```

---

## 9. Combinations
```python
def combinations(n, k):
    res, combo = [], []
    def backtrack(start):
        if len(combo) == k:
            res.append(combo[:]); return
        for i in range(start, n+1):
            combo.append(i)
            backtrack(i+1)
            combo.pop()
    backtrack(1)
    return res

# Pruning optimization
def backtrack_pruned(start):
    if len(combo) == k:
        res.append(combo[:]); return
    need = k - len(combo)
    for i in range(start, n - need + 2):   # ✅ prune: not enough elements left
        combo.append(i)
        backtrack_pruned(i+1)
        combo.pop()

# ⚠️ Pitfall: pruning bound is n - need + 2 (1-indexed) — easy to off-by-one
```

---

## 10. Combination Sum (Reuse Allowed)
```python
def combination_sum(candidates, target):
    res, path = [], []
    def backtrack(i, total):
        if total == target:
            res.append(path[:]); return
        if total > target or i >= len(candidates): return
        path.append(candidates[i])
        backtrack(i, total + candidates[i])     # ✅ same i — reuse allowed
        path.pop()
        backtrack(i+1, total)                   # skip current candidate
    backtrack(0, 0)
    return res

# ⚠️ Pitfall: pass same i (not i+1) to allow reusing current element
# ⚠️ Pitfall: check total > target early — prune invalid branches
```

---

## 11. N-Queens
```python
def solve_n_queens(n):
    res = []
    cols = set(); pos_diag = set(); neg_diag = set()

    board = [['.']*n for _ in range(n)]
    def backtrack(row):
        if row == n:
            res.append(["".join(r) for r in board]); return
        for col in range(n):
            if col in cols or (row+col) in pos_diag or (row-col) in neg_diag:
                continue
            cols.add(col); pos_diag.add(row+col); neg_diag.add(row-col)
            board[row][col] = 'Q'
            backtrack(row+1)
            cols.remove(col); pos_diag.remove(row+col); neg_diag.remove(row-col)
            board[row][col] = '.'
    backtrack(0)
    return res

# ⚠️ Pitfall: diagonals — (row+col) same for / diagonal, (row-col) same for \ diagonal
# ⚠️ Pitfall: remove from sets AND restore board on backtrack
```

---

## 12. Word Search in Grid
```python
def word_search(board, word):
    rows, cols = len(board), len(board[0])
    def dfs(r, c, i):
        if i == len(word): return True
        if r<0 or r>=rows or c<0 or c>=cols or board[r][c] != word[i]: return False
        tmp, board[r][c] = board[r][c], '#'    # mark visited
        found = any(dfs(r+dr, c+dc, i+1) for dr,dc in [(0,1),(0,-1),(1,0),(-1,0)])
        board[r][c] = tmp                       # restore
        return found
    return any(dfs(r,c,0) for r in range(rows) for c in range(cols))

# ⚠️ Pitfall: restore board[r][c] after DFS — in-place marking must be undone
# ⚠️ Pitfall: check boundary AND character match in same condition
```

---

## 13. Key Pitfalls Cheatsheet

| ⚠️ Pitfall | Fix |
|---|---|
| Appending state not copy | `res.append(path[:])` |
| Forgetting to undo choice | always `path.pop()` / restore after recurse |
| Duplicate subsets/perms | sort first + skip `nums[j] == nums[j-1]` |
| Subset dup skip condition `j > 0` | use `j > i` — only skip at current level |
| Perm dup — skip when prev used | skip when `not used[i-1]` |
| Combination sum — not reusing | pass `i` not `i+1` to allow reuse |
| Recursion depth limit | `sys.setrecursionlimit(10**6)` or go iterative |
| `lru_cache` with list param | convert to tuple |
| Mutable default memo dict | use `None` sentinel |
| Word search — not restoring board | save `tmp`, restore after DFS |
| N-Queens — wrong diagonal formula | pos=`row+col`, neg=`row-col` |
