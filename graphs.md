# Graphs — Master Reference Note

```java
// The four questions to answer for EVERY graph problem:
// 1. REPRESENTATION - adjacency list, matrix, or implicit grid? Directed or undirected? Weighted?
// 2. TRAVERSAL       - BFS (level/shortest unweighted path) or DFS (exploration, cycles, components)?
// 3. STATE            - what do you track per node? (visited, distance, color, parent, in-degree)
// 4. TERMINATION      - when do you stop? all nodes visited, queue empty, target found?

// Two traversal skeletons:
void dfs(int node, boolean[] visited, List<List<Integer>> adj) {
    visited[node] = true;
    for (int next : adj.get(node)) {
        if (!visited[next]) dfs(next, visited, adj);
    }
}
void bfs(int start, List<List<Integer>> adj) {
    Queue<Integer> q = new LinkedList<>();
    boolean[] visited = new boolean[adj.size()];
    q.offer(start); visited[start] = true;
    while (!q.isEmpty()) {
        int node = q.poll();
        for (int next : adj.get(node)) {
            if (!visited[next]) { visited[next] = true; q.offer(next); }
        }
    }
}
```

---

## Decision Fork (ask yourself this before writing any code)

> "Shortest path, UNWEIGHTED graph/grid" -> **Plain BFS** (queue, level by level). e.g., rotting oranges, word ladder.
>
> "Shortest path, WEIGHTED graph, no negative edges" -> **Dijkstra** (priority queue, greedy relax).
>
> "Shortest path, WEIGHTED graph, negative edges allowed" -> **Bellman-Ford** (relax all edges V-1 times).
>
> "Shortest path between ALL pairs" -> **Floyd-Warshall** (O(V^3), DP over intermediate nodes).
>
> "Minimum cost to CONNECT all nodes" -> **MST: Prim's or Kruskal's**.
>
> "Are these two nodes connected? / dynamic connectivity / cycle detection in undirected graph" -> **Union-Find (DSU)**.
>
> "Order tasks with dependencies" -> **Topological Sort** (Kahn's BFS or DFS post-order).
>
> "Detect cycle in directed graph" -> **DFS with 3-color state** (white/gray/black) or Kahn's (leftover nodes = cycle).
>
> "Detect cycle in undirected graph" -> **Union-Find** or **DFS with parent tracking**.
>
> "Count connected components / islands" -> **DFS/BFS flood fill** or **Union-Find**.
>
> "Bipartite check / 2-coloring" -> **BFS/DFS graph coloring**.
>
> "Find bridges / articulation points" -> **Tarjan's (DFS with discovery + low-link times)**.
>
> "Strongly connected components (directed)" -> **Kosaraju's or Tarjan's SCC**.
>
> "Grid problem - flood fill, multi-source spread" -> **BFS from multiple sources simultaneously**.
>
> "Shortest path with STATE (keys, obstacles removed, fuel)" -> **BFS with state in visited-set, not just node**.
>
> "Find if path exists with some constraint (alternating colors, k stops)" -> **Modified BFS/DFS with extra dimension in state**.

Everything else in Graph-land is a variant or combination of these.

---

## Quick Lookup Table (read this first when revisiting after a gap)

| # | Pattern | Trigger phrases | Anchor problem | Core idea |
|---|---|---|---|---|
| 1 | BFS Traversal | "shortest path unweighted", "level order", "minimum steps" | Rotting Oranges | Queue, level-by-level expansion |
| 2 | DFS Traversal | "explore all paths", "connected components", "flood fill" | Number of Islands | Recursive/stack-based deep exploration |
| 3 | Multi-source BFS | "rotting oranges", "walls and gates", "minimum time to spread" | Rotting Oranges | Push ALL sources into queue before starting BFS |
| 4 | Topological Sort | "course schedule", "task order", "build order", "alien dictionary" | Course Schedule II | Kahn's (in-degree + queue) or DFS post-order reverse |
| 5 | Cycle Detection (Directed) | "course schedule (can finish?)", "detect cycle directed" | Course Schedule | 3-color DFS or Kahn's leftover-node check |
| 6 | Cycle Detection (Undirected) | "redundant connection", "detect cycle undirected" | Redundant Connection | Union-Find: cycle if union fails (same root) |
| 7 | Union-Find (DSU) | "number of provinces", "redundant connection", "accounts merge", "smallest equivalent string" | Number of Provinces | Path compression + union by rank/size |
| 8 | Dijkstra's | "network delay time", "cheapest flights", "path with min effort" | Network Delay Time | PQ, greedily relax shortest known distance |
| 9 | Bellman-Ford | "cheapest flights within K stops", "negative edge weights" | Cheapest Flights Within K Stops | Relax all edges K times (bounded by stops/V-1) |
| 10 | Floyd-Warshall | "find the city with smallest number of neighbors", "all pairs shortest path" | Find the City With the Smallest Number of Neighbors | dp[i][j] via intermediate k |
| 11 | MST (Prim's / Kruskal's) | "min cost to connect all points", "min cost to connect cities" | Min Cost to Connect All Points | Kruskal: sort edges + Union-Find. Prim: PQ from current tree |
| 12 | Bipartite Check | "possible bipartition", "is graph bipartite", "graph coloring with 2 colors" | Is Graph Bipartite? | BFS/DFS 2-coloring; conflict = not bipartite |
| 13 | BFS with State | "shortest path with keys", "shortest path getting all keys", "minimum obstacle removal" | Shortest Path to Get All Keys | visited = (node, extra-state) not just node |
| 14 | Bridges & Articulation Points | "critical connections in a network", "find bridges" | Critical Connections in a Network | Tarjan's: discovery time + low-link value |
| 15 | Strongly Connected Components | "strongly connected components", "critical nodes in directed graph" | (specialized) | Kosaraju's (2-pass DFS) or Tarjan's (1-pass, low-link) |
| 16 | Grid as Implicit Graph | "number of islands", "max area of island", "surrounded regions" | Number of Islands | Treat each cell as node, 4-directional neighbors as edges |

---

## 1. BFS Traversal (Shortest Path, Unweighted)

**When to use it:** Any "minimum steps / shortest path" question where all edges have equal weight (or no weight at all, like grids).

**Trigger phrases:** "shortest path unweighted", "minimum number of steps", "level order traversal", "minimum knight moves", "word ladder".

**Anchor problem:** Word Ladder / Minimum Knight Moves (generic BFS shortest path).

**Template:**
```java
public int bfsShortestPath(int start, int target, List<List<Integer>> adj) {
    Queue<Integer> queue = new LinkedList<>();
    boolean[] visited = new boolean[adj.size()];
    queue.offer(start);
    visited[start] = true;
    int steps = 0;

    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {          // process one full LEVEL at a time
            int node = queue.poll();
            if (node == target) return steps;
            for (int next : adj.get(node)) {
                if (!visited[next]) {
                    visited[next] = true;
                    queue.offer(next);
                }
            }
        }
        steps++;
    }
    return -1;                                     // target unreachable
}
```
**Why level-by-level (`size` snapshot)?** This is how you count "steps"/"levels" cleanly — without it, you can't tell when one level ends and the next begins.

**Variations:**
- **Word Ladder**: each word is a node; edge exists if words differ by 1 letter. Generate neighbors on the fly (don't precompute full graph — too slow).
- **Minimum Knight Moves**: grid BFS with 8 fixed move offsets.
- **Open the Lock**: each 4-digit combo is a node; edges = turning one wheel by 1.

**Common bugs:**
- Marking visited when DEQUEUED instead of when ENQUEUED — causes duplicate enqueues, wrong distances, even infinite loops.
- Forgetting the level-snapshot (`size = queue.size()`) and instead checking `queue.size()` inside the loop where it's mutating.
- Not handling start == target as a 0-step edge case.

---

## 2. DFS Traversal (Exploration, Components)

**When to use it:** Explore every reachable node, count components, or explore "all paths" (not shortest).

**Trigger phrases:** "connected components", "flood fill", "all paths from source to target", "number of islands".

**Anchor problem:** Number of Islands.

**Template (recursive):**
```java
public int numIslands(char[][] grid) {
    int count = 0;
    for (int i = 0; i < grid.length; i++) {
        for (int j = 0; j < grid[0].length; j++) {
            if (grid[i][j] == '1') {
                count++;
                dfs(grid, i, j);
            }
        }
    }
    return count;
}
void dfs(char[][] grid, int i, int j) {
    if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] != '1') return;
    grid[i][j] = '0';                              // mark visited in-place
    dfs(grid, i + 1, j); dfs(grid, i - 1, j);
    dfs(grid, i, j + 1); dfs(grid, i, j - 1);
}
```

**Template (iterative with explicit stack — avoids stack overflow on large grids):**
```java
void dfsIterative(char[][] grid, int si, int sj) {
    Deque<int[]> stack = new ArrayDeque<>();
    stack.push(new int[]{si, sj});
    grid[si][sj] = '0';
    int[] dx = {0, 0, 1, -1}, dy = {1, -1, 0, 0};
    while (!stack.isEmpty()) {
        int[] cur = stack.pop();
        for (int d = 0; d < 4; d++) {
            int ni = cur[0] + dx[d], nj = cur[1] + dy[d];
            if (ni >= 0 && ni < grid.length && nj >= 0 && nj < grid[0].length && grid[ni][nj] == '1') {
                grid[ni][nj] = '0';
                stack.push(new int[]{ni, nj});
            }
        }
    }
}
```

**Variations:**
- **Max Area of Island**: DFS returns size of component instead of just marking it.
- **Surrounded Regions**: DFS from BORDER cells first (mark safe), then flip everything else.
- **All Paths From Source to Target** (DAG): backtracking DFS, add/remove from path list.

**Common bugs:**
- Recursive DFS on huge grids (10^5+ cells) → stack overflow. Switch to iterative or BFS.
- Forgetting to mark visited BEFORE recursing (or right when pushed to stack) → revisits, infinite loop in cyclic graphs.
- Modifying grid in-place when you need original values later — clone if so.

---

## 3. Multi-Source BFS

**When to use it:** Several starting points spread "simultaneously" — minimum time for ALL of them to reach everywhere, or distance from the NEAREST of several sources.

**Trigger phrases:** "rotting oranges", "walls and gates", "minimum time to infect/spread", "map of highest peak".

**Anchor problem:** Rotting Oranges.

**Template:**
```java
public int orangesRotting(int[][] grid) {
    int m = grid.length, n = grid[0].length, fresh = 0;
    Queue<int[]> queue = new LinkedList<>();

    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == 2) queue.offer(new int[]{i, j});   // push ALL rotten oranges first
            else if (grid[i][j] == 1) fresh++;
        }
    }

    int[] dx = {0, 0, 1, -1}, dy = {1, -1, 0, 0};
    int minutes = 0;
    while (!queue.isEmpty() && fresh > 0) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            int[] cur = queue.poll();
            for (int d = 0; d < 4; d++) {
                int ni = cur[0] + dx[d], nj = cur[1] + dy[d];
                if (ni >= 0 && ni < m && nj >= 0 && nj < n && grid[ni][nj] == 1) {
                    grid[ni][nj] = 2;
                    fresh--;
                    queue.offer(new int[]{ni, nj});
                }
            }
        }
        minutes++;
    }
    return fresh == 0 ? minutes : -1;
}
```
**Key insight:** push ALL sources into the queue BEFORE the BFS loop starts — this makes them all "level 0" simultaneously, so distances computed are "distance from nearest source."

**Variations:**
- **Walls and Gates**: multi-source BFS from all gates, fill distance into rooms.
- **Map of Highest Peak**: multi-source BFS from all water cells (height 0).
- **01 Matrix**: multi-source BFS from all 0-cells, find distance to nearest 0 for each 1-cell.

**Common bugs:**
- Running separate BFS per source instead of combining into one queue — gives wrong (non-minimal) distances when sources overlap in coverage.
- Forgetting edge case: 0 fresh oranges initially → answer is 0, not -1.

---

## 4. Topological Sort

**When to use it:** Order items with "must come before" dependencies — only works on **DAGs** (Directed Acyclic Graphs).

**Trigger phrases:** "course schedule II (return order)", "build order", "task scheduling order", "alien dictionary".

**Anchor problem:** Course Schedule II.

**Template (Kahn's Algorithm — BFS based, easier to reason about):**
```java
public int[] findOrder(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    int[] indegree = new int[numCourses];

    for (int[] p : prerequisites) {
        adj.get(p[1]).add(p[0]);                   // p[1] -> p[0]
        indegree[p[0]]++;
    }

    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) {
        if (indegree[i] == 0) queue.offer(i);
    }

    int[] order = new int[numCourses];
    int idx = 0;
    while (!queue.isEmpty()) {
        int node = queue.poll();
        order[idx++] = node;
        for (int next : adj.get(node)) {
            if (--indegree[next] == 0) queue.offer(next);
        }
    }
    return idx == numCourses ? order : new int[0];  // idx < numCourses means a cycle exists
}
```

**Template (DFS-based — post-order then reverse):**
```java
List<Integer> order = new ArrayList<>();
boolean[] visited = new boolean[n], onStack = new boolean[n];
boolean hasCycle = false;

void dfs(int node, List<List<Integer>> adj) {
    visited[node] = true;
    onStack[node] = true;
    for (int next : adj.get(node)) {
        if (onStack[next]) { hasCycle = true; return; }
        if (!visited[next]) dfs(next, adj);
    }
    onStack[node] = false;
    order.add(node);                                // post-order: add AFTER exploring all descendants
}
// after running dfs on all unvisited nodes: Collections.reverse(order);
```

**Variations:**
- **Alien Dictionary**: build edges from adjacent word comparisons, then topo sort the alphabet.
- **Course Schedule** (just yes/no, not order): same as cycle detection — if topo sort can't include all nodes, there's a cycle.
- **Minimum Height Trees**: BFS-based "peel leaves" topological-style approach.

**Common bugs:**
- Building the edge direction backwards (`p[0] -> p[1]` instead of `p[1] -> p[0]`, or vice versa) — completely inverts the order.
- For Kahn's: forgetting to check `idx == numCourses` at the end — silently returning a partial/wrong order when a cycle exists.
- For DFS-based: forgetting to reverse the post-order list at the end.

---

## 5. Cycle Detection — Directed Graph

**When to use it:** Determine if a directed graph has a cycle (e.g., circular dependencies).

**Trigger phrases:** "course schedule (can finish all courses?)", "detect cycle in directed graph".

**Anchor problem:** Course Schedule.

**Template (3-color DFS — white/gray/black):**
```java
// 0 = unvisited (white), 1 = in current DFS path (gray), 2 = fully processed (black)
public boolean canFinish(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    for (int[] p : prerequisites) adj.get(p[1]).add(p[0]);

    int[] state = new int[numCourses];
    for (int i = 0; i < numCourses; i++) {
        if (state[i] == 0 && hasCycle(i, adj, state)) return false;
    }
    return true;
}
boolean hasCycle(int node, List<List<Integer>> adj, int[] state) {
    state[node] = 1;                                 // mark as "in progress"
    for (int next : adj.get(node)) {
        if (state[next] == 1) return true;            // back-edge to a node in current path -> cycle
        if (state[next] == 0 && hasCycle(next, adj, state)) return true;
    }
    state[node] = 2;                                  // mark as "fully done"
    return false;
}
```
**Why 3 states, not just visited/unvisited?** A node finished in a PREVIOUS DFS branch (state 2) is safe to revisit. A node currently ON THE STACK (state 1) means you've looped back — that's the actual cycle signal. Two-state visited tracking can't distinguish these.

**Alternative:** run Kahn's topological sort — if you can't process all `n` nodes (some never reach indegree 0), there's a cycle. Often simpler to implement correctly under pressure.

**Common bugs:**
- Using only boolean `visited[]` (2-state) instead of 3-state — gives false positives on diamond-shaped DAGs (`A->B->D`, `A->C->D` is NOT a cycle, but 2-state marking can incorrectly flag it).
- Forgetting to reset/check `state[node] = 2` after the node's subtree is fully explored.

---

## 6. Cycle Detection — Undirected Graph

**When to use it:** Determine if an undirected graph has a cycle (different from directed — must avoid falsely flagging the edge back to your own parent).

**Trigger phrases:** "redundant connection", "detect cycle undirected graph", "graph valid tree".

**Anchor problem:** Redundant Connection.

**Template (Union-Find — cleanest for undirected cycle detection):**
```java
public int[] findRedundantConnection(int[][] edges) {
    int n = edges.length;
    int[] parent = new int[n + 1];
    for (int i = 0; i <= n; i++) parent[i] = i;

    for (int[] e : edges) {
        int ru = find(parent, e[0]), rv = find(parent, e[1]);
        if (ru == rv) return e;                       // already connected -> this edge creates a cycle
        parent[ru] = rv;
    }
    return new int[0];
}
int find(int[] parent, int x) {
    if (parent[x] != x) parent[x] = find(parent, parent[x]);  // path compression
    return parent[x];
}
```

**Template (DFS with parent tracking — alternative):**
```java
boolean hasCycle(int node, int parent, boolean[] visited, List<List<Integer>> adj) {
    visited[node] = true;
    for (int next : adj.get(node)) {
        if (next == parent) continue;                  // skip the edge back to where we came from
        if (visited[next]) return true;                // visited, and not parent -> cycle
        if (hasCycle(next, node, visited, adj)) return true;
    }
    return false;
}
```

**Variations:**
- **Graph Valid Tree**: valid tree iff exactly `n-1` edges AND no cycle AND fully connected (Union-Find handles all 3 checks naturally).
- **Redundant Connection II** (directed version with extra twist of "2 parents"): more involved — combines indegree check + Union-Find.

**Common bugs:**
- In DFS approach: forgetting to skip the parent edge — incorrectly treats every edge as a "back edge" since undirected edges appear in both directions in adjacency lists.
- In Union-Find: forgetting path compression — degrades to O(n) per find, O(n^2) overall on large inputs.

---

## 7. Union-Find (Disjoint Set Union / DSU)

**When to use it:** Dynamic connectivity questions — "are these connected?", "merge groups", "count groups" — especially when queries are interleaved with unions.

**Trigger phrases:** "number of provinces", "accounts merge", "smallest equivalent string", "number of operations to make network connected".

**Anchor problem:** Number of Provinces.

**Template (with path compression + union by size — the production-ready version):**
```java
class UnionFind {
    int[] parent, size;
    int count;                                          // number of distinct components

    UnionFind(int n) {
        parent = new int[n];
        size = new int[n];
        count = n;
        for (int i = 0; i < n; i++) { parent[i] = i; size[i] = 1; }
    }

    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);  // path compression
        return parent[x];
    }

    boolean union(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;                      // already connected
        if (size[rx] < size[ry]) { int t = rx; rx = ry; ry = t; }  // union by size
        parent[ry] = rx;
        size[rx] += size[ry];
        count--;
        return true;
    }
}
```

**Usage (Number of Provinces):**
```java
public int findCircleNum(int[][] isConnected) {
    int n = isConnected.length;
    UnionFind uf = new UnionFind(n);
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            if (isConnected[i][j] == 1) uf.union(i, j);
        }
    }
    return uf.count;
}
```

**Variations:**
- **Accounts Merge**: union accounts sharing any email, then group by root.
- **Smallest Equivalent String**: union characters, pick smallest char as representative per group.
- **Number of Operations to Make Network Connected**: need `edges >= n-1` first, then answer = `components - 1`.

**Common bugs:**
- Forgetting path compression — find() degrades to O(n), making the whole structure effectively O(n^2).
- Forgetting union by rank/size — without it, trees can degenerate into long chains.
- Off-by-one when nodes are 1-indexed vs the array being 0-indexed.

---

## 8. Dijkstra's Algorithm

**When to use it:** Shortest path in a weighted graph with **non-negative edge weights**. The greedy "always expand the closest unvisited node" approach.

**Trigger phrases:** "network delay time", "cheapest flights" (without K-stop constraint), "path with minimum effort", "path with maximum probability".

**Anchor problem:** Network Delay Time.

**Template:**
```java
public int networkDelayTime(int[][] times, int n, int k) {
    List<List<int[]>> adj = new ArrayList<>();          // adj.get(u) = list of {v, weight}
    for (int i = 0; i <= n; i++) adj.add(new ArrayList<>());
    for (int[] t : times) adj.get(t[0]).add(new int[]{t[1], t[2]});

    int[] dist = new int[n + 1];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[k] = 0;

    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);  // {node, distance}
    pq.offer(new int[]{k, 0});

    while (!pq.isEmpty()) {
        int[] cur = pq.poll();
        int node = cur[0], d = cur[1];
        if (d > dist[node]) continue;                    // stale entry, skip (lazy deletion)

        for (int[] edge : adj.get(node)) {
            int next = edge[0], weight = edge[1];
            if (dist[node] + weight < dist[next]) {
                dist[next] = dist[node] + weight;
                pq.offer(new int[]{next, dist[next]});
            }
        }
    }

    int maxDist = 0;
    for (int i = 1; i <= n; i++) {
        if (dist[i] == Integer.MAX_VALUE) return -1;
        maxDist = Math.max(maxDist, dist[i]);
    }
    return maxDist;
}
```
**Why `if (d > dist[node]) continue`?** Java's PQ doesn't support efficient key-decrease, so you allow duplicate/stale entries into the queue and just skip them lazily when popped — far simpler than implementing a custom decrease-key heap.

**Variations:**
- **Path With Minimum Effort**: same Dijkstra skeleton, but the "distance" being minimized is the max edge weight along the path, not the sum.
- **Path With Maximum Probability**: same skeleton, but use a MAX-heap and multiply probabilities instead of summing weights.
- **Cheapest Flights Within K Stops**: Dijkstra needs modification (track stops too) — usually cleaner with Bellman-Ford instead (see #9).

**Common bugs:**
- Using Dijkstra when negative edges exist — gives WRONG answers silently (no error, just incorrect result). Must use Bellman-Ford instead.
- Forgetting the stale-entry check — without it, correctness still holds but you do wasted work; with very large graphs this can TLE.
- 0-indexed vs 1-indexed node confusion (very common in this specific problem family).

---

## 9. Bellman-Ford

**When to use it:** Shortest path with **negative edge weights allowed**, or when the path is constrained by a maximum number of edges/stops.

**Trigger phrases:** "cheapest flights within K stops", "negative edge weights", "detect negative cycle".

**Anchor problem:** Cheapest Flights Within K Stops.

**Template:**
```java
public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    for (int i = 0; i <= k; i++) {                       // relax edges (k+1) times = at most k+1 edges in path
        int[] temp = dist.clone();                        // IMPORTANT: use a snapshot, don't mutate dist mid-relaxation
        for (int[] f : flights) {
            int u = f[0], v = f[1], w = f[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < temp[v]) {
                temp[v] = dist[u] + w;
            }
        }
        dist = temp;
    }
    return dist[dst] == Integer.MAX_VALUE ? -1 : dist[dst];
}
```
**Why clone `dist` each round?** Bellman-Ford relaxes in ROUNDS — within one round, all edges should use distances from the END of the PREVIOUS round, not partially-updated values from the same round. Mutating in place can cause a single round to behave like multiple rounds, breaking the K-stops constraint.

**Generic Bellman-Ford (V-1 rounds, detect negative cycle on round V):**
```java
int[] dist = new int[n];
Arrays.fill(dist, Integer.MAX_VALUE);
dist[src] = 0;
for (int i = 0; i < n - 1; i++) {
    for (int[] edge : edges) {
        int u = edge[0], v = edge[1], w = edge[2];
        if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) dist[v] = dist[u] + w;
    }
}
// One more pass: if any distance still improves, there's a negative cycle
for (int[] edge : edges) {
    int u = edge[0], v = edge[1], w = edge[2];
    if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) { /* negative cycle exists */ }
}
```

**Common bugs:**
- Mutating `dist` in place during a single relaxation round — breaks the "K stops" semantics (silently allows more hops than intended).
- Not running the extra (V-th) pass when negative-cycle detection is required.
- Confusing "K stops" with "K edges" — K stops means K+1 edges in the path; off-by-one is extremely common here.

---

## 10. Floyd-Warshall (All-Pairs Shortest Path)

**When to use it:** Need shortest paths between EVERY pair of nodes, and graph is small enough (`V <= ~400`) for O(V^3).

**Trigger phrases:** "all pairs shortest path", "find the city with smallest number of reachable neighbors within threshold".

**Anchor problem:** Find the City With the Smallest Number of Neighbors at a Threshold Distance.

**Template:**
```java
public int findTheCity(int n, int[][] edges, int distanceThreshold) {
    int[][] dist = new int[n][n];
    for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE / 2);
    for (int i = 0; i < n; i++) dist[i][i] = 0;
    for (int[] e : edges) {
        dist[e[0]][e[1]] = e[2];
        dist[e[1]][e[0]] = e[2];                          // undirected
    }

    for (int k = 0; k < n; k++) {                          // k = intermediate node
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                dist[i][j] = Math.min(dist[i][j], dist[i][k] + dist[k][j]);
            }
        }
    }

    int bestCity = -1, minCount = Integer.MAX_VALUE;
    for (int i = 0; i < n; i++) {
        int count = 0;
        for (int j = 0; j < n; j++) {
            if (i != j && dist[i][j] <= distanceThreshold) count++;
        }
        if (count <= minCount) { minCount = count; bestCity = i; }
    }
    return bestCity;
}
```
**Core idea (DP framing):** `dist[i][j]` after considering intermediate node `k` = "shortest path from i to j using only nodes `0..k` as possible waypoints." This is literally a DP over which nodes are "allowed" as stepping stones.

**Common bugs:**
- Loop order MUST be `k` (outermost), then `i`, then `j` — this is non-negotiable, unlike many other nested loops.
- Sentinel overflow: `Integer.MAX_VALUE + Integer.MAX_VALUE` overflows to negative — use `MAX_VALUE / 2` as the "infinity" sentinel instead.

---

## 11. Minimum Spanning Tree (Prim's / Kruskal's)

**When to use it:** Connect all nodes with minimum total edge weight, using exactly `n-1` edges, no cycles.

**Trigger phrases:** "min cost to connect all points", "min cost to connect all cities", "optimize water distribution in a village".

**Anchor problem:** Min Cost to Connect All Points.

**Template (Kruskal's — sort edges + Union-Find, best when edges are given as a list):**
```java
public int minCostConnectPoints(int[][] points) {
    int n = points.length;
    List<int[]> edges = new ArrayList<>();              // {cost, u, v}
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            int cost = Math.abs(points[i][0] - points[j][0]) + Math.abs(points[i][1] - points[j][1]);
            edges.add(new int[]{cost, i, j});
        }
    }
    edges.sort((a, b) -> a[0] - b[0]);

    UnionFind uf = new UnionFind(n);
    int totalCost = 0, edgesUsed = 0;
    for (int[] e : edges) {
        if (uf.union(e[1], e[2])) {                       // union returns true if it actually merged 2 groups
            totalCost += e[0];
            edgesUsed++;
            if (edgesUsed == n - 1) break;
        }
    }
    return totalCost;
}
```

**Template (Prim's — PQ-based, best when graph is given as adjacency list and is dense):**
```java
public int minCostConnectPoints(int[][] points) {
    int n = points.length;
    boolean[] inMST = new boolean[n];
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);  // {cost, node}
    pq.offer(new int[]{0, 0});
    int totalCost = 0, nodesAdded = 0;

    while (nodesAdded < n) {
        int[] cur = pq.poll();
        int cost = cur[0], node = cur[1];
        if (inMST[node]) continue;                        // lazy deletion of stale entries

        inMST[node] = true;
        totalCost += cost;
        nodesAdded++;

        for (int next = 0; next < n; next++) {
            if (!inMST[next]) {
                int dist = Math.abs(points[node][0] - points[next][0]) + Math.abs(points[node][1] - points[next][1]);
                pq.offer(new int[]{dist, next});
            }
        }
    }
    return totalCost;
}
```

**When to choose which:** Kruskal's is simpler when you already have an edge list and graph is sparse. Prim's is better for dense graphs or when generating edges on the fly (like geometric points) is cheap.

**Common bugs:**
- Forgetting to stop early once `n-1` edges are used in Kruskal's (not strictly required for correctness here, but wastes time on large inputs).
- In Prim's: not checking `inMST[node]` before processing a popped entry — leads to double-counting cost.

---

## 12. Bipartite Check (2-Coloring)

**When to use it:** Can you split nodes into 2 groups such that no edge connects two nodes in the SAME group?

**Trigger phrases:** "is graph bipartite", "possible bipartition", "graph coloring with 2 colors".

**Anchor problem:** Is Graph Bipartite?

**Template (BFS coloring):**
```java
public boolean isBipartite(int[][] graph) {
    int n = graph.length;
    int[] color = new int[n];                            // 0 = uncolored, 1 = color A, -1 = color B

    for (int i = 0; i < n; i++) {
        if (color[i] != 0) continue;
        color[i] = 1;
        Queue<Integer> queue = new LinkedList<>();
        queue.offer(i);
        while (!queue.isEmpty()) {
            int node = queue.poll();
            for (int next : graph[node]) {
                if (color[next] == 0) {
                    color[next] = -color[node];           // opposite color
                    queue.offer(next);
                } else if (color[next] == color[node]) {
                    return false;                          // same color as neighbor -> conflict
                }
            }
        }
    }
    return true;
}
```

**Variations:**
- **Possible Bipartition**: identical pattern, just different input format (list of "dislike" pairs instead of adjacency list).

**Common bugs:**
- Forgetting to loop over ALL nodes as potential starting points — graph might be disconnected, single BFS from node 0 won't cover everything.
- Using `0/1` for colors instead of `1/-1` makes the "opposite color" assignment clunkier (need if-else instead of negation).

---

## 13. BFS with Extra State

**When to use it:** Shortest path questions where "visited" can't just be the node — you also need to track some additional condition (keys collected, obstacles removed, parity of moves).

**Trigger phrases:** "shortest path to get all keys", "shortest path with alternating colors", "minimum number of days to eat fruits with cooldown".

**Anchor problem:** Shortest Path to Get All Keys.

**Template:**
```java
public int shortestPathAllKeys(String[] grid) {
    int m = grid.length, n = grid[0].length();
    int startR = 0, startC = 0, allKeys = 0;
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            char c = grid[i].charAt(j);
            if (c == '@') { startR = i; startC = j; }
            if (c >= 'a' && c <= 'f') allKeys |= (1 << (c - 'a'));
        }
    }

    // state = (row, col, keysCollectedBitmask) -- NOT just (row, col)
    Set<String> visited = new HashSet<>();
    Queue<int[]> queue = new LinkedList<>();             // {row, col, keysMask}
    queue.offer(new int[]{startR, startC, 0});
    visited.add(startR + "," + startC + "," + 0);

    int[] dx = {0, 0, 1, -1}, dy = {1, -1, 0, 0};
    int steps = 0;

    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int s = 0; s < size; s++) {
            int[] cur = queue.poll();
            int r = cur[0], c = cur[1], keys = cur[2];
            if (keys == allKeys) return steps;

            for (int d = 0; d < 4; d++) {
                int nr = r + dx[d], nc = c + dy[d];
                if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
                char ch = grid[nr].charAt(nc);
                if (ch == '#') continue;                  // wall
                int newKeys = keys;
                if (ch >= 'A' && ch <= 'F' && (keys & (1 << (ch - 'A'))) == 0) continue;  // locked, no key yet
                if (ch >= 'a' && ch <= 'f') newKeys |= (1 << (ch - 'a'));  // pick up key

                String state = nr + "," + nc + "," + newKeys;
                if (!visited.contains(state)) {
                    visited.add(state);
                    queue.offer(new int[]{nr, nc, newKeys});
                }
            }
        }
        steps++;
    }
    return -1;
}
```
**Key insight:** the SAME `(row, col)` cell can be legitimately visited multiple times with DIFFERENT key-sets — so `visited` must encode `(position, extra-state)`, not just position. This is the single most important idea in this whole category.

**Common bugs:**
- Using `visited[row][col]` (2D boolean array) instead of `visited[row][col][stateMask]` — silently blocks valid revisits with a different state, giving wrong (too-large or -1) answers.
- Forgetting that picking up a key should update `newKeys` only on the NEXT state, not retroactively on the current one mid-loop.

---

## 14. Bridges & Articulation Points (Tarjan's)

**When to use it:** Find edges/nodes whose removal would disconnect the graph (critical infrastructure-style problems).

**Trigger phrases:** "critical connections in a network", "find bridges", "find articulation points".

**Anchor problem:** Critical Connections in a Network.

**Template:**
```java
int timer = 0;
int[] disc, low;
List<List<Integer>> adj;
List<List<Integer>> bridges;

public List<List<Integer>> criticalConnections(int n, List<List<Integer>> connections) {
    adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (List<Integer> c : connections) {
        adj.get(c.get(0)).add(c.get(1));
        adj.get(c.get(1)).add(c.get(0));
    }
    disc = new int[n]; low = new int[n];
    Arrays.fill(disc, -1);
    bridges = new ArrayList<>();
    dfs(0, -1, n);
    return bridges;
}

void dfs(int node, int parent, int n) {
    disc[node] = low[node] = timer++;
    for (int next : adj.get(node)) {
        if (next == parent) continue;                    // skip the edge back to direct parent
        if (disc[next] == -1) {                            // not visited yet
            dfs(next, node, n);
            low[node] = Math.min(low[node], low[next]);
            if (low[next] > disc[node]) {                  // KEY CHECK: bridge condition
                bridges.add(Arrays.asList(node, next));
            }
        } else {
            low[node] = Math.min(low[node], disc[next]);   // back edge found
        }
    }
}
```
**Intuition:** `low[node]` = earliest "discovery time" reachable from `node`'s subtree (including back edges). If a child's subtree can't reach back to `node` or earlier (`low[next] > disc[node]`), then the edge `node-next` is a bridge — removing it disconnects the graph.

**Common bugs:**
- Forgetting to skip the parent edge — undirected adjacency lists store edges both ways, so without this check you'd treat the edge back to your own parent as a "back edge" to a different component.
- Confusing `disc[]` (discovery time, fixed once set) with `low[]` (can be updated as DFS explores children) — they serve different purposes.

---

## 15. Strongly Connected Components (SCC)

**When to use it:** In a DIRECTED graph, find maximal groups of nodes where every node can reach every other node in the group.

**Trigger phrases:** "strongly connected components", "critical nodes in directed graph", "minimum edges to make graph strongly connected".

**Template (Kosaraju's — easier to remember, 2-pass DFS):**
```java
// Pass 1: DFS on original graph, push nodes onto stack in POST-ORDER (finish order)
// Pass 2: DFS on REVERSED graph, popping from stack -- each DFS tree = one SCC

void kosaraju(int n, List<List<Integer>> adj, List<List<Integer>> reverseAdj) {
    boolean[] visited = new boolean[n];
    Deque<Integer> stack = new ArrayDeque<>();
    for (int i = 0; i < n; i++) {
        if (!visited[i]) fillOrder(i, visited, adj, stack);
    }

    Arrays.fill(visited, false);
    List<List<Integer>> sccs = new ArrayList<>();
    while (!stack.isEmpty()) {
        int node = stack.pop();
        if (!visited[node]) {
            List<Integer> component = new ArrayList<>();
            dfsCollect(node, visited, reverseAdj, component);
            sccs.add(component);
        }
    }
}
void fillOrder(int node, boolean[] visited, List<List<Integer>> adj, Deque<Integer> stack) {
    visited[node] = true;
    for (int next : adj.get(node)) if (!visited[next]) fillOrder(next, visited, adj, stack);
    stack.push(node);                                     // post-order push
}
void dfsCollect(int node, boolean[] visited, List<List<Integer>> reverseAdj, List<Integer> component) {
    visited[node] = true;
    component.add(node);
    for (int next : reverseAdj.get(node)) if (!visited[next]) dfsCollect(next, visited, reverseAdj, component);
}
```

**Common bugs:**
- Forgetting to build the REVERSED graph for pass 2 — this is the entire trick of Kosaraju's; without it you just get connected components, not strongly connected ones.
- Not popping in the correct stack order for pass 2.

---

## 16. Grid as Implicit Graph

**When to use it:** Most grid problems ARE graph problems — each cell is a node, adjacent cells (4-dir or 8-dir) are edges. No explicit adjacency list needed.

**Trigger phrases:** "number of islands", "max area of island", "surrounded regions", "pacific atlantic water flow".

**Template (generic 4-directional neighbor generation, reusable everywhere):**
```java
int[] dx = {0, 0, 1, -1}, dy = {1, -1, 0, 0};
for (int d = 0; d < 4; d++) {
    int ni = i + dx[d], nj = j + dy[d];
    if (ni >= 0 && ni < rows && nj >= 0 && nj < cols /* && other conditions */) {
        // process neighbor (ni, nj)
    }
}
```

**Variations:**
- **Pacific Atlantic Water Flow**: reverse-thinking — BFS/DFS FROM both oceans inward (water flows downhill, so reverse and flow uphill from the ocean).
- **Surrounded Regions**: DFS from border 'O's first to mark "safe", then flip all unmarked 'O's to 'X'.
- **As Far from Land as Possible**: multi-source BFS from all land cells.

**Common bugs:**
- Forgetting boundary checks before array access → `ArrayIndexOutOfBoundsException`.
- 4-directional vs 8-directional confusion — read the problem carefully, this changes the `dx/dy` arrays.

---

## Suggested learning order if starting from scratch

1. **BFS + DFS Traversal basics (Number of Islands, generic shortest path)** — foundational; internalize "visited tracking + queue/stack/recursion".
2. **Multi-source BFS (Rotting Oranges)** — small twist on BFS, very common interview pattern.
3. **Union-Find (Number of Provinces)** — learn path compression + union by size; reused everywhere.
4. **Cycle Detection — Undirected (Redundant Connection)** — direct application of Union-Find.
5. **Topological Sort (Course Schedule II)** — Kahn's algorithm; foundational for dependency problems.
6. **Cycle Detection — Directed (Course Schedule)** — 3-color DFS; subtly different from undirected version.
7. **Bipartite Check (Is Graph Bipartite?)** — simple 2-coloring BFS/DFS.
8. **Dijkstra's (Network Delay Time)** — first weighted-graph algorithm; learn the PQ + lazy-deletion pattern.
9. **Bellman-Ford (Cheapest Flights Within K Stops)** — learn why negative edges / step-constraints break Dijkstra.
10. **Floyd-Warshall (Find the City...)** — all-pairs; simple triple loop once you see the DP framing.
11. **MST — Kruskal's then Prim's (Min Cost to Connect All Points)** — builds directly on Union-Find + PQ skills.
12. **BFS with Extra State (Shortest Path to Get All Keys)** — the "visited isn't just the node" leap; trips up most people first time.
13. **Bridges/Articulation Points (Critical Connections)** — Tarjan's; more advanced, lower frequency but high-signal if asked.
14. **Strongly Connected Components** — most advanced; lowest frequency in SDE2-level interviews, good to know exists.

**Universal traps across every Graph problem:**
- **Directed vs undirected confusion** — determines whether you add edges one-way or both-ways in the adjacency list. Misreading this silently breaks everything downstream.
- **Mark visited at ENQUEUE time (BFS), not at DEQUEUE time** — the single most common BFS bug; causes duplicate processing and wrong distances.
- **Visited set must capture the FULL state**, not just the node, whenever there's extra constraint (keys, parity, remaining moves) — see pattern #13.
- **Disconnected graphs** — always loop over ALL nodes as potential start points unless the problem guarantees connectivity; a single BFS/DFS from node 0 silently misses unreachable components.
- **Dijkstra fails silently on negative weights** — no crash, just wrong answers. If you see "negative weight" anywhere, switch to Bellman-Ford immediately.
- **Cycle detection: 2-color is NOT enough for directed graphs** — need 3 states (white/gray/black) to distinguish "back edge" (cycle) from "cross edge to already-finished branch" (no cycle).
- **Off-by-one with "K stops" vs "K edges"** — K stops = K+1 edges in the path. Extremely common Bellman-Ford bug.
- **Integer overflow in Floyd-Warshall** — `MAX_VALUE + MAX_VALUE` overflows to negative; use `MAX_VALUE / 2` as infinity sentinel.
- **Forgetting path compression / union by size in Union-Find** — correctness holds without them, but performance degrades from near-O(1) to O(n) per operation.
- **Recursive DFS stack overflow** on large grids/graphs (10^4+ nodes) — switch to iterative DFS or BFS when input size is large.
