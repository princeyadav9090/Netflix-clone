<think>The user asks: "give me approach of leetcode 987". Likely they refer to LeetCode problem #987: "Vertical Order Traversal of a Binary Tree". The user wants approach. Provide explanation of problem, algorithm, data structures, steps, complexity, maybe code snippet, and tips.

We should ensure we are not violating policies: not providing copyrighted solution; but providing explanation of algorithm is permissible. The user just wants the approach. So respond with high-level approach: BFS traversal with mapping by column and then order by row and value ordering. Use map of col -> list of (row, value). Then sort by column, then by row, then value. Complexity O(N log N). Or alternative with priority queue for order.

We can give pseudocode and optionally an implementation in Python.

We'll also mention dealing with duplicate nodes same column and row.

We can also talk about DFS vs BFS.

Thus respond with description.

Make sure not to mention policy.

Will give approach.
</think>

**LeetCode 987 – Vertical Order Traversal of a Binary Tree**  

---

### 1.  What the problem asks  

For every column of the binary tree (left‑most column = column 0), output the nodes that belong to that column **top‑to‑bottom** and, for nodes that lie on the same row, order them **by value**.  
The result is a list of lists where `result[i]` contains the values of the nodes in column i (from leftmost to rightmost column).

---

### 2.  Core idea – a 2‑step process  

1. **Collect (column, row, value) for every node** while traversing the tree.  
2. **Group** the collected points by column and **order** them:

   * first by **row** (smaller row → higher on the screen)  
   * then by **value** (smaller value first)

---

### 3.  Data structures  

| Structure | Why we use it |
|-----------|---------------|
| `deque` for BFS (or recursion stack) | We need a level‑order traversal so that we naturally get the correct row numbers (`row = parent.row + 1`). |
| `defaultdict(list)` mapping `col → list of (row, val)` | Keeps all nodes that belong to the same column together. |
| `sorted` (or a priority queue) | To order the entries inside each column by `(row, val)`. Because the total number of nodes is at most `N`, sorting each column separately (or sorting the whole list once) is fast enough. |

---

### 4.  Detailed algorithm  

```
verticalTraversal(root):
    if root is None: return []

    col_map = defaultdict(list)      # col -> [(row, val), ...]
    queue   = deque([(root, col=0, row=0)])

    while queue:
        node, col, row = queue.popleft()
        col_map[col].append((row, node.val))

        if node.left:  queue.append((node.left,  col-1, row+1))
        if node.right: queue.append((node.right, col+1, row+1))

    # At this point col_map holds all nodes.
    # We need to output columns from the smallest key to the largest.
    result = []
    for col in sorted(col_map.keys()):
        # sort the list of (row, val) inside this column
        col_map[col].sort()          # Python sorts tuples lexicographically
        # extract only the values, preserving the order
        result.append([val for _, val in col_map[col]])

    return result
```

**Key points**

* The BFS (level‑order) guarantees that when we first see a node, its `row` is the smallest possible (the tree is traversed top‑down).
* For nodes that end up in the same column **and** the same row, we need the extra ordering by value – the tuple `(row, val)` handles that automatically because Python sorts by the first element, then by the second.

---

### 5.  Complexity analysis  

* **Time**  
  * Traversing the tree: `O(N)` where `N` = number of nodes.  
  * Inserting into `col_map` is `O(1)` amortised.  
  * Sorting the values inside each column: the total number of items across all columns is `N`, so the total sorting work is `O(N log N)`.  
  * Overall: **`O(N log N)`**.

* **Space**  
  * The `col_map` stores `N` entries → **`O(N)`** extra space.  
  * The BFS queue holds at most `O(width_of_tree) ≤ O(N)` nodes in the worst case.

---

### 6.  Alternative (priority‑queue based)  

If you prefer to avoid the final sorting step, you can push `(col, row, val)` into a **min‑heap** (priority queue) while doing the BFS.  
The heap keeps the correct order at all times, but the heap operations cost `O(log N)` each, leading to the same `O(N log N)` total time and slightly higher constant factors.  

The simple two‑step map‑then‑sort approach is usually easier to implement and is fast enough for the constraints.

---

### 7.  Common pitfalls  

| Pitfall | Why it hurts | How to avoid |
|--------|--------------|--------------|
| Using DFS without tracking row properly | DFS visits left subtree completely before right, so the first time you see a node its row might not be the smallest. | Use BFS or store the smallest row you ever encounter for each `(col, val)` pair (extra bookkeeping). |
| Forgetting nodes with the same column & row | If two nodes share column and row, they must be ordered by value, not just by traversal order. | Use `(row, val)` tuples; simple BFS order may break tie incorrectly. |
| Not handling negative columns | The root can have children to the left → column `-1`, `-2`, … | The `defaultdict` and `sorted(col_map.keys())` automatically handle any integer keys. |

---

### 8.  Reference implementation (Python 3)

```python
from collections import defaultdict, deque
from typing import Optional, List

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def verticalTraversal(root: Optional[TreeNode]) -> List[List[int]]:
    if not root:
        return []

    col_map = defaultdict(list)            # col -> [(row, val), ...]
    q = deque()
    q.append((root, col=0, row=0))

    while q:
        node, col, row = q.popleft()
        col_map[col].append((row, node.val))

        if node.left:
            q.append((node.left, col-1, row+1))
        if node.right:
            q.append((node.right, col+1, row+1))

    # Build the answer column by column (sorted by column index)
    ans = []
    for col in sorted(col_map.keys()):
        col_map[col].sort()                 # sort by (row, val)
        ans.append([val for _, val in col_map[col]])

    return ans
```

---

### 9.  Quick cheat‑sheet summary  

| Step | Action | Data structure |
|------|--------|----------------|
| 1️⃣   | BFS the tree, record `(col, row, node.val)` | `deque` + `defaultdict(list)` |
| 2️⃣   | For each column, sort its list by `(row, val)` | `list.sort()` |
| 3️⃣   | Output column by column (left → right) | `sorted(col_map.keys())` |

That’s the classic, clean, and optimal approach for LeetCode 987. Happy coding!
