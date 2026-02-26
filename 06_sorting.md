# Python Sorting + cmp_to_key — Interview Snippets & Pitfalls

## 1. Built-in Sort
```python
a.sort()                            # in-place, returns None
sorted(a)                           # returns new list
sorted(a, reverse=True)             # descending
a.sort(key=lambda x: x[1])         # sort by second element
sorted(a, key=len)                  # sort strings by length

# Multi-key sort (tuple key)
sorted(a, key=lambda x: (x[1], -x[0]))  # asc by [1], desc by [0]

# ⚠️ Pitfall: a.sort() returns None — never do a = a.sort()
# ⚠️ Pitfall: sort is STABLE in Python — equal elements preserve original order
```

---

## 2. cmp_to_key — Custom Comparator
```python
from functools import cmp_to_key

# comparator(a, b):
#   return negative → a comes first
#   return positive → b comes first
#   return 0        → equal

def compare(a, b):
    if a < b: return -1
    if a > b: return  1
    return 0

sorted(a, key=cmp_to_key(compare))

# ⚠️ Pitfall: can't use comparator directly in sorted() — must wrap with cmp_to_key
# ⚠️ Pitfall: return -1/0/1 pattern, not True/False
```

---

## 3. Largest Number (Classic cmp_to_key Problem)
```python
def largest_number(nums):
    from functools import cmp_to_key
    def compare(a, b):
        if a+b > b+a: return -1    # a should come first
        if a+b < b+a: return  1
        return 0
    nums = list(map(str, nums))
    nums.sort(key=cmp_to_key(compare))
    result = "".join(nums)
    return "0" if result[0] == "0" else result

# ⚠️ Pitfall: compare string concatenations, not numeric values
# ⚠️ Pitfall: all zeros edge case → return "0" not "000"
```

---

## 4. Sort by Multiple Criteria
```python
# Primary: length ascending, Secondary: alphabetical descending
sorted(words, key=lambda w: (len(w), [-ord(c) for c in w]))

# Negate numeric fields for descending
sorted(people, key=lambda p: (-p.age, p.name))  # age desc, name asc

# Sort with None / missing values last
sorted(a, key=lambda x: (x is None, x))         # None goes last

# ⚠️ Pitfall: negating works for numbers only — for strings use cmp_to_key
# ⚠️ Pitfall: (x is None, x) → False/True sorts non-None first
```

---

## 5. Interval Sorting
```python
intervals = [[1,3],[2,6],[8,10],[15,18]]

# Sort by start time
intervals.sort(key=lambda x: x[0])

# Merge overlapping intervals
def merge_intervals(intervals):
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0]]
    for start, end in intervals[1:]:
        if start <= merged[-1][1]:          # overlap
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])
    return merged

# ⚠️ Pitfall: sort BEFORE merging — overlapping check only works on sorted input
# ⚠️ Pitfall: use max(merged[-1][1], end) — new interval may be fully contained
```

---

## 6. Partial Sort — heapq.nlargest / nsmallest
```python
import heapq

heapq.nlargest(k, nums)             # top k largest — O(n log k)
heapq.nsmallest(k, nums)           # top k smallest — O(n log k)
heapq.nlargest(k, dicts, key=lambda x: x['score'])

# When to use:
# k == 1    → max() / min()         O(n)
# k << n    → heapq.nlargest        O(n log k)
# k ~ n     → sorted()              O(n log n)

# ⚠️ Pitfall: nlargest/nsmallest return lists — not heaps
```

---

## 7. Counting Sort (O(n) for bounded integers)
```python
def counting_sort(nums, max_val):
    count = [0] * (max_val + 1)
    for n in nums:
        count[n] += 1
    return [n for n, c in enumerate(count) for _ in range(c)]

# ⚠️ Pitfall: only works for non-negative integer keys
# ⚠️ Pitfall: large max_val → huge count array — check range first
```

---

## 8. Comparator Patterns

```python
# Sort strings case-insensitively
sorted(words, key=str.lower)
sorted(words, key=lambda w: w.casefold())   # ✅ better for unicode

# Sort by frequency then value
from collections import Counter
freq = Counter(nums)
sorted(nums, key=lambda x: (-freq[x], x))

# Topological-style custom order
order = {c: i for i, c in enumerate("dcba")}
sorted(s, key=lambda c: order.get(c, len(order)))

# ⚠️ Pitfall: casefold() > lower() for unicode case-insensitive compare
```

---

## 9. Key Pitfalls Cheatsheet

| ⚠️ Pitfall | Fix |
|---|---|
| `a = a.sort()` | `a.sort()` in-place OR `a = sorted(a)` |
| Comparator directly in `sorted()` | wrap with `cmp_to_key` |
| Negating string for descending | use `cmp_to_key` instead |
| Largest number — numeric compare | compare string concatenations |
| Merge intervals without sorting | sort by start time first |
| Merge intervals — contained check | `max(merged[-1][1], end)` not just `end` |
| `nlargest` when k == 1 | use `max()` — faster |
| Counting sort with negatives | shift values: `n - min_val` |
| `lower()` for unicode comparison | use `casefold()` |
| Unstable sort assumption | Python sort IS stable — safe to rely on |
