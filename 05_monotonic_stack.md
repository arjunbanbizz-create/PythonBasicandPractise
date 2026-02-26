# Python Monotonic Stack — Interview Snippets & Pitfalls

## 1. Core Idea
```python
# Monotonic Increasing Stack — bottom to top: small → large
# Monotonic Decreasing Stack — bottom to top: large → small

# Template: Next Greater Element
def next_greater(nums):
    res = [-1] * len(nums)
    stack = []                          # stores indices
    for i, n in enumerate(nums):
        while stack and nums[stack[-1]] < n:
            idx = stack.pop()
            res[idx] = n                # n is next greater for idx
        stack.append(i)
    return res

# ⚠️ Pitfall: store INDICES not values — you need position for result array
# ⚠️ Pitfall: remaining items in stack after loop have no greater element → -1 (default)
```

---

## 2. Next Greater / Smaller — All Variants
```python
# Next Greater (strictly greater)
while stack and nums[stack[-1]] < n:   # pop when smaller

# Next Greater or Equal
while stack and nums[stack[-1]] <= n:  # pop when smaller or equal ⚠️ subtle

# Previous Greater
# → iterate left to right, check BEFORE appending
for i, n in enumerate(nums):
    while stack and nums[stack[-1]] <= n:
        stack.pop()
    res[i] = nums[stack[-1]] if stack else -1
    stack.append(i)

# Next Smaller
while stack and nums[stack[-1]] > n:   # pop when greater

# ⚠️ Pitfall: < vs <= changes whether equal elements trigger a pop
# ⚠️ Pitfall: next vs previous — result assigned at pop time (next) or append time (prev)
```

---

## 3. Daily Temperatures
```python
def daily_temperatures(temps):
    res = [0] * len(temps)
    stack = []                          # indices of unresolved days
    for i, t in enumerate(temps):
        while stack and temps[stack[-1]] < t:
            idx = stack.pop()
            res[idx] = i - idx          # days to wait
        stack.append(i)
    return res

# ⚠️ Pitfall: result is distance (i - idx), not the temperature itself
```

---

## 4. Largest Rectangle in Histogram
```python
def largest_rectangle(heights):
    stack = []                          # monotonic increasing stack of indices
    max_area = 0
    heights.append(0)                   # ✅ sentinel to flush remaining stack

    for i, h in enumerate(heights):
        start = i
        while stack and heights[stack[-1]] > h:
            idx = stack.pop()
            width = i - stack[-1] - 1 if stack else i
            max_area = max(max_area, heights[idx] * width)
            start = idx                 # can extend rectangle to here
        stack.append((i, h))            # wait — store original start

    return max_area

# Cleaner version
def largest_rectangle_v2(heights):
    stack, max_area = [], 0
    for i, h in enumerate(heights + [0]):
        start = i
        while stack and stack[-1][1] > h:
            idx, height = stack.pop()
            max_area = max(max_area, height * (i - idx))
            start = idx
        stack.append((start, h))
    return max_area

# ⚠️ Pitfall: append sentinel 0 to heights to flush all remaining stack entries
# ⚠️ Pitfall: width = i - stack[-1] - 1 when stack not empty (wall on left)
# ⚠️ Pitfall: width = i when stack is empty (extends to left boundary)
```

---

## 5. Trapping Rain Water
```python
# Stack approach O(n) time O(n) space
def trap(height):
    stack, water = [], 0
    for i, h in enumerate(height):
        while stack and height[stack[-1]] < h:
            bottom = stack.pop()
            if not stack: break
            left = stack[-1]
            width = i - left - 1
            bounded_h = min(height[left], h) - height[bottom]
            water += width * bounded_h
        stack.append(i)
    return water

# Two-pointer O(n) time O(1) space (preferred)
def trap_two_pointer(height):
    l, r = 0, len(height) - 1
    left_max = right_max = water = 0
    while l < r:
        if height[l] < height[r]:
            if height[l] >= left_max: left_max = height[l]
            else: water += left_max - height[l]
            l += 1
        else:
            if height[r] >= right_max: right_max = height[r]
            else: water += right_max - height[r]
            r -= 1
    return water

# ⚠️ Pitfall: stack approach — must check stack not empty after first pop
```

---

## 6. Remove K Digits (Smallest Number)
```python
def remove_k_digits(num, k):
    stack = []
    for d in num:
        while k and stack and stack[-1] > d:
            stack.pop()
            k -= 1
        stack.append(d)
    # Remove from end if k remaining
    stack = stack[:-k] if k else stack
    return "".join(stack).lstrip('0') or "0"

# ⚠️ Pitfall: after loop, if k > 0 remove last k digits (they are largest)
# ⚠️ Pitfall: lstrip('0') then fallback to "0" for all-zero result
```

---

## 7. Valid Parentheses
```python
def is_valid(s):
    stack = []
    pairs = {')':'(', ']':'[', '}':'{'}
    for c in s:
        if c in '([{':
            stack.append(c)
        elif not stack or stack[-1] != pairs[c]:
            return False
        else:
            stack.pop()
    return not stack

# ⚠️ Pitfall: check `not stack` before stack[-1] to avoid IndexError
# ⚠️ Pitfall: return `not stack` — stack must be empty for valid string
```

---

## 8. Monotonic Stack — When to Use

| Problem Pattern | Stack Type |
|---|---|
| Next Greater Element | Decreasing (pop when greater found) |
| Next Smaller Element | Increasing (pop when smaller found) |
| Largest Rectangle | Increasing (pop when height drops) |
| Trapping Rain Water | Decreasing |
| Remove K Digits | Increasing (keep small digits) |
| Daily Temperatures | Decreasing |

---

## 9. Key Pitfalls Cheatsheet

| ⚠️ Pitfall | Fix |
|---|---|
| Storing values not indices | store indices — needed for width/distance calc |
| Items left in stack after loop | default answer handles them (e.g. `-1`, `0`) |
| `<` vs `<=` for pop condition | `<` for strictly greater next; `<=` includes equal |
| Histogram — no sentinel | append `0` to flush remaining stack |
| Histogram width calc — empty stack | `width = i` (extends to left edge) |
| Parentheses — accessing empty stack | check `not stack` before `stack[-1]` |
| `remove_k_digits` — k remaining | trim last k from stack after loop |
| Rain water stack — no left wall | check `if not stack: break` after pop |
| Assigning result at wrong time | next element → assign at pop; prev → assign before append |
