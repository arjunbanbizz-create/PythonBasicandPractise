# Python Trie — Interview Snippets & Pitfalls

## 1. Trie Node & Class
```python
class TrieNode:
    def __init__(self):
        self.children = {}          # char → TrieNode
        self.is_end = False         # marks end of a word

class Trie:
    def __init__(self):
        self.root = TrieNode()

# ⚠️ Pitfall: use dict not list[26] — dict is simpler and handles any char
# ⚠️ Pitfall: is_end flag is critical — "app" ≠ prefix of "apple" without it
```

---

## 2. Insert
```python
def insert(self, word):
    node = self.root
    for c in word:
        if c not in node.children:
            node.children[c] = TrieNode()
        node = node.children[c]
    node.is_end = True

# O(m) where m = word length
# ⚠️ Pitfall: don't forget to set is_end = True at the last node
```

---

## 3. Search & StartsWith
```python
def search(self, word):             # exact match
    node = self.root
    for c in word:
        if c not in node.children: return False
        node = node.children[c]
    return node.is_end              # ✅ must be end of word

def starts_with(self, prefix):     # prefix match
    node = self.root
    for c in prefix:
        if c not in node.children: return False
        node = node.children[c]
    return True                     # any node reached = valid prefix

# ⚠️ Pitfall: search must check is_end — "app" not found if only "apple" inserted
# ⚠️ Pitfall: starts_with doesn't need is_end check
```

---

## 4. Delete
```python
def delete(self, word):
    def _delete(node, word, i):
        if i == len(word):
            if not node.is_end: return False    # word not found
            node.is_end = False
            return len(node.children) == 0      # safe to delete node?
        c = word[i]
        if c not in node.children: return False
        should_delete = _delete(node.children[c], word, i+1)
        if should_delete:
            del node.children[c]
            return not node.is_end and len(node.children) == 0
        return False
    _delete(self.root, word, 0)

# ⚠️ Pitfall: only delete nodes that aren't shared by other words
```

---

## 5. Word Search in Grid (DFS + Trie)
```python
def find_words(board, words):
    trie = Trie()
    for w in words: trie.insert(w)

    rows, cols = len(board), len(board[0])
    res = set()

    def dfs(node, r, c, path):
        if node.is_end: res.add(path)
        if r < 0 or r >= rows or c < 0 or c >= cols: return
        ch = board[r][c]
        if ch not in node.children or ch == '#': return
        board[r][c] = '#'                           # mark visited
        for dr, dc in [(0,1),(0,-1),(1,0),(-1,0)]:
            dfs(node.children[ch], r+dr, c+dc, path+ch)
        board[r][c] = ch                            # restore

    for r in range(rows):
        for c in range(cols):
            dfs(trie.root, r, c, "")
    return list(res)

# ⚠️ Pitfall: mark cell visited in-place with '#' — restore after backtrack
# ⚠️ Pitfall: use set for res — same word can be found multiple paths
```

---

## 6. Count Words With Prefix
```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False
        self.count = 0              # words passing through this node

def insert(self, word):
    node = self.root
    for c in word:
        if c not in node.children:
            node.children[c] = TrieNode()
        node = node.children[c]
        node.count += 1             # increment at every node
    node.is_end = True

def count_prefix(self, prefix):
    node = self.root
    for c in prefix:
        if c not in node.children: return 0
        node = node.children[c]
    return node.count

# ⚠️ Pitfall: count must be incremented during insert, not just at end node
```

---

## 7. Trie with defaultdict (Compact)
```python
from collections import defaultdict

def make_trie(): return defaultdict(make_trie)

# Usage
trie = make_trie()
def insert(trie, word):
    node = trie
    for c in word:
        node = node[c]
    node['#'] = True                # '#' marks end of word

def search(trie, word):
    node = trie
    for c in word:
        if c not in node: return False
        node = node[c]
    return '#' in node

# ⚠️ Pitfall: elegant but auto-creates keys on access — can bloat memory
# ⚠️ Pitfall: use '#' or sentinel carefully — avoid collision with real chars
```

---

## 8. Complexity Cheatsheet

| Operation | Time | Space |
|---|---|---|
| Insert word | O(m) | O(m) |
| Search word | O(m) | O(1) |
| Prefix search | O(m) | O(1) |
| Delete word | O(m) | O(1) |
| Build trie (n words) | O(n·m) | O(n·m) |

*m = average word length*

---

## 9. Key Pitfalls Cheatsheet

| ⚠️ Pitfall | Fix |
|---|---|
| Forgetting `is_end = True` on insert | always set at last character |
| `search()` without `is_end` check | returns True for prefixes too |
| `{}` vs `defaultdict` | defaultdict auto-inserts — use with care |
| Word search — not restoring board | always restore `board[r][c]` after DFS |
| Word search — duplicate results | use `set` not `list` for results |
| Deleting shared prefix nodes | only delete if no children and not end |
| `count` not updated on insert | increment `count` at every node level |
| `list[26]` for children | use `dict` — simpler and handles Unicode |
