# Python itertools & functools — Interview Snippets & Pitfalls

---

# ITERTOOLS

## 1. Combinations & Permutations
```python
from itertools import combinations, permutations, combinations_with_replacement

lst = [1,2,3,4]

list(combinations(lst, 2))               # [(1,2),(1,3),(1,4),(2,3),(2,4),(3,4)]
list(combinations_with_replacement(lst,2))# includes (1,1),(2,2)...
list(permutations(lst, 2))               # all ordered pairs

# Count without generating
from math import comb, perm
comb(4, 2)      # 6  — C(4,2)
perm(4, 2)      # 12 — P(4,2)

# ⚠️ Pitfall: combinations/permutations return TUPLES not lists
# ⚠️ Pitfall: they return iterators — wrap in list() to materialize
# ⚠️ Pitfall: combinations treats position not value — duplicates in input → duplicate tuples
list(combinations([1,1,2], 2))           # [(1,1),(1,2),(1,2)] — not deduplicated!
```

---

## 2. product — Cartesian Product
```python
from itertools import product

list(product([1,2],[3,4]))               # [(1,3),(1,4),(2,3),(2,4)]
list(product('AB', repeat=2))            # [('A','A'),('A','B'),('B','A'),('B','B')]

# Replaces nested for loops
for i, j, k in product(range(n), repeat=3):  # replaces 3 nested loops
    pass

# Generate all binary strings of length n
["".join(t) for t in product("01", repeat=n)]

# ⚠️ Pitfall: product output size = product of all input sizes — can explode
# ⚠️ Pitfall: repeat keyword only, not positional
```

---

## 3. chain — Flatten Iterables
```python
from itertools import chain

list(chain([1,2],[3,4],[5]))             # [1,2,3,4,5]
list(chain.from_iterable([[1,2],[3,4]])) # same — use when input is one iterable of iterables

# ✅ Flatten 2D list O(n) — better than sum([[],[]],[])
matrix = [[1,2],[3,4],[5,6]]
flat = list(chain.from_iterable(matrix))

# ⚠️ Pitfall: chain(*nested) unpacks eagerly — use chain.from_iterable for lazy evaluation
# ⚠️ Pitfall: sum([[1],[2]],[]) is O(n²) — always prefer chain.from_iterable
```

---

## 4. groupby — Group Consecutive Elements
```python
from itertools import groupby

# ⚠️ Pitfall: groupby only groups CONSECUTIVE equal elements — sort first!
data = [1,1,2,2,1,1]
[(k, list(g)) for k,g in groupby(data)]
# [(1,[1,1]),(2,[2,2]),(1,[1,1])]  — NOT three groups!

# ✅ Sort first to group ALL equal elements
data.sort()
[(k, list(g)) for k,g in groupby(data)]

# String compression
s = "aaabbc"
"".join(k + str(sum(1 for _ in g)) for k,g in groupby(s))  # "a3b2c1"

# ⚠️ Pitfall: must consume group g before moving to next key — it's a lazy iterator
for k, g in groupby(data):
    next_key = ...             # ⚠️ don't advance iterator before consuming g
    items = list(g)            # ✅ consume immediately
```

---

## 5. accumulate — Running Totals
```python
from itertools import accumulate
import operator

list(accumulate([1,2,3,4]))                      # [1,3,6,10] — prefix sums
list(accumulate([1,2,3,4], operator.mul))         # [1,2,6,24] — prefix products
list(accumulate([3,1,4,1,5], max))               # [3,3,4,4,5] — running max
list(accumulate([1,2,3,4], initial=0))            # [0,1,3,6,10] ✅ starts with seed

# ⚠️ Pitfall: default is sum — pass func for other operations
# ⚠️ Pitfall: initial param available in Python 3.8+ — output has n+1 elements with it
```

---

## 6. islice, takewhile, dropwhile
```python
from itertools import islice, takewhile, dropwhile

list(islice(range(100), 5))              # [0,1,2,3,4] — first 5
list(islice(range(100), 2, 8))           # [2,3,4,5,6,7] — slice
list(islice(range(100), 0, 100, 3))      # [0,3,6,...] — step

list(takewhile(lambda x: x<5, [1,3,5,2,4]))  # [1,3] — stops at first False
list(dropwhile(lambda x: x<5, [1,3,5,2,4]))  # [5,2,4] — drops until first False

# ⚠️ Pitfall: takewhile/dropwhile stop/start at FIRST condition change — not filter!
# [1,3,5,2,4] takewhile <5 → [1,3] only (stops at 5, ignores 2,4 after)
```

---

## 7. zip_longest, starmap, cycle, repeat
```python
from itertools import zip_longest, starmap, cycle, repeat

# zip_longest — fills missing with fillvalue
list(zip_longest([1,2,3],[4,5], fillvalue=0))   # [(1,4),(2,5),(3,0)]

# starmap — map with argument unpacking
list(starmap(pow, [(2,3),(3,2),(4,2)]))          # [8,9,16]

# cycle — infinite cycling
from itertools import cycle, islice
list(islice(cycle([1,2,3]), 7))                  # [1,2,3,1,2,3,1]

# repeat — repeat element
list(repeat(0, 5))                               # [0,0,0,0,0]

# ⚠️ Pitfall: cycle and repeat are INFINITE — always use with islice or in for loop with break
```

---
---

# FUNCTOOLS

## 8. lru_cache & cache
```python
from functools import lru_cache, cache

@lru_cache(maxsize=128)     # caches last 128 calls
def dp(n): ...

@lru_cache(maxsize=None)    # unlimited cache size
def dp(n): ...

@cache                      # Python 3.9+ — shorthand for lru_cache(maxsize=None)
def dp(n): ...

# Cache info & clearing
dp.cache_info()             # CacheInfo(hits=..., misses=..., maxsize=None, currsize=...)
dp.cache_clear()            # ✅ clear between test cases!

# ⚠️ Pitfall: args must be HASHABLE — convert list to tuple before calling
dp(tuple(nums))

# ⚠️ Pitfall: cache persists across calls — clear between independent test runs
# ⚠️ Pitfall: cache on methods — use on module-level functions, not instance methods
```

---

## 9. reduce
```python
from functools import reduce
import operator

reduce(operator.add, [1,2,3,4])          # 10 — sum
reduce(operator.mul, [1,2,3,4])          # 24 — product
reduce(lambda a,b: a+b, [1,2,3], 0)     # 6  — with initial value
reduce(max, [3,1,4,1,5])                 # 5  — running max

# ⚠️ Pitfall: reduce on empty list without initial value → TypeError
reduce(operator.add, [])                 # ⚠️ TypeError
reduce(operator.add, [], 0)             # ✅ returns 0

# ⚠️ Pitfall: for sum/max/min — prefer built-ins (faster and clearer)
sum([1,2,3])    # ✅ better than reduce
```

---

## 10. partial — Fix Arguments
```python
from functools import partial

def power(base, exp): return base ** exp

square = partial(power, exp=2)
cube   = partial(power, exp=3)

square(4)   # 16
cube(3)     # 27

# Useful for sorting with extra params
import bisect
bisect_right = partial(bisect.bisect_right)

# ⚠️ Pitfall: partial fixes args LEFT to right by default — use keyword args to fix specific params
double = partial(power, exp=2)          # ✅ fixes exp
wrong  = partial(power, 2)             # fixes BASE not exp
```

---

## 11. cmp_to_key
```python
from functools import cmp_to_key

def compare(a, b):
    return -1 if a < b else (1 if a > b else 0)

sorted(a, key=cmp_to_key(compare))

# Largest number
def largest_num_cmp(a, b):
    return -1 if a+b > b+a else (1 if a+b < b+a else 0)

nums = list(map(str, [3,30,34,5,9]))
nums.sort(key=cmp_to_key(largest_num_cmp))

# ⚠️ Pitfall: comparator must return int (neg/0/pos) — not bool
# ⚠️ Pitfall: can't pass comparator directly to sorted() without cmp_to_key
```

---

## 12. Key Pitfalls Cheatsheet

| ⚠️ Pitfall | Fix |
|---|---|
| `combinations` on list with dupes | pre-deduplicate input if unique combos needed |
| `product` output size explosion | check sizes before generating |
| `groupby` without sorting | sort first — groups only consecutive runs |
| Consuming `groupby` group lazily | `list(g)` immediately inside loop |
| `takewhile` vs filter | takewhile stops at first False — not a filter |
| `cycle`/`repeat` are infinite | always pair with `islice` or break |
| `reduce` on empty iterable | always provide initial value |
| `lru_cache` with list args | convert to `tuple` |
| `cache_clear()` between tests | call `fn.cache_clear()` explicitly |
| `partial` fixes left args by default | use keyword args to fix specific params |
| `cmp_to_key` comparator returns bool | return `-1`, `0`, `1` not `True`/`False` |
| `chain(*nested)` eager unpack | use `chain.from_iterable(nested)` |
