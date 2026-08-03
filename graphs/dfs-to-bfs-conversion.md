# Converting DFS to BFS (and Back) — Reference Note

A pattern reference for turning any DFS solution into a BFS solution, and knowing when NOT to. Distilled from the LC 547, 323, 841, 797, 133 walkthroughs in [`bfs-dfs-traversal.md`](./bfs-dfs-traversal.md).

---

## TL;DR — the one-sentence version

> **Replace the (implicit or explicit) stack with a queue, and move whatever recursion "remembered for you" into the queue element itself.**

Everything else in this note is unpacking those two ideas.

---

## Why the swap works at all

DFS and BFS are the **same algorithm** with **one difference**: which "next node to visit" you pick out of the frontier.

- **DFS:** frontier is a **stack** (LIFO). You always pick the *most recently added* node → you dive deep.
- **BFS:** frontier is a **queue** (FIFO). You always pick the *earliest added* node → you fan out level by level.

Everything else — the `visited` tracking, the neighbour enumeration, the outer loop for disconnected graphs, the answer-computation — is identical. Which is why the conversion is mostly mechanical.

---

## The mechanical swap (skeleton diff)

### Recursive DFS
```java
void dfs(int node, Graph g, boolean[] visited) {
    visited[node] = true;
    // ... do work with node ...
    for (int nei : g.neighbours(node)) {
        if (!visited[nei]) dfs(nei, g, visited);
    }
}
```

### Iterative DFS (uses an explicit stack — bridge form)
```java
void dfsIter(int start, Graph g, boolean[] visited) {
    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(start);
    visited[start] = true;               // mark on PUSH (see rule below)
    while (!stack.isEmpty()) {
        int node = stack.pop();
        // ... do work with node ...
        for (int nei : g.neighbours(node)) {
            if (!visited[nei]) {
                visited[nei] = true;     // mark BEFORE pushing
                stack.push(nei);
            }
        }
    }
}
```

### BFS
```java
void bfs(int start, Graph g, boolean[] visited) {
    Queue<Integer> queue = new ArrayDeque<>();
    queue.offer(start);
    visited[start] = true;               // mark on ENQUEUE
    while (!queue.isEmpty()) {
        int node = queue.poll();
        // ... do work with node ...
        for (int nei : g.neighbours(node)) {
            if (!visited[nei]) {
                visited[nei] = true;     // mark BEFORE enqueueing
                queue.offer(nei);
            }
        }
    }
}
```

**Spot the difference between iterative DFS and BFS?** It is one character: `push`/`pop` vs `offer`/`poll` (or equivalently, `Deque` used as a stack vs a queue). That is it. If you can write iterative DFS, you can write BFS.

---

## The three universal rules for the swap

### Rule 1 — Mark visited BEFORE you add to the frontier, not after

Both iterative DFS and BFS need this. If you mark on **pop/poll** instead of on **push/offer**, the same node can get added to the frontier multiple times before it is first processed → duplicate work, sometimes wrong answers.

```java
// WRONG
queue.offer(nei);
visited[nei] = true;      // too late — another neighbour of ours might re-add nei

// RIGHT
visited[nei] = true;      // claim the node first
queue.offer(nei);
```

Recursive DFS gets this for free because the recursive call happens immediately — there is no in-between window.

### Rule 2 — Whatever recursion "remembered for you", the queue element must carry

This is the subtle part. In recursive DFS, each stack frame automatically holds its own local variables — the current path, current depth, accumulated sum, whatever. When you flatten recursion to a loop, those locals have to live somewhere. That "somewhere" is **the queue element**.

Concrete examples from the traversal notes:

| Problem | What DFS carried in a stack frame | What BFS packs into each queue entry |
|---------|-----------------------------------|--------------------------------------|
| LC 547 Provinces | just the node | just the node |
| LC 323 Components | just the node | just the node |
| LC 841 Keys and Rooms | just the node | just the node |
| LC 797 All Paths | `path` list (grows & shrinks as we recurse) | a **copy of the whole path so far** per entry |
| LC 133 Clone Graph | the current node (map does the wiring) | the current node (map does the wiring) |
| Shortest path in unweighted grid | node + depth | `(node, depth)` pair |

Whenever DFS uses backtracking (add-then-remove), BFS has to **copy** state on branch. That is why BFS is often more memory-hungry than DFS for enumeration/path problems.

### Rule 3 — Answer-computation lives in the same place, structurally

Whether the "am I done?" or "record this result" check runs before recursing (DFS) or after polling (BFS), the *logic* is the same. Do not try to move it earlier or later during the conversion — just translate the position.

```java
// DFS
if (node == target) result.add(snapshot(path));
else for (int nei : ...) dfs(nei, ...);

// BFS
if (node == target) { result.add(path); continue; }
for (int nei : ...) queue.offer(...);
```

---

## When DFS and BFS are truly interchangeable

You can freely swap between them if the answer does not depend on **order of visitation**:

- Counting connected components (LC 547, 323)
- Reachability / "can we visit all?" (LC 841)
- Deep-copy a graph (LC 133)
- Cycle detection (undirected)
- Enumerating all reachable nodes

In these problems, the choice between DFS and BFS is stylistic (code length, stack safety, personal taste).

---

## When they are NOT interchangeable (or one is strictly better)

### Prefer BFS when...
- **Shortest path in an unweighted graph.** BFS visits nodes in non-decreasing distance order, so the first time you reach the target, that distance is optimal. DFS gives no such guarantee.
- **Level-order questions.** "Nodes at depth k", "cheapest path with at most k stops" — BFS naturally processes one level at a time.
- **Very deep graphs where recursion would overflow.** BFS uses heap-allocated queue memory; recursive DFS uses the JVM/OS stack (much smaller). Convert to iterative when depth might exceed ~10k.

### Prefer DFS when...
- **Enumeration ("return all X").** Backtracking DFS reclaims memory as it un-chooses, so extra space is O(depth). BFS has to keep every in-flight partial state alive in the queue → space blows up. Concrete: LC 797 is O(n) extra for DFS vs O(n · 2^n) for BFS.
- **Constructing a parallel structure via recursion return values.** LC 133 Clone Graph is much cleaner as DFS because the recursion "wires" parent to child through the returned clone. BFS works but requires two map lookups per edge.
- **Post-order / tree-shaped work.** Anything that needs "process children first, then me" (subtree sums, topological sort via DFS, articulation points, Tarjan SCC) reads naturally as recursion.

### Neither works without adaptation
- **Weighted shortest path** → Dijkstra (priority queue, not plain queue).
- **Shortest path with negative edges** → Bellman-Ford, not BFS.
- **All-pairs shortest path** → Floyd-Warshall.

BFS is not "just shortest path" — it is "shortest path when every edge has the same cost."

---

## Worked-out conversion: LC 547 Provinces

**Recursive DFS**
```java
void dfs(int i, int[][] adj, boolean[] visited) {
    visited[i] = true;
    for (int j = 0; j < adj.length; j++) {
        if (adj[i][j] == 1 && !visited[j]) dfs(j, adj, visited);
    }
}
```

**Step 1 — Convert to iterative DFS (explicit stack)**
```java
void dfsIter(int start, int[][] adj, boolean[] visited) {
    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(start);
    visited[start] = true;
    while (!stack.isEmpty()) {
        int i = stack.pop();
        for (int j = 0; j < adj.length; j++) {
            if (adj[i][j] == 1 && !visited[j]) {
                visited[j] = true;
                stack.push(j);
            }
        }
    }
}
```

**Step 2 — Rename `stack.push/pop` → `queue.offer/poll`. Done.**
```java
void bfs(int start, int[][] adj, boolean[] visited) {
    Queue<Integer> queue = new ArrayDeque<>();
    queue.offer(start);
    visited[start] = true;
    while (!queue.isEmpty()) {
        int i = queue.poll();
        for (int j = 0; j < adj.length; j++) {
            if (adj[i][j] == 1 && !visited[j]) {
                visited[j] = true;
                queue.offer(j);
            }
        }
    }
}
```

The visitation order changes (fan-out instead of dive-deep), but for province-counting the order does not matter. Same answer, same complexity.

---

## Worked-out conversion: LC 797 (where state-packing matters)

**Recursive DFS** carries `path` on the call stack; backtracks via `path.remove(...)`.
```java
void dfs(int node, int target, int[][] g,
         List<Integer> path, List<List<Integer>> result) {
    path.add(node);
    if (node == target) result.add(new ArrayList<>(path));   // snapshot
    else for (int nei : g[node]) dfs(nei, target, g, path, result);
    path.remove(path.size() - 1);                            // backtrack
}
```

**BFS conversion** — `path` cannot be shared across queue entries (they all live in the queue simultaneously and cannot backtrack). So each entry carries **its own copy** of the path.
```java
Queue<List<Integer>> q = new ArrayDeque<>();
q.offer(new ArrayList<>(List.of(0)));
while (!q.isEmpty()) {
    List<Integer> path = q.poll();
    int last = path.get(path.size() - 1);
    if (last == target) { result.add(path); continue; }
    for (int nei : g[last]) {
        List<Integer> next = new ArrayList<>(path);          // copy on branch
        next.add(nei);
        q.offer(next);
    }
}
```

This is Rule 2 in action: recursion's implicit "each frame has its own path" becomes BFS's explicit "each queue element has its own path." The time complexity stays the same (O(n · 2^n)), but the space cost goes up because BFS holds all partial paths alive at once.

---

## Conversion cheat sheet

| Step | DFS (recursive) | DFS (iterative) | BFS |
|------|-----------------|-----------------|-----|
| Frontier structure | JVM call stack (implicit) | `Deque` used as stack | `Deque`/`Queue` used as queue |
| Add to frontier | `dfs(nei, ...)` | `stack.push(nei)` | `queue.offer(nei)` |
| Take from frontier | (function call returns) | `stack.pop()` | `queue.poll()` |
| Mark visited | first line of `dfs` | before push | before offer |
| Per-frame state | local variables | pack into a wrapper class/array/tuple | pack into a wrapper class/array/tuple |
| Backtracking | trivial (undo after recursion) | manual save/restore | usually not possible — copy state instead |

---

## Common conversion bugs (and their symptoms)

| Bug | Symptom |
|-----|---------|
| Marking visited on `poll` instead of `offer` | Same node processed twice; TLE or wrong count |
| Sharing a mutable `path` across queue entries | All results end up identical (usually the last-explored path) |
| Forgetting to pack per-frame state into the queue element | `NullPointerException`, wrong distances, off-by-one on depth |
| Trying to backtrack a shared BFS structure | Impossible — BFS has no "undo" moment; use copies instead |
| Using BFS for weighted shortest path | Wrong answer on non-uniform edge weights (use Dijkstra) |
| Using BFS for enumeration ("all paths") | Correct but eats memory; prefer DFS backtracking |

---

## Mental checklist before you convert

1. **What does the recursive DFS carry in each frame?** Just the node? A path? A depth? A running sum? → that is your queue element shape.
2. **Do I need backtracking?** If yes and I am moving to BFS, I need to **copy** state on branch, not share it.
3. **Does the answer depend on visit order?** If no → DFS/BFS are equivalent, pick freely. If yes and I need level-order → BFS. If yes and I need "deepest first" → DFS.
4. **Any depth concerns?** Very deep recursion → prefer iterative DFS or BFS to dodge stack overflow.
5. **Weighted graph?** Neither DFS nor BFS — reach for Dijkstra/Bellman-Ford/etc.

---

## Related notes in this repo

- [`bfs-dfs-traversal.md`](./bfs-dfs-traversal.md) — problem-by-problem walkthroughs (LC 547, 323, 841, 797, 133, 1557) with side-by-side DFS and BFS code.
- [`adjacency-representations.md`](./adjacency-representations.md) — how the input shape (matrix / edge list / adjacency list / grid) drives your neighbour-enumeration code, independent of DFS vs BFS.
