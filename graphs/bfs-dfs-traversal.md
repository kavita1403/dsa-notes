# BFS / DFS Traversal — Revision Notes

Quick-glance notes for graph traversal problems. Each entry captures what tripped me up, the key insight, approaches with complexity, and a pre-flight checklist for next time.

---

## 1. Number of Provinces — LC 547

**Link:** https://leetcode.com/problems/number-of-provinces/

**Problem (1-liner):** Given an `n x n` adjacency **matrix** `isConnected` of an undirected graph, return the number of connected components ("provinces").

### What tripped me up
- Didn't reach for DFS on my own even after getting the hint.
- Got stuck on **how to mark visited nodes** and **how to invoke DFS** across a disconnected graph.
- Kept mentally treating it like a grid problem (look up/down/left/right). It's **not** a grid — neighbours aren't spatial.

### Key insight
- Input is an **adjacency matrix**, not a grid. To find neighbours of node `i`, scan row `i`: every `j` where `isConnected[i][j] == 1` is a neighbour.
- Outer loop over every node `i` from `0..n-1`: if `i` is unvisited, start a DFS/BFS from it and bump the province count. That outer loop is what handles the "graph may be disconnected" case — it *is* the whole point of the problem.
- BFS vs DFS traversal is nearly identical here — only the data structure differs (queue vs stack/recursion).

### Approach 1 — DFS
```java
void dfs(int i, int[][] adj, boolean[] visited) {
    visited[i] = true;
    for (int j = 0; j < adj.length; j++) {
        if (adj[i][j] == 1 && !visited[j]) dfs(j, adj, visited);
    }
}

int findCircleNum(int[][] adj) {
    int n = adj.length, count = 0;
    boolean[] visited = new boolean[n];
    for (int i = 0; i < n; i++) {
        if (!visited[i]) { dfs(i, adj, visited); count++; }
    }
    return count;
}
```
- **Time:** O(N²) — every matrix cell inspected once.
- **Space:** O(N) visited + O(N) recursion stack worst case.

### Approach 2 — BFS
Same structure; replace recursion with a queue. Enqueue `i`, pop, scan its row for `1`s, enqueue unvisited neighbours, mark visited on enqueue (not on pop) to avoid duplicates.
- **Time:** O(N²)  **Space:** O(N).

### Approach 3 — Union-Find (DSU)
- Initialise `parent[i] = i` for all `i`.
- For every pair `i, j` with `j > i` (upper triangle only — matrix is symmetric) where `adj[i][j] == 1`, `union(i, j)`.
- Count distinct roots at the end → that's the province count.
- **Time:** O(N² · α(N))  **Space:** O(N).

### Next-time checklist
1. **Read the input format first.** Matrix vs list vs grid dictates how you enumerate neighbours.
2. **Adjacency matrix → neighbours = scan the row.** No 4-directional movement.
3. **Disconnected graph?** Outer `for i in 0..n-1` + `if !visited[i]` is your friend. Counting these DFS/BFS launches = number of components.
4. **Mark visited BEFORE recursing / on enqueue**, not after — prevents infinite loops and duplicate work.
5. Default first weapon: **DFS** (cleanest code). Reach for **Union-Find** only if the problem streams edges or asks connectivity queries mid-flight.

### Gotchas I missed
- `isConnected[i][i] == 1` always (diagonal is self-loops). Doesn't break anything because `visited[i]` is already `true` when you're processing `i`, but worth knowing.
- Matrix is **symmetric** (undirected). That's why Union-Find can iterate only the upper triangle `j > i` and skip half the work.
- Recursion stack can hit `O(N)` on a fully-connected graph — for interviews on huge N, mention the iterative-stack or BFS variant.
- Don't confuse "province count" with "largest province size" — different problem, easy misread under pressure.

---
