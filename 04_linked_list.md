# Python Linked List — Interview Snippets & Pitfalls

## 1. Node Definition
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

# Build from list (helper for testing)
def build(vals):
    dummy = ListNode(0)
    cur = dummy
    for v in vals:
        cur.next = ListNode(v)
        cur = cur.next
    return dummy.next
```

---

## 2. Traversal & Length
```python
def traverse(head):
    cur = head
    while cur:
        print(cur.val)
        cur = cur.next

def length(head):
    count, cur = 0, head
    while cur:
        count += 1
        cur = cur.next
    return count

# ⚠️ Pitfall: always check `cur` not `cur.next` in while condition for traversal
```

---

## 3. Dummy Node Pattern
```python
# Use dummy head to simplify edge cases (empty list, head deletion)
def remove_val(head, val):
    dummy = ListNode(0, head)
    cur = dummy
    while cur.next:
        if cur.next.val == val:
            cur.next = cur.next.next   # skip node
        else:
            cur = cur.next
    return dummy.next                  # ✅ dummy.next handles head removal cleanly

# ⚠️ Pitfall: without dummy, deleting head requires special case logic
# ⚠️ Pitfall: don't advance cur when deleting — next node needs checking too
```

---

## 4. Reverse Linked List
```python
# Iterative
def reverse(head):
    prev, cur = None, head
    while cur:
        nxt = cur.next
        cur.next = prev
        prev = cur
        cur = nxt
    return prev                     # prev is new head

# Recursive
def reverse_rec(head):
    if not head or not head.next: return head
    new_head = reverse_rec(head.next)
    head.next.next = head
    head.next = None
    return new_head

# ⚠️ Pitfall: save nxt before overwriting cur.next — or you lose the list!
# ⚠️ Pitfall: recursive reverse — stack depth O(n), risk of stack overflow on large input
```

---

## 5. Fast & Slow Pointers
```python
# Middle of linked list
def find_middle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow                     # slow is at middle

# ⚠️ Pitfall: condition is fast AND fast.next — prevents null pointer
# For even length: slow lands on SECOND middle — use fast.next.next to get first

# Detect cycle (Floyd's algorithm)
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast: return True
    return False

# Find cycle start
def cycle_start(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            slow = head             # reset slow to head
            while slow != fast:
                slow = slow.next
                fast = fast.next    # both move one step
            return slow             # meeting point = cycle start
    return None

# ⚠️ Pitfall: after detecting cycle, reset only slow — fast stays at meeting point
```

---

## 6. Nth Node From End
```python
def remove_nth_from_end(head, n):
    dummy = ListNode(0, head)
    fast = slow = dummy
    for _ in range(n+1):            # advance fast n+1 steps
        fast = fast.next
    while fast:
        slow = slow.next
        fast = fast.next
    slow.next = slow.next.next      # remove node
    return dummy.next

# ⚠️ Pitfall: advance n+1 (not n) so slow.next is the target node, not slow itself
# ⚠️ Pitfall: start both pointers at dummy — handles removing head node
```

---

## 7. Merge Two Sorted Lists
```python
def merge(l1, l2):
    dummy = ListNode(0)
    cur = dummy
    while l1 and l2:
        if l1.val <= l2.val:
            cur.next = l1; l1 = l1.next
        else:
            cur.next = l2; l2 = l2.next
        cur = cur.next
    cur.next = l1 or l2             # attach remaining
    return dummy.next

# ⚠️ Pitfall: don't forget to attach the remaining non-null list at the end
```

---

## 8. Palindrome Check
```python
def is_palindrome(head):
    # 1. Find middle
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    # 2. Reverse second half
    prev, cur = None, slow
    while cur:
        nxt = cur.next; cur.next = prev; prev = cur; cur = nxt
    # 3. Compare
    l, r = head, prev
    while r:                        # r is shorter or equal
        if l.val != r.val: return False
        l = l.next; r = r.next
    return True

# ⚠️ Pitfall: compare while r (reversed half) — not while l (original may be longer)
```

---

## 9. Intersection of Two Lists
```python
def get_intersection(headA, headB):
    a, b = headA, headB
    while a != b:
        a = a.next if a else headB  # switch to other list on exhaustion
        b = b.next if b else headA
    return a                        # None if no intersection

# ⚠️ Pitfall: compare nodes by REFERENCE (==), not by value
# ⚠️ Pitfall: if no intersection, both become None simultaneously → returns None ✅
```

---

## 10. Reverse in K-Groups
```python
def reverse_k_group(head, k):
    dummy = ListNode(0, head)
    group_prev = dummy
    while True:
        kth = get_kth(group_prev, k)
        if not kth: break
        group_next = kth.next
        prev, cur = kth.next, group_prev.next
        while cur != group_next:
            nxt = cur.next; cur.next = prev; prev = cur; cur = nxt
        tmp = group_prev.next
        group_prev.next = kth
        group_prev = tmp
    return dummy.next

def get_kth(cur, k):
    while cur and k > 0:
        cur = cur.next; k -= 1
    return cur

# ⚠️ Pitfall: if fewer than k nodes remain — don't reverse, just break
```

---

## 11. Key Pitfalls Cheatsheet

| ⚠️ Pitfall | Fix |
|---|---|
| Not saving `nxt` before reversing | `nxt = cur.next` before `cur.next = prev` |
| Deleting head without dummy | always use `dummy = ListNode(0, head)` |
| Not advancing `cur` after skip | only advance when NOT deleting |
| Fast/slow — wrong while condition | `while fast and fast.next` |
| nth from end — advance n not n+1 | advance `n+1` so slow lands before target |
| Forgetting remaining list in merge | `cur.next = l1 or l2` |
| Palindrome compare length mismatch | compare `while r` not `while l` |
| Intersection compared by value | compare by reference `a != b` |
| Recursive reverse — deep recursion | prefer iterative for large lists |
| Cycle start — reset wrong pointer | reset `slow` to head, keep `fast` |
