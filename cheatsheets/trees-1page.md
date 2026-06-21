# Trees — Cheat Sheet

**Decision fork:** answer needs info from children → bottom-up (post-order). Answer needs info from ancestors → top-down (pre-order, pass state down). Need level-by-level → BFS.

## DFS Bottom-Up
```java
ReturnType dfs(TreeNode node) {
    if (node == null) return /* base */;
    ReturnType left = dfs(node.left);
    ReturnType right = dfs(node.right);
    return combine(left, right, node.val);
}
```

## DFS Top-Down
```java
void dfs(TreeNode node, State state) {
    if (node == null) return;
    State newState = update(state, node.val);
    if (node.left == null && node.right == null) { /* leaf */ }
    dfs(node.left, newState);
    dfs(node.right, newState);
}
```
⚠️ Don't mutate a shared state object across branches — pass a fresh copy or backtrack.

## BFS / Level Order
```java
int size = queue.size(); // snapshot BEFORE inner loop — the #1 BFS bug if skipped
```

## LCA (general tree)
```java
if (node == null || node == p || node == q) return node;
left = lca(node.left, p, q); right = lca(node.right, p, q);
if (left != null && right != null) return node;
return left != null ? left : right;
```
BST version: walk down using value comparisons, no full traversal needed.

## Validate BST
Pass `(lower, upper)` bounds down, tighten at each node. Use `long` bounds to avoid overflow. Don't only check immediate children — bounds must propagate all the way down.

## Tree DP (multi-state)
Return an array of states per node (e.g., `{robThis, notRobThis}`), not a single value — let parent pick the best combo.

## Trie
Children keyed by character (`Map<Character, TrieNode>`), mark `isWord` explicitly at the end of insert.
