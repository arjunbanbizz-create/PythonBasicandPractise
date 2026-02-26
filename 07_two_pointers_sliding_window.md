# Python Two Pointers & Sliding Window — Interview Snippets & Pitfalls

---

# TWO POINTERS

## 1. Opposite-End Pointers (Sorted Array)
```python
# Two Sum II — sorted input
def two_sum(nums, target):
    l, r = 0, len(nums) - 1
    while l < r:
        s = nums[l] + nums[r]
        if s == target:  return [l, r]
        elif s < target: l += 1
        else:            r -= 1
    return []

# ⚠️ Pitfall: only works on SORTED array
# ⚠️ Pitfall: l < r (not <=) — pointers must not cross
```

---

## 2. Three Sum
```python
def three_sum(nums):
    nums.sort()
    res = []
    for i in range(len(nums) - 2):
        if i > 0 and nums[i] == nums[i-1]: continue    # skip duplicate
        l, r = i+1, len(nums)-1
        while l < r:
            s = nums[i] + nums[l] + nums[r]
            if s == 0:
                res.append([nums[i], nums[l], nums[r]])
                while l < r and nums[l] == nums[l+1]: l += 1  # skip dup
                while l < r and nums[r] == nums[r-1]: r -= 1  # skip dup
                l += 1; r -= 1
            elif s < 0: l += 1
            else:       r -= 1
    return res

# ⚠️ Pitfall: skip duplicates at BOTH outer loop and inner pointers
# ⚠️ Pitfall: after finding triplet, advance BOTH pointers then skip dupes
```

---

## 3. Fast & Slow Pointers
```python
# Already covered in Linked List — quick reference:
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next

# Happy number (cycle detection on numbers)
def is_happy(n):
    def next_n(x):
        return sum(int(d)**2 for d in str(x))
    slow = fast = n
    while True:
        slow = next_n(slow)
        fast = next_n(next_n(fast))
        if fast == 1: return True
        if slow == fast: return False   # cycle → not happy

# ⚠️ Pitfall: fast must move TWO steps — slow one step
```

---

## 4. Partition / Dutch Flag
```python
# Sort 0s, 1s, 2s in-place (Dutch National Flag)
def sort_colors(nums):
    lo = mid = 0
    hi = len(nums) - 1
    while mid <= hi:
        if nums[mid] == 0:
            nums[lo], nums[mid] = nums[mid], nums[lo]
            lo += 1; mid += 1
        elif nums[mid] == 1:
            mid += 1
        else:
            nums[mid], nums[hi] = nums[hi], nums[mid]
            hi -= 1                 # ⚠️ don't advance mid — swapped val unchecked

# ⚠️ Pitfall: when swapping with hi — don't increment mid (new val needs checking)
# ⚠️ Pitfall: when swapping with lo — safe to increment both lo and mid
```

---
---

# SLIDING WINDOW

## 5. Fixed-Size Window
```python
# Max sum subarray of size k
def max_sum_fixed(nums, k):
    window = sum(nums[:k])
    best = window
    for i in range(k, len(nums)):
        window += nums[i] - nums[i-k]   # slide: add right, remove left
        best = max(best, window)
    return best

# ⚠️ Pitfall: subtract nums[i-k] not nums[i-k-1] — correct left element to remove
```

---

## 6. Variable-Size Window — Expand/Shrink
```python
# Longest subarray / substring satisfying condition
def longest_window(nums, condition):
    l = res = 0
    window = ...                        # track window state (sum, count, freq)
    for r in range(len(nums)):
        # expand — add nums[r] to window
        while not condition(window):
            # shrink — remove nums[l] from window
            l += 1
        res = max(res, r - l + 1)
    return res

# ⚠️ Pitfall: window size = r - l + 1 (inclusive both ends)
# ⚠️ Pitfall: update window state BEFORE checking condition
```

---

## 7. Longest Substring Without Repeating Chars
```python
def length_of_longest(s):
    seen = {}
    l = res = 0
    for r, c in enumerate(s):
        if c in seen and seen[c] >= l:
            l = seen[c] + 1             # jump l past last occurrence
        seen[c] = r
        res = max(res, r - l + 1)
    return res

# ⚠️ Pitfall: check seen[c] >= l — char may be outside current window
# ⚠️ Pitfall: jump l directly instead of shrinking one-by-one — O(n) not O(n²)
```

---

## 8. Minimum Window Substring
```python
from collections import Counter

def min_window(s, t):
    need = Counter(t)
    have, required = 0, len(need)
    window = {}
    res, res_len = [-1,-1], float('inf')
    l = 0
    for r, c in enumerate(s):
        window[c] = window.get(c, 0) + 1
        if c in need and window[c] == need[c]:
            have += 1
        while have == required:
            if (r - l + 1) < res_len:
                res, res_len = [l, r], r - l + 1
            window[s[l]] -= 1
            if s[l] in need and window[s[l]] < need[s[l]]:
                have -= 1
            l += 1
    return s[res[0]:res[1]+1] if res_len != float('inf') else ""

# ⚠️ Pitfall: have tracks DISTINCT chars satisfied — not total char count
# ⚠️ Pitfall: update result BEFORE shrinking window
# ⚠️ Pitfall: decrement have only when window count drops BELOW need
```

---

## 9. Longest Repeating Char Replacement
```python
def character_replacement(s, k):
    freq = {}
    l = max_freq = res = 0
    for r, c in enumerate(s):
        freq[c] = freq.get(c, 0) + 1
        max_freq = max(max_freq, freq[c])
        # window size - max_freq = chars to replace
        while (r - l + 1) - max_freq > k:
            freq[s[l]] -= 1
            l += 1
        res = max(res, r - l + 1)
    return res

# ⚠️ Pitfall: max_freq never decreases — ok because we only care about larger windows
# ⚠️ Pitfall: (window_size - max_freq) > k → need to shrink
```

---

## 10. Subarray Sum Equals K (Prefix + HashMap)
```python
def subarray_sum(nums, k):
    freq = {0: 1}               # ✅ seed: empty prefix sums to 0
    prefix = count = 0
    for n in nums:
        prefix += n
        count += freq.get(prefix - k, 0)
        freq[prefix] = freq.get(prefix, 0) + 1
    return count

# ⚠️ Pitfall: seed {0:1} — handles subarrays starting at index 0
# ⚠️ Pitfall: add to freq AFTER checking — avoid using current element twice
```

---

## 11. Sliding Window Maximum (Monotonic Deque)
```python
from collections import deque

def sliding_window_max(nums, k):
    dq = deque()                # indices, decreasing value order
    res = []
    for i, n in enumerate(nums):
        while dq and dq[0] < i - k + 1:    # remove out-of-window
            dq.popleft()
        while dq and nums[dq[-1]] < n:     # maintain decreasing order
            dq.pop()
        dq.append(i)
        if i >= k - 1:
            res.append(nums[dq[0]])
    return res

# ⚠️ Pitfall: store indices not values — need for boundary check
# ⚠️ Pitfall: only output result when i >= k-1 (window is full)
```

---

## 12. When to Use Which Pattern

| Pattern | Signal in Problem |
|---|---|
| Opposite pointers | sorted array, pair/triplet sum |
| Fast & slow | cycle detection, find middle |
| Fixed window | "subarray of size k" |
| Variable window expand/shrink | "longest/shortest subarray with condition" |
| Prefix sum + hashmap | "subarray sum equals k", count subarrays |
| Monotonic deque window | "max/min in sliding window" |

---

## 13. Key Pitfalls Cheatsheet

| ⚠️ Pitfall | Fix |
|---|---|
| Two sum on unsorted array | sort first |
| `l <= r` for two-pointer | use `l < r` — pointers shouldn't cross |
| 3Sum — not skipping duplicates | skip at outer loop AND after finding triplet |
| Dutch flag — advance mid after hi swap | don't advance mid — check swapped value |
| Fixed window off-by-one removal | subtract `nums[i-k]` not `nums[i-k-1]` |
| Window size formula | `r - l + 1` (both inclusive) |
| Minimum window — when to update | update result INSIDE while loop, BEFORE shrinking |
| `have` tracking — wrong decrement | decrement `have` only when count drops below need |
| Subarray sum — missing seed | always seed `{0: 1}` in prefix freq map |
| Sliding window max — value not index | store indices in deque |
