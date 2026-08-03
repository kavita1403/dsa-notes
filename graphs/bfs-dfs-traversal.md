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

## 2. Number of Connected Components in an Undirected Graph — LC 323

**Link:** https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/

**Problem (1-liner):** Given `n` nodes labelled `0..n-1` and an **edge list** `edges[i] = [u, v]`, return the number of connected components.

### What tripped me up
- Wasn't sure whether I actually needed to build an adjacency list — the edges are already right there in the input, felt redundant.
- Once I built the adjacency list, DFS was easy: visit every neighbour, mark visited, done.

### Key insight — YES, you do want the adjacency list
The input is an **edge list**, not an adjacency structure. If you skip building the adjacency list and try to DFS directly, then to find the neighbours of node `u` you'd have to scan the entire `edges` array every time — that's O(E) per lookup, O(V · E) total. Ugly.

Build the adjacency list **once** in O(E), then every neighbour lookup is O(1) amortised. That's the whole trade: **one linear pre-pass buys you fast neighbour access forever after.**

Compare with LC 547: input was already a **matrix**, so neighbours = "scan row `i`" was built in — no adjacency list construction needed. Here it isn't, so you build it yourself.

**Undirected → add both directions:** for every edge `[u, v]`, do `adj[u].add(v)` AND `adj[v].add(u)`. Forgetting the second one is the single most common bug in this problem.

### Approach 1 — DFS with adjacency list
```java
void dfs(int node, List<List<Integer>> adj, boolean[] visited) {
    visited[node] = true;
    for (int nei : adj.get(node)) {
        if (!visited[nei]) dfs(nei, adj, visited);
    }
}

int countComponents(int n, int[][] edges) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (int[] e : edges) {
        adj.get(e[0]).add(e[1]);
        adj.get(e[1]).add(e[0]);   // undirected — BOTH ways
    }

    boolean[] visited = new boolean[n];
    int count = 0;
    for (int i = 0; i < n; i++) {
        if (!visited[i]) { dfs(i, adj, visited); count++; }
    }
    return count;
}
```
- **Time:** O(V + E). Build adjacency list = O(E). DFS visits each node once (O(V)) and each edge twice (once from each endpoint) = O(2E) = O(E). Total = O(V + E).
- **Space:** O(V + E) — adjacency list holds 2E entries + O(V) visited + O(V) recursion stack worst case.

**Why the `V` in `O(V + E)` matters:** even if `E = 0` (all isolated nodes), the outer `for i in 0..n-1` still runs, so you can never be faster than O(V). And you need the V to actually *count* isolated components.

### Approach 2 — BFS with adjacency list
```java
int countComponents(int n, int[][] edges) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (int[] e : edges) {
        adj.get(e[0]).add(e[1]);
        adj.get(e[1]).add(e[0]);
    }

    boolean[] visited = new boolean[n];
    int count = 0;
    for (int i = 0; i < n; i++) {
        if (visited[i]) continue;
        count++;
        Queue<Integer> q = new ArrayDeque<>();
        q.offer(i);
        visited[i] = true;             // mark on ENQUEUE, not on poll
        while (!q.isEmpty()) {
            int node = q.poll();
            for (int nei : adj.get(node)) {
                if (!visited[nei]) {
                    visited[nei] = true;
                    q.offer(nei);
                }
            }
        }
    }
    return count;
}
```
- **Time:** O(V + E) — same reasoning as DFS.
- **Space:** O(V + E) adjacency list + O(V) queue worst case + O(V) visited. No recursion stack, so slightly friendlier for very deep graphs.

### Approach 3 — Union-Find (DSU) — arguably the cleanest fit here
Since the input is literally a list of edges, Union-Find feels *natural* — you don't even need to build an adjacency list.

**Trick:** start `count = n` (every node is its own component). For each edge, try to `union` the two endpoints. If they were in **different** components (union actually merged something), decrement `count`. If they were already connected, do nothing. Final `count` is the answer.

```java
class DSU {
    int[] parent, rank;
    DSU(int n) {
        parent = new int[n]; rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);   // path compression
        return parent[x];
    }
    boolean union(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;                        // already connected
        if (rank[ra] < rank[rb]) { parent[ra] = rb; }
        else if (rank[ra] > rank[rb]) { parent[rb] = ra; }
        else { parent[rb] = ra; rank[ra]++; }
        return true;
    }
}

int countComponents(int n, int[][] edges) {
    DSU dsu = new DSU(n);
    int count = n;
    for (int[] e : edges) {
        if (dsu.union(e[0], e[1])) count--;
    }
    return count;
}
```
- **Time:** O(V + E · α(V)) where α is the inverse Ackermann function (effectively constant, `< 5` for any realistic input). Initialising `parent` is O(V); each of the E unions is near-constant.
- **Space:** O(V) for `parent` and `rank`. **No adjacency list needed** — that's the win.

### When to pick which
| Approach     | Best when                                            | Downside                              |
|--------------|------------------------------------------------------|---------------------------------------|
| DFS          | Default choice, cleanest recursive code              | Recursion stack on deep graphs        |
| BFS          | Very deep graphs (avoid stack overflow)              | Slightly more boilerplate (queue)     |
| Union-Find   | Input is edge list, or connectivity queries stream in | Have to remember the DSU boilerplate  |

For LC 323 specifically, **Union-Find is often the shortest correct answer** because the input shape matches DSU's API 1:1.

### Next-time checklist
1. **Input is edge list?** Build adjacency list up front — one O(E) pass, then O(1) neighbour lookups forever.
2. **Undirected graph?** Add BOTH `adj[u].add(v)` AND `adj[v].add(u)`. Miss one → half your graph disappears.
3. **Isolated nodes count as components.** The outer `for i in 0..n-1` handles them for free — don't forget it.
4. **Mark visited on enqueue / before recursing.** Same rule as LC 547.
5. **Edge-list problems love Union-Find.** If the problem also has "tell me connectivity after each new edge", DSU is basically mandatory.

### Gotchas I missed
- **Duplicate edges** are allowed in some graph problems. DFS/BFS handle them for free via `visited`. Union-Find handles them via the "already same root → skip" branch. Nothing to do — just be aware.
- **Self-loops** (`edge = [u, u]`) — DFS ignores them (already visited), Union-Find's `find(u) == find(u)` returns false, no decrement. Also harmless.
- **Path compression + union by rank** together give near-O(1) per op. Skipping either drops you to O(log V) per op — still fast, but not the textbook complexity. Mention both in interviews.
- Complexity phrasing: it's **O(V + E)** for DFS/BFS, not O(E). Forgetting the V loses points when the graph has lots of isolated nodes.
- Small correction to what you said: "once I build the adjacency list, DFS is easy" — 100% true, but the *reason* it becomes easy is that you've converted the problem from LC 323 (edge list) into LC 547 shape (per-node neighbour lookup). That mental step — **"normalise the input into the shape my algorithm expects"** — is the reusable pattern here.

---
