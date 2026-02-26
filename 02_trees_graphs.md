# Python Trees & Graphs DFS/BFS — Interview Snippets & Pitfalls

## 1. Tree Node Definition
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

---

## 2. Tree Traversals — Recursive
```python
def inorder(root):      # left → root → right (sorted for BST)
    if not root: return []
    return inorder(root.left) + [root.val] + inorder(root.right)

def preorder(root):     # root → left → right
    if not root: return []
    return [root.val] + preorder(root.left) + preorder(root.right)

def postorder(root):    # left → right → root
    if not root: return []
    return postorder(root.left) + postorder(root.right) + [root.val]

# ⚠️ Pitfall: + list concatenation is O(n) per call — use append + helper for O(n) total
def inorder_opt(root, res=[]):      # ⚠️ mutable default arg trap!
    pass
# ✅ always pass res as parameter or use nonlocal
def inorder_opt(root):
    res = []
    def dfs(node):
        if not node: return
        dfs(node.left); res.append(node.val); dfs(node.right)
    dfs(root)
    return res
```

---

## 3. Tree Traversals — Iterative
```python
# Inorder iterative
def inorder_iter(root):
    stack, res = [], []
    cur = root
    while cur or stack:
        while cur:
            stack.append(cur)
            cur = cur.left
        cur = stack.pop()
        res.append(cur.val)
        cur = cur.right
    return res

# Preorder iterative
def preorder_iter(root):
    if not root: return []
    stack, res = [root], []
    while stack:
        node = stack.pop()
        res.append(node.val)
        if node.right: stack.append(node.right)  # right first → left processed first
        if node.left:  stack.append(node.left)
    return res

# ⚠️ Pitfall: push right before left — stack is LIFO so left pops first
```

---

## 4. BFS — Level Order
```python
from collections import deque

def level_order(root):
    if not root: return []
    q, res = deque([root]), []
    while q:
        level = []
        for _ in range(len(q)):         # ✅ snapshot size before processing
            node = q.popleft()
            level.append(node.val)
            if node.left:  q.append(node.left)
            if node.right: q.append(node.right)
        res.append(level)
    return res

# ⚠️ Pitfall: must snapshot len(q) before the inner loop — it grows as you append
# ⚠️ Pitfall: use deque not list — list.pop(0) is O(n)
```

---

## 5. Tree — Common Patterns
```python
# Height / max depth
def max_depth(root):
    if not root: return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))

# Diameter
def diameter(root):
    res = [0]
    def depth(node):
        if not node: return 0
        l, r = depth(node.left), depth(node.right)
        res[0] = max(res[0], l + r)     # path through node
        return 1 + max(l, r)
    depth(root)
    return res[0]

# Lowest Common Ancestor
def lca(root, p, q):
    if not root or root == p or root == q: return root
    left  = lca(root.left, p, q)
    right = lca(root.right, p, q)
    return root if left and right else left or right

# Validate BST
def is_valid_bst(root, lo=float('-inf'), hi=float('inf')):
    if not root: return True
    if not (lo < root.val < hi): return False
    return is_valid_bst(root.left, lo, root.val) and \
           is_valid_bst(root.right, root.val, hi)

# ⚠️ Pitfall: BST validation needs range bounds — not just left < root < right
# ⚠️ Pitfall: diameter path doesn't have to pass through root — use helper + res[]
```

---

## 6. Graph Representations
```python
# Adjacency list — most common in interviews
graph = {
    0: [1, 2],
    1: [0, 3],
    2: [0],
    3: [1]
}

# Build from edge list
from collections import defaultdict
graph = defaultdict(list)
for u, v in edges:
    graph[u].append(v)
    graph[v].append(u)      # undirected — omit for directed

# ⚠️ Pitfall: always clarify directed vs undirected — changes graph construction
```

---

## 7. Graph DFS
```python
# Recursive DFS
def dfs(graph, node, visited=None):
    if visited is None: visited = set()     # ✅ never use mutable default arg
    visited.add(node)
    for nei in graph[node]:
        if nei not in visited:
            dfs(graph, nei, visited)
    return visited

# Iterative DFS
def dfs_iter(graph, start):
    visited = set([start])
    stack = [start]
    while stack:
        node = stack.pop()
        for nei in graph[node]:
            if nei not in visited:
                visited.add(nei)
                stack.append(nei)

# ⚠️ Pitfall: mark visited BEFORE pushing to stack — avoid duplicates in stack
# ⚠️ Pitfall: recursive DFS can hit Python's recursion limit — use iterative for deep graphs
```

---

## 8. Graph BFS
```python
from collections import deque

def bfs(graph, start):
    visited = set([start])
    q = deque([start])
    while q:
        node = q.popleft()
        for nei in graph[node]:
            if nei not in visited:
                visited.add(nei)
                q.append(nei)

# BFS shortest path
def shortest_path(graph, src, dst):
    visited = set([src])
    q = deque([(src, 0)])           # (node, distance)
    while q:
        node, dist = q.popleft()
        if node == dst: return dist
        for nei in graph[node]:
            if nei not in visited:
                visited.add(nei)
                q.append((nei, dist+1))
    return -1

# ⚠️ Pitfall: BFS gives shortest path in UNWEIGHTED graphs only
# ⚠️ Pitfall: add to visited when enqueuing NOT when dequeuing — prevents duplicates
```

---

## 9. Cycle Detection
```python
# Undirected — DFS with parent tracking
def has_cycle_undirected(graph, n):
    visited = set()
    def dfs(node, parent):
        visited.add(node)
        for nei in graph[node]:
            if nei not in visited:
                if dfs(nei, node): return True
            elif nei != parent: return True     # back edge
        return False
    return any(dfs(i, -1) for i in range(n) if i not in visited)

# Directed — DFS with rec_stack (3-color)
def has_cycle_directed(graph, n):
    WHITE, GRAY, BLACK = 0, 1, 2
    color = [WHITE] * n
    def dfs(u):
        color[u] = GRAY
        for v in graph[u]:
            if color[v] == GRAY: return True    # back edge → cycle
            if color[v] == WHITE and dfs(v): return True
        color[u] = BLACK
        return False
    return any(dfs(i) for i in range(n) if color[i] == WHITE)

# ⚠️ Pitfall: undirected cycle detection needs parent — nei != parent check
# ⚠️ Pitfall: GRAY = currently in stack — finding GRAY neighbor = cycle
```

---

## 10. Topological Sort
```python
# Kahn's Algorithm (BFS) — detects cycle if not all nodes processed
from collections import deque, defaultdict

def topo_sort(n, prerequisites):
    graph = defaultdict(list)
    indegree = [0] * n
    for a, b in prerequisites:
        graph[b].append(a)
        indegree[a] += 1
    q = deque([i for i in range(n) if indegree[i] == 0])
    order = []
    while q:
        node = q.popleft()
        order.append(node)
        for nei in graph[node]:
            indegree[nei] -= 1
            if indegree[nei] == 0:
                q.append(nei)
    return order if len(order) == n else []     # [] = cycle detected

# ⚠️ Pitfall: if len(order) != n → cycle exists (impossible ordering)
# ⚠️ Pitfall: edge direction matters — prerequisite b→a means b must come before a
```

---

## 11. Union-Find (Disjoint Set)
```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # path compression
        return self.parent[x]

    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py: return False           # already connected
        if self.rank[px] < self.rank[py]: px, py = py, px
        self.parent[py] = px
        if self.rank[px] == self.rank[py]: self.rank[px] += 1
        return True

    def connected(self, x, y):
        return self.find(x) == self.find(y)

# ⚠️ Pitfall: always use path compression — without it, O(n) per find
# ⚠️ Pitfall: union by rank — prevents tall trees
```

---

## 12. Key Pitfalls Cheatsheet

| ⚠️ Pitfall | Fix |
|---|---|
| Mutable default arg `visited=set()` | use `None` + init inside |
| BFS with `list.pop(0)` | use `deque.popleft()` |
| Add to visited when dequeuing | add when **enqueuing** |
| Recursive DFS on large graph | use iterative to avoid stack overflow |
| BST valid check left/right only | pass `lo`/`hi` bounds recursively |
| Diameter assumes path thru root | use `res[]` with helper tracking max |
| Snapshot `len(q)` after appending | snapshot **before** inner loop |
| Undirected cycle — no parent check | skip `nei == parent` not all visited |
| Directed cycle — wrong state | use 3-color (WHITE/GRAY/BLACK) |
| Topo sort length != n | means cycle — return empty |
| BFS for weighted shortest path | use Dijkstra (heapq) instead |
