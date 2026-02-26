# Python Binary Search + bisect — Interview Snippets & Pitfalls

## 1. Classic Binary Search Template
```python
def binary_search(nums, target):
    l, r = 0, len(nums) - 1
    while l <= r:
        mid = l + (r - l) // 2      # ✅ avoids overflow vs (l+r)//2
        if nums[mid] == target:   return mid
        elif nums[mid] < target:  l = mid + 1
        else:                     r = mid - 1
    return -1

# ⚠️ Pitfall: l <= r (not l < r) for exact match search
# ⚠️ Pitfall: mid = (l+r)//2 can overflow in other languages — use l+(r-l)//2 as habit
```

---

## 2. Find Leftmost / Rightmost Occurrence
```python
def find_left(nums, target):        # first index of target
    l, r = 0, len(nums) - 1
    res = -1
    while l <= r:
        mid = l + (r - l) // 2
        if nums[mid] == target:   res = mid; r = mid - 1   # keep going left
        elif nums[mid] < target:  l = mid + 1
        else:                     r = mid - 1
    return res

def find_right(nums, target):       # last index of target
    l, r = 0, len(nums) - 1
    res = -1
    while l <= r:
        mid = l + (r - l) // 2
        if nums[mid] == target:   res = mid; l = mid + 1   # keep going right
        elif nums[mid] < target:  l = mid + 1
        else:                     r = mid - 1
    return res

# ⚠️ Pitfall: forgetting to save res before moving pointer
```

---

## 3. bisect Module
```python
import bisect

a = [1, 3, 3, 5, 7]

bisect.bisect_left(a, 3)    # 1 — leftmost index to insert 3
bisect.bisect_right(a, 3)   # 3 — rightmost index to insert 3 (after existing)
bisect.bisect(a, 3)         # same as bisect_right

bisect.insort_left(a, 4)    # insert 4 maintaining sorted order
bisect.insort_right(a, 4)   # insert 4 to the right of existing 4s

# Check existence
idx = bisect.bisect_left(a, target)
found = idx < len(a) and a[idx] == target

# ⚠️ Pitfall: bisect doesn't check if value exists — always verify a[idx] == target
# ⚠️ Pitfall: bisect assumes array is SORTED — wrong results on unsorted input
```

---

## 4. bisect Practical Patterns
```python
# Count elements < target
bisect.bisect_left(a, target)

# Count elements <= target
bisect.bisect_right(a, target)

# Count elements in range [lo, hi]
bisect.bisect_right(a, hi) - bisect.bisect_left(a, lo)

# Find closest value
def closest(a, target):
    idx = bisect.bisect_left(a, target)
    candidates = []
    if idx < len(a): candidates.append(a[idx])
    if idx > 0:      candidates.append(a[idx-1])
    return min(candidates, key=lambda x: abs(x - target))

# ⚠️ Pitfall: always check both neighbors (idx and idx-1) for closest element
```

---

## 5. Search in Rotated Sorted Array
```python
def search_rotated(nums, target):
    l, r = 0, len(nums) - 1
    while l <= r:
        mid = l + (r - l) // 2
        if nums[mid] == target: return mid
        if nums[l] <= nums[mid]:            # left half is sorted
            if nums[l] <= target < nums[mid]: r = mid - 1
            else:                             l = mid + 1
        else:                               # right half is sorted
            if nums[mid] < target <= nums[r]: l = mid + 1
            else:                             r = mid - 1
    return -1

# ⚠️ Pitfall: use nums[l] <= nums[mid] (not <) to handle duplicates edge case
# ⚠️ Pitfall: two conditions needed — which half is sorted, then is target in it
```

---

## 6. Search on Answer (Binary Search on Value Range)
```python
# "Find minimum X such that condition(X) is True"
def search_on_answer(lo, hi):
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if condition(mid):  hi = mid        # mid might be answer, shrink right
        else:               lo = mid + 1    # mid too small, move right
    return lo                               # lo == hi == answer

# Example: Koko eating bananas
def min_eating_speed(piles, h):
    def can_finish(speed):
        return sum(-(-p // speed) for p in piles) <= h  # ceiling division

    lo, hi = 1, max(piles)
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if can_finish(mid): hi = mid
        else:               lo = mid + 1
    return lo

# ⚠️ Pitfall: lo < hi (not <=) for search on answer template
# ⚠️ Pitfall: set hi = mid (not mid-1) when mid could be the answer
# ⚠️ Pitfall: ceiling division: -(-a // b) or math.ceil(a/b)
```

---

## 7. Find Peak Element
```python
def find_peak(nums):
    l, r = 0, len(nums) - 1
    while l < r:
        mid = l + (r - l) // 2
        if nums[mid] > nums[mid+1]: r = mid      # peak is on left side
        else:                       l = mid + 1  # peak is on right side
    return l

# ⚠️ Pitfall: l < r not l <= r — avoids mid+1 out of bounds
```

---

## 8. Key Pitfalls Cheatsheet

| ⚠️ Pitfall | Fix |
|---|---|
| `(l+r)//2` integer overflow | `l + (r-l)//2` |
| `l < r` vs `l <= r` | exact match → `<=`; search on answer → `<` |
| `bisect` on unsorted array | sort first |
| `bisect_left` ≠ element exists | verify `a[idx] == target` |
| `hi = mid-1` in search-on-answer | use `hi = mid` — mid may be the answer |
| Missing closest neighbor check | check both `a[idx]` and `a[idx-1]` |
| Rotated array — wrong half check | verify which half is sorted first |
| Ceiling division | use `-(-a // b)` or `math.ceil(a/b)` |
