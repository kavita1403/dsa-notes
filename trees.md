# Tree Patterns — Master Reference Note

```java
// Standard node definition assumed throughout
class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

---

## 🧭 Decision Fork (ask yourself this before writing any code)

> Does the answer for a node need info **from its children**? → **Bottom-Up** (post-order). Recurse first, combine on the way back up.
>
> Does the answer need info **from its ancestors**? → **Top-Down** (pre-order). Pass state DOWN as you recurse, decide at the leaves.
>
> Do you need the answer **level-by-level**? → **BFS**. Queue, snapshot size before the inner loop.
>
> Everything else in this topic is a variant or combination of these three questions.

---

## Quick Lookup Table (read this first when revisiting after a gap)

| # | Pattern | 🔑 Trigger phrases | ⚓ Anchor problem | Core idea |
|---|---|---|---|---|
| 1 | DFS Bottom-Up | "height", "count", "is balanced", "diameter" | Maximum Depth of Binary Tree | Recurse first, combine children's results into your own |
| 2 | DFS Top-Down | "root-to-leaf", "path sum" | Path Sum | Pass state DOWN as you recurse, decide at leaves |
| 3 | BFS / Level Order | "level by level", "shortest path (unweighted)", "right/left side view" | Binary Tree Level Order Traversal | Queue, snapshot size before inner loop |
| 4 | LCA | "lowest common ancestor", "split point of two nodes" | Lowest Common Ancestor of a Binary Tree | Bottom-up: both sides found something → I'm the answer |
| 5 | Construction / Serialization | "build tree from X and Y", "serialize/deserialize" | Construct Binary Tree from Preorder and Inorder Traversal | Use traversal order properties to reconstruct uniquely |
| 6 | Tree DP (multi-state) | "rob houses", "max path sum with constraint" | House Robber III | Return an array/tuple of states per node, not one value |
| 7 | BST-specific | problem says "BST" / sorted property usable | Validate Binary Search Tree | Use value comparisons to skip half the tree, or bounds-passing |
| 8 (bonus) | Trie | "prefix", "autocomplete", "word search dictionary" | Implement Trie (Prefix Tree) | Tree keyed by characters, not fixed left/right |

---

## 1. DFS — Bottom-Up (Post-order: "get info from children, then build your own")

**When to use it:** Whenever a node's answer is computed *from* its subtrees' answers — height, count of nodes, diameter, balanced-check, subtree sum, identical-trees check.

**🔑 Trigger phrases:** "height", "depth", "count nodes", "is balanced", "diameter", "is same tree", "is subtree of".

**⚓ Anchor problem:** Maximum Depth of Binary Tree — if you forget everything else about this pattern, re-derive it from this one.

**Template:**
```java
ReturnType dfs(TreeNode node) {
    if (node == null) return /* base case value */;

    ReturnType left = dfs(node.left);
    ReturnType right = dfs(node.right);

    return combine(left, right, node.val); // build this node's answer
}
```

**Why this shape:** You literally cannot compute a node's answer before you know what its children produced — that's the definition of "depends on subtrees." Post-order (left, right, then process current) is the only traversal order that guarantees children are fully resolved before the parent needs them.

**Edge cases:**
- Empty tree (`root == null`) — decide your base case value carefully: `0` for height/count, `true` for "isBalanced", `Integer.MIN_VALUE` for max-path type problems.
- Height vs. count convention: height of a null node is often `-1` (so a single node has height 0), while node-count of null is `0`. Mixing these up is a classic off-by-one.

**Common pitfalls:**
- Forgetting the null check first → NullPointerException.
- Confusing "value returned to parent" with "final answer to the problem." Sometimes you need a separate instance variable to track a global answer (e.g., Diameter of Binary Tree: you return *height* up the recursion, but track the *diameter* as a side variable updated at every node).
- Not handling the leaf case distinctly when the combine logic assumes both children exist.

**📝 My Mistakes (fill in as you go):**
- _Add your own here when you debug something for more than 10 minutes — write down why you got it wrong, not just the fix. This is the section generic notes can't write for you._

**Example problems:** Maximum Depth of Binary Tree, Balanced Binary Tree, Diameter of Binary Tree, Same Tree, Subtree of Another Tree, Count Complete Tree Nodes.

---

## 2. DFS — Top-Down (Pre-order: "pass info down from parent to children")

**When to use it:** When the answer/condition depends on the path *from the root down* to the current node — root-to-leaf sums, validating constraints that tighten as you go deeper, tracking depth.

**🔑 Trigger phrases:** "root-to-leaf", "path sum", "sum of all paths", "path from root".

**⚓ Anchor problem:** Path Sum — if you forget everything else, rebuild from this.

**Template:**
```java
void dfs(TreeNode node, StateFromParent state) {
    if (node == null) return;

    State newState = update(state, node.val);

    if (node.left == null && node.right == null) {
        // leaf reached — finalize/record something here if needed
    }

    dfs(node.left, newState);
    dfs(node.right, newState);
}
```

**Why this shape:** Each node needs to know something accumulated from its ancestors (running sum, depth, min/max bounds) before it can decide anything about itself or pass it further down. There's nothing to "wait for" from children — the information flows the opposite direction from Pattern 1.

**Edge cases:**
- Leaf detection (`left == null && right == null`) is often essential to know when a "path" has ended — don't confuse a leaf with a node that just has one null child.
- Deciding whether to recurse into null children at all, or stop earlier.

**Common pitfalls:**
- Mutating a shared, single state object instead of creating an updated copy per call — both recursive branches (left/right) need their *own* independent version of the state, not a shared mutable reference.
- Forgetting the leaf check and accidentally treating internal nodes as valid path endpoints.

**Worked example of the mutation pitfall — Binary Tree Paths (collect every root-to-leaf path):**

Tree:
```
    1
   / \
  2   3
```
Expected output: `["1->2", "1->3"]`

**Buggy version — one shared `path` list, no cleanup before returning:**
```java
List<String> result = new ArrayList<>();
List<Integer> path = new ArrayList<>(); // ONE list shared across the whole traversal

void dfs(TreeNode node) {
    if (node == null) return;
    path.add(node.val);                 // mutates the SAME list every call

    if (node.left == null && node.right == null) {
        result.add(pathToString(path));
    }

    dfs(node.left);
    dfs(node.right);
    // missing: nothing undoes the path.add(node.val) above before returning
}
```
Trace it by hand:
- `dfs(1)`: `path = [1]`
- `dfs(2)`: `path = [1, 2]` → leaf → records `"1->2"` ✓
- back in `dfs(1)`, now calls `dfs(3)`: `path` is **still** `[1, 2]` from the left branch (nothing removed it) → `path.add(3)` → `path = [1, 2, 3]` → leaf → records `"1->2->3"` ✗

Actual output: `["1->2", "1->2->3"]` — wrong, and silently wrong, which is what makes this bug nasty to spot.

**Fix, option A — backtrack (undo the mutation before returning to the parent):**
```java
void dfs(TreeNode node) {
    if (node == null) return;
    path.add(node.val);

    if (node.left == null && node.right == null) {
        result.add(pathToString(path));
    }

    dfs(node.left);
    dfs(node.right);

    path.remove(path.size() - 1); // undo, so the parent's list is clean for its next branch
}
```

**Fix, option B — never share state at all; pass a fresh copy down each call:**
```java
void dfs(TreeNode node, List<Integer> path) {
    if (node == null) return;

    List<Integer> newPath = new ArrayList<>(path); // independent copy for this branch
    newPath.add(node.val);

    if (node.left == null && node.right == null) {
        result.add(pathToString(newPath));
    }

    dfs(node.left, newPath);
    dfs(node.right, newPath);
}
```

Dry run, same tree, starting call `dfs(1, [])`:
- `dfs(1, [])`: builds `newPath = [1]` (its own copy, scoped to this call). Not a leaf. Calls `dfs(2, [1])`, then `dfs(3, [1])` — note both children are handed the **same `[1]`**, taken from this frame's `newPath`, regardless of what either child does with it.
- `dfs(2, [1])`: builds its own `newPath = [1, 2]` (a brand-new copy — `[1]` from the parent is untouched). Leaf → records `"1->2"`.
- Control returns to `dfs(1, ...)`. Its `newPath` is still exactly `[1]` — the call to `dfs(2, ...)` only ever mutated *its own* copy, never the parent's. So nothing needs to be undone here, unlike option A.
- `dfs(3, [1])`: builds its own `newPath = [1, 3]` from that same untouched `[1]`. Leaf → records `"1->3"`.

Final result: `["1->2", "1->3"]` ✓ — correct, and notice there's no `path.remove(...)` anywhere, because each recursive call owns a private copy. That's the whole trade: option A reuses one list and must clean up after itself; option B never shares a list, so there's nothing to clean up, at the cost of an `O(depth)` copy on every single call.

**📝 My Mistakes (fill in as you go):**
- _Add your own here — e.g. the exact moment you caught (or didn't catch) a shared-state bug like the one above, in your own words._

**Example problems:** Path Sum, Path Sum II (collect actual paths), Sum Root to Leaf Numbers, Validate BST (bounds-passing variant — see Pattern 7).

---

## 3. BFS / Level Order Traversal

**When to use it:** "Level by level" processing, shortest path in an unweighted tree, right/left side view, zigzag order, level averages.

**🔑 Trigger phrases:** "level by level", "level order", "right side view", "zigzag", "average of each level", "minimum depth" (unweighted shortest path).

**⚓ Anchor problem:** Binary Tree Level Order Traversal.

**Template:**
```java
List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;

    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int size = queue.size();          // <-- THE key line
        List<Integer> level = new ArrayList<>();

        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
    }
    return result;
}
```

**Why this shape:** A queue is FIFO, so nodes come out in the exact order they were discovered — which is naturally level-by-level for a tree. The line `int size = queue.size()` taken **before** the inner loop freezes "how many nodes belong to the current level," so the children you enqueue inside the loop don't get mistakenly processed as part of the same level.

**Edge cases:**
- Empty tree — check before creating the queue.
- Single-node tree — should produce one level with one element; trace through manually if unsure.

**Common pitfalls:**
- **The #1 BFS bug:** not snapshotting `size` before the loop, or calling `queue.size()` inside the loop condition directly — this silently corrupts level boundaries as children get added mid-loop.
- Off-by-one when also tracking depth/level number alongside values.

**📝 My Mistakes (fill in as you go):**
- _Add your own here._

**Example problems:** Binary Tree Level Order Traversal, Binary Tree Right Side View, Average of Levels, Zigzag Level Order Traversal, Minimum Depth of Binary Tree (BFS gives early-exit advantage over DFS here).

---

## 4. Lowest Common Ancestor (LCA)

**When to use it:** "Find the lowest/deepest node that is an ancestor of both p and q."

**🔑 Trigger phrases:** "lowest common ancestor", "split point of two nodes", "deepest node that is an ancestor of both".

**⚓ Anchor problem:** Lowest Common Ancestor of a Binary Tree.

**General binary tree template:**
```java
TreeNode lca(TreeNode node, TreeNode p, TreeNode q) {
    if (node == null || node == p || node == q) return node;

    TreeNode left = lca(node.left, p, q);
    TreeNode right = lca(node.right, p, q);

    if (left != null && right != null) return node; // p, q found in different subtrees
    return left != null ? left : right;               // only one side found something — bubble it up
}
```

**BST-specific optimization** (skip full traversal, use value comparisons):
```java
TreeNode lcaBST(TreeNode node, TreeNode p, TreeNode q) {
    while (node != null) {
        if (p.val < node.val && q.val < node.val) node = node.left;
        else if (p.val > node.val && q.val > node.val) node = node.right;
        else return node; // this is the split point
    }
    return null;
}
```

**Why this shape:** Bottom-up, each call answers "did I find p or q anywhere below me?" If both subtrees report a find, the current node is exactly the split point — the LCA. If only one side finds something, that finding has to propagate further up, since the true LCA must sit higher in the tree.

**Edge cases:**
- p or q being the root itself.
- One of p/q being a direct ancestor of the other (still handled correctly by the same logic — trace through manually to convince yourself).
- Some problem variants guarantee both p and q exist in the tree; check constraints before assuming this.

**Common pitfalls:**
- Forgetting the base case `node == p || node == q` (not just `node == null`) — this is what lets a node "find itself" without needing to look further down.
- Using the general recursive version when the BST iterative version is available — unnecessarily slower and uses extra stack space.

**📝 My Mistakes (fill in as you go):**
- _Add your own here._

**Example problems:** Lowest Common Ancestor of a Binary Tree, Lowest Common Ancestor of a BST.

---

## 5. Tree Construction / Serialization

**When to use it:** Building a tree from traversal arrays, or serializing/deserializing a tree to/from a string.

**🔑 Trigger phrases:** "build tree from preorder/inorder", "serialize", "deserialize", "reconstruct the tree".

**⚓ Anchor problem:** Construct Binary Tree from Preorder and Inorder Traversal.

**Serialize/deserialize (preorder + null markers):**
```java
void serialize(TreeNode node, StringBuilder sb) {
    if (node == null) { sb.append("null,"); return; }
    sb.append(node.val).append(",");
    serialize(node.left, sb);
    serialize(node.right, sb);
}

int idx = 0; // shared mutable index across recursive calls
TreeNode deserialize(String[] values) {
    String val = values[idx++];
    if (val.equals("null")) return null;
    TreeNode node = new TreeNode(Integer.parseInt(val));
    node.left = deserialize(values);
    node.right = deserialize(values);
    return node;
}
```

**Build from preorder + inorder:**
```java
Map<Integer, Integer> inorderIndex = new HashMap<>();
int preIdx = 0;

TreeNode build(int[] preorder, int[] inorder, int left, int right) {
    if (left > right) return null;

    int rootVal = preorder[preIdx++];
    TreeNode root = new TreeNode(rootVal);
    int mid = inorderIndex.get(rootVal);

    root.left = build(preorder, inorder, left, mid - 1);
    root.right = build(preorder, inorder, mid + 1, right);
    return root;
}
```

**Why this shape:** Preorder visits a node *before* its children, so reading values back in the same order you wrote them reconstructs the exact shape — null markers tell you exactly where each subtree ends without needing separate size bookkeeping. For preorder+inorder construction: preorder always gives you the root first; inorder tells you which values belong to the left subtree (everything before the root's position) vs. right subtree (everything after).

**Edge cases:**
- Duplicate values break the inorder-index-lookup approach — confirm the problem guarantees unique values, or switch to tracking by index/range instead of value.

**Common pitfalls:**
- `preIdx` (or `idx` in deserialize) must be a shared mutable field, not a value re-passed by parameter — it needs to keep advancing across recursive branches in a specific global order.
- Off-by-one errors in the `left`/`right`/`mid` bounds when splitting the inorder range.

**📝 My Mistakes (fill in as you go):**
- _Add your own here._

**Example problems:** Construct Binary Tree from Preorder and Inorder Traversal, Serialize and Deserialize Binary Tree.

---

## 6. Tree DP — Multi-State Return (need more than one value from children)

**When to use it:** When a single number per node isn't enough — you need to track multiple scenarios and let the parent pick the best combination (classic dynamic programming, just tree-shaped).

**🔑 Trigger phrases:** "rob houses" (tree-shaped), "max path sum with a constraint", "best score with adjacent restriction".

**⚓ Anchor problem:** House Robber III.

**Template (House Robber III style — "rob this node" vs "don't rob"):**
```java
int[] dfs(TreeNode node) {
    if (node == null) return new int[]{0, 0}; // {rob this node, don't rob this node}

    int[] left = dfs(node.left);
    int[] right = dfs(node.right);

    int robThis    = node.val + left[1] + right[1];                     // children can't be robbed
    int notRobThis = Math.max(left[0], left[1]) + Math.max(right[0], right[1]); // children free to pick their best

    return new int[]{robThis, notRobThis};
}
```

**Why this shape:** Some problems genuinely need multiple "modes" tracked simultaneously (rob/not-rob, path-open/path-closed) because collapsing to one number too early loses information the parent needs to make its own optimal choice.

**Edge cases:**
- Null nodes must return the correct "zero" baseline for *every* state slot, not just one.

**Common pitfalls:**
- Collapsing two states into one number too early in the recursion — you can't un-lose that information later.
- Forgetting that "don't rob this node" must let each child independently choose its own best option (not force left and right to agree on rob/not-rob).

**📝 My Mistakes (fill in as you go):**
- _Add your own here._

**Example problems:** House Robber III, Binary Tree Maximum Path Sum (bottom-up return of "best single-direction path through this node" + a side global-max variable, similar to Diameter in Pattern 1).

---

## 7. Binary Search Tree (BST)-Specific Patterns

**When to use it:** Problem explicitly involves a BST, or you can exploit the sorted-order property (inorder traversal of a BST = sorted sequence).

**🔑 Trigger phrases:** "BST", "binary search tree", "kth smallest", problem mentions values are sorted/ordered.

**⚓ Anchor problem:** Validate Binary Search Tree.

**Validate BST (bounds-passing, top-down variant):**
```java
boolean isValidBST(TreeNode node, long lower, long upper) {
    if (node == null) return true;
    if (node.val <= lower || node.val >= upper) return false;
    return isValidBST(node.left, lower, node.val)
        && isValidBST(node.right, node.val, upper);
}
// initial call: isValidBST(root, Long.MIN_VALUE, Long.MAX_VALUE)
```

**Insert (iterative or recursive, O(log n) average):**
```java
TreeNode insert(TreeNode root, int val) {
    if (root == null) return new TreeNode(val);
    if (val < root.val) root.left = insert(root.left, val);
    else root.right = insert(root.right, val);
    return root;
}
```

**Why this shape:** Each node constrains the legal value range for everything beneath it — the left subtree must stay strictly below the node's value, the right subtree strictly above, and these bounds tighten with every level down. This lets you skip exploring whole subtrees during search/insert, since you always know which side a value must be on.

**Edge cases:**
- Duplicate values — confirm whether the problem allows them (most assume distinct values).
- Using `int` for bounds can overflow if node values can be `Integer.MIN_VALUE`/`MAX_VALUE` — that's exactly why the validate-BST template above uses `long` bounds.

**Common pitfalls:**
- **The #1 BST-validation bug:** only checking a node against its *immediate* children (`left.val < node.val < right.val`) instead of propagating bounds further down — this misses violations from grandchildren two or more levels deep.

**📝 My Mistakes (fill in as you go):**
- _Add your own here._

**Example problems:** Validate Binary Search Tree, Insert into a BST, Delete Node in a BST, Kth Smallest Element in a BST (inorder traversal + counter).

---

## 8 (bonus). Trie / Prefix Tree

Different shape from the rest — each node has a map of children keyed by *character*, not fixed left/right — but commonly grouped with tree problems.

**🔑 Trigger phrases:** "prefix", "autocomplete", "dictionary of words", "word search II".

**⚓ Anchor problem:** Implement Trie (Prefix Tree).

```java
class TrieNode {
    Map<Character, TrieNode> children = new HashMap<>();
    boolean isWord = false;
}

class Trie {
    TrieNode root = new TrieNode();

    void insert(String word) {
        TrieNode cur = root;
        for (char c : word.toCharArray()) {
            cur = cur.children.computeIfAbsent(c, k -> new TrieNode());
        }
        cur.isWord = true;
    }

    boolean search(String word) {
        TrieNode node = find(word);
        return node != null && node.isWord;
    }

    boolean startsWith(String prefix) {
        return find(prefix) != null;
    }

    private TrieNode find(String s) {
        TrieNode cur = root;
        for (char c : s.toCharArray()) {
            cur = cur.children.get(c);
            if (cur == null) return null;
        }
        return cur;
    }
}
```

**Why this shape:** Each root-to-node path represents a prefix; branching by character lets many words share common prefixes in the structure, making prefix lookups proportional to word length rather than the number of stored words.

**Common pitfalls:**
- Forgetting to mark `isWord = true` on insert — `search("cat")` can incorrectly return false if "cat" is also a prefix of another inserted word like "catalog," because the node exists but isn't flagged as a complete word.

**📝 My Mistakes (fill in as you go):**
- _Add your own here._

**Example problems:** Implement Trie (Prefix Tree), Word Search II, Design Add and Search Words Data Structure.

---

## Suggested revision order if relearning from scratch

1. DFS Bottom-Up (most fundamental, underlies half the others)
2. BFS / Level Order
3. DFS Top-Down
4. BST-specific
5. LCA
6. Construction/Serialization
7. Tree DP
8. Trie (separate study track, not urgent for core tree fluency)

**Habit to build, same as intervals:** after solving any tree problem, ask "is this bottom-up, top-down, or BFS — and why did the problem force that choice?" Saying the *why* out loud is what makes it transfer to the next unfamiliar problem instead of staying memorized to one.
