# BFS / DFS Traversal — Revision Notes

Quick-glance notes for graph traversal problems. Each entry captures what tripped me up, the key insight, approaches with complexity, and a pre-flight checklist for next time.

> **Tip:** Code blocks are collapsed by default (click the ▸ arrow to expand). Keeps the notes skim-friendly; deep-dive when you need to.

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

<details>
<summary><b>Show DFS code (Java)</b></summary>

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

</details>

- **Time:** O(N²) — every matrix cell inspected once.
- **Space:** O(N) visited + O(N) recursion stack worst case.

### Approach 2 — BFS
Same structure; replace recursion with a queue. Enqueue `i`, pop, scan its row for `1`s, enqueue unvisited neighbours, mark visited on enqueue (not on pop) to avoid duplicates.

<details>
<summary><b>Show BFS code (Java)</b></summary>

```java
int findCircleNum(int[][] adj) {
    int n = adj.length, count = 0;
    boolean[] visited = new boolean[n];

    for (int i = 0; i < n; i++) {
        if (visited[i]) continue;
        count++;
        Queue<Integer> q = new ArrayDeque<>();
        q.offer(i);
        visited[i] = true;                 // mark on ENQUEUE

        while (!q.isEmpty()) {
            int node = q.poll();
            for (int j = 0; j < n; j++) {
                if (adj[node][j] == 1 && !visited[j]) {
                    visited[j] = true;     // mark BEFORE enqueue → no dupes
                    q.offer(j);
                }
            }
        }
    }
    return count;
}
```

</details>

- **Time:** O(N²) — same row-scan per node.
- **Space:** O(N) visited + O(N) queue worst case.

### Approach 3 — Union-Find (DSU)
- Initialise `parent[i] = i` for all `i`.
- For every pair `i, j` with `j > i` (upper triangle only — matrix is symmetric) where `adj[i][j] == 1`, `union(i, j)`.
- Count distinct roots at the end → that's the province count.

<details>
<summary><b>Show Union-Find code (Java)</b></summary>

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
        if (rank[ra] < rank[rb]) parent[ra] = rb;
        else if (rank[ra] > rank[rb]) parent[rb] = ra;
        else { parent[rb] = ra; rank[ra]++; }
        return true;
    }
}

int findCircleNum(int[][] adj) {
    int n = adj.length;
    DSU dsu = new DSU(n);
    int count = n;                                         // start: every node its own province

    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {                  // upper triangle only
            if (adj[i][j] == 1 && dsu.union(i, j)) count--;
        }
    }
    return count;
}
```

</details>

- **Time:** O(N² · α(N)) — scan upper triangle (N²/2 cells) + near-constant union per edge.
- **Space:** O(N) for `parent` and `rank`.

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

<details>
<summary><b>Show DFS code (Java)</b></summary>

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

</details>

- **Time:** O(V + E). Build adjacency list = O(E). DFS visits each node once (O(V)) and each edge twice (once from each endpoint) = O(2E) = O(E). Total = O(V + E).
- **Space:** O(V + E) — adjacency list holds 2E entries + O(V) visited + O(V) recursion stack worst case.

**Why the `V` in `O(V + E)` matters:** even if `E = 0` (all isolated nodes), the outer `for i in 0..n-1` still runs, so you can never be faster than O(V). And you need the V to actually *count* isolated components.

### Approach 2 — BFS with adjacency list

<details>
<summary><b>Show BFS code (Java)</b></summary>

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

</details>

- **Time:** O(V + E) — same reasoning as DFS.
- **Space:** O(V + E) adjacency list + O(V) queue worst case + O(V) visited. No recursion stack, so slightly friendlier for very deep graphs.

### Approach 3 — Union-Find (DSU) — arguably the cleanest fit here
Since the input is literally a list of edges, Union-Find feels *natural* — you don't even need to build an adjacency list.

**Trick:** start `count = n` (every node is its own component). For each edge, try to `union` the two endpoints. If they were in **different** components (union actually merged something), decrement `count`. If they were already connected, do nothing. Final `count` is the answer.

<details>
<summary><b>Show Union-Find code (Java)</b></summary>

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

</details>

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

## 3. Keys and Rooms — LC 841

**Link:** https://leetcode.com/problems/keys-and-rooms/

**Problem (1-liner):** `n` rooms labelled `0..n-1`, all locked except room `0`. `rooms[i]` is the list of keys found inside room `i` (each key unlocks the room with that number). Starting from room `0`, return `true` if you can visit **every** room.

### What tripped me up
- Nothing much — I realised straight away that `rooms` **is** the adjacency list. `rooms[i]` = neighbours of node `i`. No conversion needed.
- Just DFS/BFS from node `0`, mark visited, at the end check if every node got visited.

### Key insight — the input IS the adjacency list
This problem is the third flavour in the input-shape trilogy:

| Problem | Input given | Do you build the adjacency list yourself? |
|---------|-------------|-------------------------------------------|
| LC 547 Provinces | `int[][]` adjacency **matrix** | No — scan row `i` for neighbours |
| LC 323 Components | **edge list** `[[u,v], ...]` | **Yes** — one O(E) pre-pass |
| LC 841 Keys and Rooms | `List<List<Integer>>` **adjacency list** | No — just index into it |

So the "hard part" of graph problems (getting to fast neighbour lookup) is already done for you. Whatever effort you saved goes into two things:
1. **Choosing a start node** — problem tells you: room `0`.
2. **Checking the answer condition** — did we visit all `n` rooms? Track a counter or check `visited[]` at the end.

### One critical thing to notice: the graph is DIRECTED
Having a key to room `j` in room `i` lets you go `i → j`. It does **not** automatically let you go `j → i` (room `j` might not contain a key for room `i`).

This directedness matters when you consider Union-Find (see below).

### Approach 1 — DFS

<details>
<summary><b>Show DFS code (Java)</b></summary>

```java
void dfs(int room, List<List<Integer>> rooms, boolean[] visited) {
    visited[room] = true;
    for (int key : rooms.get(room)) {
        if (!visited[key]) dfs(key, rooms, visited);
    }
}

boolean canVisitAllRooms(List<List<Integer>> rooms) {
    int n = rooms.size();
    boolean[] visited = new boolean[n];
    dfs(0, rooms, visited);
    for (boolean v : visited) if (!v) return false;
    return true;
}
```

</details>

- **Time:** O(V + E) — each room visited once (V), each key inspected once across all rooms (total keys = E).
- **Space:** O(V) visited + O(V) recursion stack worst case. No extra adjacency structure needed — `rooms` **is** the structure.

### Approach 2 — BFS

<details>
<summary><b>Show BFS code (Java)</b></summary>

```java
boolean canVisitAllRooms(List<List<Integer>> rooms) {
    int n = rooms.size();
    boolean[] visited = new boolean[n];
    int visitedCount = 0;

    Queue<Integer> q = new ArrayDeque<>();
    q.offer(0);
    visited[0] = true;
    visitedCount++;

    while (!q.isEmpty()) {
        int room = q.poll();
        for (int key : rooms.get(room)) {
            if (!visited[key]) {
                visited[key] = true;   // mark on ENQUEUE
                visitedCount++;
                q.offer(key);
            }
        }
    }
    return visitedCount == n;
}
```

</details>

- **Time:** O(V + E) — same reasoning as DFS.
- **Space:** O(V) visited + O(V) queue worst case. `visitedCount` avoids the final scan of `visited[]`.

### Approach 3 — Union-Find? Nope, wrong tool here

Union-Find works on **undirected** connectivity — its `union(a, b)` is symmetric: once you merge `a` and `b`, they are indistinguishable, and any future query treats them as mutually reachable.

But this graph is **directed**. If room `0` has a key to room `1`, but room `1` has no keys at all, DSU would happily union `{0, 1}` — but you actually **cannot** reach `0` from `1`. DSU has no way to encode "reachable from 0, but not necessarily back".

Lesson: **DSU is for undirected connectivity, not directed reachability.** For directed reachability from a source, BFS/DFS is the correct tool. (For strongly-connected components in directed graphs you would reach for Kosaraju or Tarjan, not DSU.)

### When to pick which (for LC 841 specifically)
| Approach     | Verdict for LC 841                                     |
|--------------|--------------------------------------------------------|
| DFS          | **Default pick.** Shortest code, matches the problem shape. |
| BFS          | Equally valid. Slightly better on very deep chains (no recursion). |
| Union-Find   | **Wrong tool.** Graph is directed; DSU would give false positives. |

### Next-time checklist
1. **Look at the input type before anything else.** `List<List<Integer>>` = adjacency list already built. `int[][]` with square dims = matrix. `int[][]` of pairs = edge list.
2. **Directed or undirected?** Look at how the problem describes edges. "Has a key to" is one-way → directed. "Is friends with" / "is connected to" is usually two-way → undirected.
3. **Start node:** problem-specified here (room `0`). If not specified, you probably need an outer loop over all nodes (like LC 547 / 323).
4. **Answer condition:** "visit all" → track a counter or verify `visited[]` at the end. Return early with `visitedCount == n` — cleaner than a post-scan.
5. **DSU red flag:** if the graph is directed OR the problem asks about reachability from a specific source, do NOT reach for Union-Find.

### Gotchas I missed
- **Duplicate keys** inside the same room (`rooms[i]` can contain the same key twice, or a key you already have). `visited[]` handles both for free — no special case needed.
- **Self-key** (`rooms[i]` contains `i`) — harmless; `visited[i]` is already `true`, so the recursive/enqueue branch skips.
- **Room 0 comes pre-unlocked** — that is why the DFS/BFS starts there without needing a key. Do not accidentally require a key for room 0.
- **Answer type is `boolean`, not a count.** Easy misread under pressure — do not return `visitedCount` by accident.
- Minor phrasing correction: it is not really "just DFS/BFS traversal" that solves this — it is DFS/BFS **plus a reachability check** (did we hit every node?). The traversal is the mechanism; the check is what the problem is actually asking. Two-part answer.

---

## 4. All Paths From Source to Target — LC 797

**Link:** https://leetcode.com/problems/all-paths-from-source-to-target/

**Problem (1-liner):** Given a **DAG** with `n` nodes labelled `0..n-1`, where `graph[i]` is the adjacency list of nodes reachable from `i`, return **all possible paths** from node `0` to node `n-1`.

### What tripped me up
- DFS felt natural — add current node to a temp path, recurse on neighbours, when I hit `n-1` push a **snapshot** of temp into the answer, then **backtrack** (pop the last node).
- BFS was where I got stuck. I could not figure out how to keep each path separate as it evolved. Looked at the solution: **each queue element is a whole path, not just a node.** For every neighbour, make a **copy** of the current path, append the neighbour, and enqueue. That is the whole trick. Obvious in hindsight.

### Key insight — this is DAG enumeration, not connectivity
Most graph problems so far (547, 323, 841) asked *counting* or *reachability* questions. This one asks for **all paths** — an enumeration problem. That single word changes the complexity class from polynomial to **exponential**, and it changes which data structure you carry around during traversal.

- **Counting/reachability →** carry `visited[]` and a counter. One bit per node.
- **Enumeration →** carry the **current path** (mutable list for DFS, immutable snapshot for BFS).

**No `visited[]` needed here.** The graph is a DAG, so there are no cycles → you cannot loop forever. Adding `visited[]` would actually be **wrong**: different paths from `0` to `n-1` often share intermediate nodes (`0 → 2 → 3` and `0 → 1 → 2 → 3` both use node `2`), and a global `visited[]` would block the second path from ever forming.

### Approach 1 — DFS + backtracking (default choice)

<details>
<summary><b>Show DFS code (Java)</b></summary>

```java
void dfs(int node, int target, int[][] graph,
         List<Integer> path, List<List<Integer>> result) {
    path.add(node);                              // choose
    if (node == target) {
        result.add(new ArrayList<>(path));       // SNAPSHOT — must copy!
    } else {
        for (int nei : graph[node]) {
            dfs(nei, target, graph, path, result);
        }
    }
    path.remove(path.size() - 1);                // un-choose (backtrack)
}

List<List<Integer>> allPathsSourceTarget(int[][] graph) {
    List<List<Integer>> result = new ArrayList<>();
    dfs(0, graph.length - 1, graph, new ArrayList<>(), result);
    return result;
}
```

</details>

**Why the snapshot?** `path` is a single mutable list that gets reused across the whole recursion. If you push `path` itself into `result`, every entry in `result` ends up pointing to the same (eventually empty) list. **Always `new ArrayList<>(path)`** when saving.

### Approach 2 — BFS with path-per-queue-entry

<details>
<summary><b>Show BFS code (Java)</b></summary>

```java
List<List<Integer>> allPathsSourceTarget(int[][] graph) {
    int target = graph.length - 1;
    List<List<Integer>> result = new ArrayList<>();

    Queue<List<Integer>> q = new ArrayDeque<>();
    List<Integer> start = new ArrayList<>();
    start.add(0);
    q.offer(start);

    while (!q.isEmpty()) {
        List<Integer> path = q.poll();
        int last = path.get(path.size() - 1);

        if (last == target) {
            result.add(path);                    // already a fresh list — no copy needed
            continue;
        }
        for (int nei : graph[last]) {
            List<Integer> next = new ArrayList<>(path);   // COPY on branch
            next.add(nei);
            q.offer(next);
        }
    }
    return result;
}
```

</details>

**Mental model:** the queue does not hold "nodes to visit next" — it holds **partial paths in progress**. Each pop is one path being extended. When a path reaches the target, it goes straight into `result`.

### Time Complexity — the honest breakdown (this is the part I want to nail)

Both DFS and BFS have the **same** big-O here. Let me walk through why step by step, because "all paths" TC is not obvious.

**Step 1 — How many paths can there be in the worst case?**

Worst case is a **complete DAG**: every node `i` has an edge to every node `j > i`. In this graph, a path from `0` to `n-1` is defined by *which subset of the intermediate nodes* `{1, 2, ..., n-2}` you pass through, in order. Any subset works because the DAG is complete.

Number of subsets of an `(n-2)`-element set = **2^(n-2) ≈ 2^n**.

So the **number of paths** is O(2^n) in the worst case. This is why enumeration problems blow up.

**Step 2 — How much work per path?**

Each path has up to `n` nodes. For DFS, adding to `path` is O(1), but the **snapshot copy** at the target is O(n). For BFS, each `new ArrayList<>(path)` inside the neighbour loop is O(path length) = O(n).

So per path: **O(n) work**.

**Step 3 — Multiply.**

**Time = O(paths × work per path) = O(2^n × n) = O(n · 2^n).**

That is the answer. Both DFS and BFS hit it.

### Space Complexity

- **Output size** dominates: `O(n · 2^n)` — up to `2^n` paths, each up to length `n`. This is unavoidable; the problem asks for the answer.
- **DFS auxiliary space** (excluding output): O(n) for the recursion stack + O(n) for the current `path`. So **O(n) extra**.
- **BFS auxiliary space** (excluding output): the queue can hold many partial paths at once. In the worst case this is `O(n · 2^n)` too — much worse than DFS. This is why **DFS + backtracking is the standard tool for enumeration**: it emits paths and reclaims memory as it backtracks; BFS keeps all in-flight paths alive.

### DFS vs BFS for enumeration — quick rule
| Metric | DFS + backtracking | BFS with path-copy |
|--------|--------------------|--------------------|
| Time | O(n · 2^n) | O(n · 2^n) |
| Extra space | **O(n)** | O(n · 2^n) |
| Code length | Shortest | Chattier (queue + copy) |
| When to use | **Default** for "return all paths / subsets / combinations" | Rare — only if you need paths in shortest-first order |

For LeetCode enumeration problems, reach for DFS + backtracking by default. BFS shines for **shortest-path** questions, not all-paths.

### Next-time checklist
1. **"Return all paths / subsets / combinations" → enumeration.** Complexity is exponential in the worst case. Do not panic when you see 2^n show up.
2. **DAG?** Skip `visited[]` — cycles are impossible, and `visited[]` would incorrectly block shared intermediate nodes.
3. **DFS backtracking template:**
   - `path.add(node)` before recursing
   - `if (done) result.add(new ArrayList<>(path))` — **snapshot, always copy**
   - `path.remove(last)` after recursing
4. **BFS enumeration template:** queue holds **paths**, not nodes. Copy path on every branch.
5. **Default to DFS** for all-paths. Same time, way better space.

### Gotchas I missed
- **The snapshot copy is not optional.** Pushing `path` directly instead of `new ArrayList<>(path)` is the #1 bug on this problem. All entries in `result` end up pointing to the same object.
- **Do NOT add `visited[]` reflexively.** Works on Provinces / Components / Keys and Rooms — breaks here. The rule is: `visited[]` prevents re-*visiting*; backtracking wants to re-visit nodes on different paths.
- **Backtracking pop is unconditional** — always runs, whether or not we hit the target. Putting it inside the `else` branch (skipping the pop on target) leaves stale entries and corrupts subsequent paths.
- Correction to how I initially thought about it: BFS is not "harder" than DFS here — it is just **carrying more state per queue entry** than the counting/reachability problems taught us. Once you accept "queue element = a whole partial path", the code writes itself.
- Small terminology: this problem is DAG-enumeration. The complexity class here is the same as generating all subsets — that is your mental yardstick. Any time you see "return all X" in a graph/tree problem, expect `2^n` or `n!` in your TC.

---

## 5. Clone Graph — LC 133

**Link:** https://leetcode.com/problems/clone-graph/

**Problem (1-liner):** Given a reference to a node in a **connected, undirected** graph (each node has an `int val` and a `List<Node> neighbors`), return a **deep copy** of the entire graph.

### What tripped me up
- Did not think of using a map. Could not figure out how to make sure that when node `A` and node `B` both point to node `C`, the *cloned* `A` and `B` end up pointing to the **same** cloned `C` (not two separate copies).
- Struggled with the idea that DFS needs to **return** the clone up to its parent. I assumed the wiring would happen automatically. It does not — the `return newNode;` is what actually connects the parent clone to the child clone.

### Key insight — this is graph CONSTRUCTION, not traversal-for-answer
Every previous problem (547, 323, 841, 797) used DFS/BFS to **compute** something (count, boolean, list of paths) while walking the graph. Here, the walk itself is a means to an end — the real job is **building a mirror graph in parallel**.

Two things happen at every DFS call:
1. **Traverse:** visit this node.
2. **Construct:** create the clone, and wire the parents clone to this one via the return value.

That second job is why the recursion has to return the newly-built clone up the call stack. In pure traversal problems, DFS could return `void`. Here it cannot.

### The map does DOUBLE duty
This is the piece I want future-me to internalise. The `Map<Node, Node>` (original → clone) is not just a caching optimisation — it is doing **two independent jobs at once**:

1. **Acts as `visited[]`** — this graph is **undirected and can have cycles**, so without a memo you would recurse forever. The `if (mp.containsKey(node)) return mp.get(node);` line is literally the cycle-breaker.
2. **Acts as the "original → clone" lookup** — when a later DFS branch enters a node that was already cloned (because two originals share a neighbour), we must return **the same clone instance**, not build a new one. That is what makes `A` and `B` both point to the same `C`.

One data structure, two responsibilities. Miss either interpretation and the code either loops forever or produces a mangled duplicate-riddled clone.

### The `mp.put(node, newNode)` MUST come BEFORE recursing
Order matters and this is the second-most-common bug:

```java
Node newNode = new Node(node.val);
mp.put(node, newNode);                              // put FIRST
for (Node nei : node.neighbors) {
    newNode.neighbors.add(dfs(mp, nei));            // THEN recurse
}
```

If you swap the order (recurse first, then put), the cycle-break in the recursive call never sees your in-progress node → infinite recursion → stack overflow. Insert the clone into the map **the moment you create it**, before you touch any neighbours.

### Approach 1 — DFS

<details>
<summary><b>Show DFS code (Java)</b></summary>

```java
Node dfs(Map<Node, Node> mp, Node node) {
    if (node == null) return null;                  // defensive: empty graph
    if (mp.containsKey(node)) return mp.get(node); // cache hit → return existing clone

    Node newNode = new Node(node.val);
    mp.put(node, newNode);                          // PUT BEFORE recursing (cycle break)

    for (Node nei : node.neighbors) {
        newNode.neighbors.add(dfs(mp, nei));        // return value = the wired-up clone
    }
    return newNode;                                 // hand this clone back to the caller
}

public Node cloneGraph(Node node) {
    return dfs(new HashMap<>(), node);
}
```

</details>

**Why `return newNode` is not optional:** the caller (parent frame) is executing `newNode.neighbors.add(dfs(mp, nei));`. It is literally consuming the return value to build its own neighbour list. Without the return, `add(null)` would run and no edges would exist in the clone.

### Approach 2 — BFS (map still does double duty)

BFS makes the two jobs of the map more visible because construction and traversal are separated in code rather than intertwined in recursion.

<details>
<summary><b>Show BFS code (Java)</b></summary>

```java
public Node cloneGraph(Node node) {
    if (node == null) return null;

    Map<Node, Node> mp = new HashMap<>();
    mp.put(node, new Node(node.val));               // clone the start node up front

    Queue<Node> q = new ArrayDeque<>();
    q.offer(node);

    while (!q.isEmpty()) {
        Node curr = q.poll();
        for (Node nei : curr.neighbors) {
            if (!mp.containsKey(nei)) {
                mp.put(nei, new Node(nei.val));    // create clone the FIRST time we see nei
                q.offer(nei);                       // and only then enqueue for exploration
            }
            mp.get(curr).neighbors.add(mp.get(nei)); // wire clone-of-curr to clone-of-nei
        }
    }
    return mp.get(node);
}
```

</details>

**Why this is clean:** every neighbour edge is wired exactly once by looking both endpoints up in the map. The maps "have I cloned this yet?" role is what stops us from enqueueing the same node twice (which is BFSs cycle break).

### Time & Space Complexity

**Time = O(V + E).**
- Every node is created and inserted into the map exactly once → O(V).
- Every edge is traversed exactly twice in an undirected graph (once from each endpoint) → O(2E) = O(E).
- Map `put`/`get` are O(1) amortised.
- Total: **O(V + E)**.

**Space = O(V + E) for the cloned graph itself + O(V) for the map + O(V) recursion stack (DFS) or O(V) queue (BFS).**
- The cloned graph is the output → unavoidable O(V + E).
- Auxiliary space (excluding output) = **O(V)**.

### DFS vs BFS for this problem
| Aspect | DFS | BFS |
|--------|-----|-----|
| Code length | Shorter (recursion carries the wiring) | A bit longer (explicit queue + neighbour wiring) |
| Wiring style | Via return value up the call stack | Via two map lookups per edge |
| Deep graphs | Risk of stack overflow | Safe (heap-allocated queue) |
| Default pick | Yes — this is textbook DFS territory | Fine, and clearer if you are learning the pattern |

### Next-time checklist
1. **"Deep copy / clone a graph" → build a `Map<Node, Node>`.** No exceptions.
2. **Put the clone in the map BEFORE recursing into neighbours.** Otherwise cycles blow the stack.
3. **DFS returns the clone** so the caller can wire it into the parents neighbour list. This is why the recursion is not `void`.
4. **BFS wires edges via two map lookups** — `mp.get(curr).neighbors.add(mp.get(nei))`. Enqueue only on first-sight of a neighbour.
5. **Recognise the pattern:** "traverse a structure + build a parallel structure alongside" also shows up in copy-tree-with-random-pointer, deep-clone linked list, tree-to-graph conversions, etc. Same map trick every time.

### Gotchas I missed
- **Map does two jobs — do not conflate them but do not separate them either.** It is *simultaneously* the visited-set (cycle break) AND the original→clone lookup (edge-sharing correctness). One HashMap covers both. If you tried to use `Set<Node>` for visited and a separate map for cloning, you would just be doing extra work.
- **Undirected graph, so every edge shows up twice.** Wiring `A → C` when processing `A`, and `C → A` when processing `C`, happens naturally — no special case, no de-duplication needed, because you follow the inputs neighbour lists verbatim.
- **`return newNode` is a wire, not a formality.** Correction to my initial thinking: I assumed recursion would "just handle" the connections. It does not. Every DFS frame produces exactly one clone, and its return value is the edge the parent uses to hook it in. Pure side-effect DFS (like LC 547/323/841) is a different beast.
- **Contrast with LC 797 (All Paths):** there we said "no visited[] needed, DAG." Here we say "map IS the visited set, needed because of cycles." Same word (visited), different rule — driven by whether cycles exist. Always check: is this graph cyclic?
- Small correction to my mental model: the "recursion connects the graph automatically" instinct is not wrong for a **traversal** problem (where you are just walking) — it is wrong for a **construction** problem (where you are building new nodes). The recursive return value is the physical wire.

---

## 6. Minimum Number of Vertices to Reach All Nodes — LC 1557

**Link:** https://leetcode.com/problems/minimum-number-of-vertices-to-reach-all-nodes/

**Problem (1-liner):** Given a **DAG** with `n` nodes and an edge list `edges[i] = [from, to]`, return the **smallest set of vertices** such that every node in the graph is reachable from at least one vertex in the set.

### What tripped me up
- At first I thought this was a traversal / connected-components problem — pick a start, DFS/BFS, count how many launches. It is not.
- The unlock: **the answer is exactly the set of nodes with `in-degree == 0`.** No DFS, no BFS, no adjacency list even needed.
- Learned two new concepts: **in-degree / out-degree**, and how they combined with the **DAG** property turn this into a one-pass linear scan.

### New concepts I picked up
- **In-degree of node `v`** = number of edges *pointing INTO* `v`. In an edge list, that is the number of edges where `v` is the second element (`edge[1]`).
- **Out-degree of node `v`** = number of edges *going OUT of* `v`. Number of edges where `v` is the first element (`edge[0]`).
- For an undirected edge, both endpoints get +1 degree (no direction). For a directed edge `u -> v`, `u` gets +1 out-degree, `v` gets +1 in-degree.
- **DAG = Directed Acyclic Graph** — directed edges, no cycles. That "no cycles" part is what makes this trick work (see below).

### Key insight — why `in-degree == 0` is the whole answer

Split every node into two buckets:

- **`in-degree == 0`:** no edge points to this node → **no other node can reach it** → we MUST include it in our answer.
- **`in-degree >= 1`:** some node `u` points to this node → we can reach it from `u`. And `u` is itself either in-degree 0 (already in our answer) or reachable from some ancestor. Because the graph is a DAG (no cycles), walking backwards from any node eventually reaches an in-degree-0 root. So every in-degree-`>=1` node is **transitively reachable** from the in-degree-0 set.

That covers "sufficient" (every node is reachable from an in-degree-0 node) AND "necessary" (in-degree-0 nodes cannot be reached from anywhere else, so we cannot skip any of them). Answer = set of nodes with in-degree 0. Nothing to optimise, nothing to explore.

### The DAG property is doing heavy lifting
If cycles were allowed, this trick breaks. Example: `A -> B -> C -> A`. All three have in-degree 1, so the "in-degree 0" set would be **empty** — but you obviously still need one vertex (any of the three) to cover the cycle. In a general directed graph you would need something more sophisticated (find strongly connected components, contract each SCC into a super-node, then apply the in-degree trick on the DAG of SCCs).

**So: the "answer = in-degree 0 nodes" rule is only correct because the problem promises a DAG.** Always check the problem statement for the DAG guarantee before reaching for this trick.

### Approach — one pass over edges

<details>
<summary><b>Show code (Java)</b></summary>

```java
public List<Integer> findSmallestSetOfVertices(int n, List<List<Integer>> edges) {
    boolean[] hasIncoming = new boolean[n];
    for (List<Integer> e : edges) {
        hasIncoming[e.get(1)] = true;      // e.get(1) is the destination
    }

    List<Integer> result = new ArrayList<>();
    for (int v = 0; v < n; v++) {
        if (!hasIncoming[v]) result.add(v);
    }
    return result;
}
```

</details>

**Micro-optimisation to notice:** we do not even need an integer in-degree *count* — a single boolean "does anything point at this node?" is enough, since we only care about the "== 0 vs > 0" distinction. If a future problem asks you to remove edges in a specific order (topological sort / Kahn), then you would need the actual counts.

### Time & Space Complexity

**Time = O(V + E).**
- One pass over `edges` to fill `hasIncoming` → O(E).
- One pass over `0..n-1` to collect nodes without incoming edges → O(V).
- Total: **O(V + E)**.

**Space = O(V).**
- `hasIncoming` array: O(V).
- Output list: up to O(V) if every node is a root (edge case: `edges` is empty).
- No adjacency list, no visited set, no queue, no recursion stack. Nothing else.

### Where in-degree shows up elsewhere (worth knowing)
- **Topological sort (Kahns algorithm):** start with all in-degree-0 nodes in a queue, pop one, decrement in-degrees of its out-neighbours, enqueue any that just hit 0. Result is a valid topological order.
- **Course Schedule (LC 207, 210):** cycle detection in directed graphs. If Kahns algorithm cannot drain the queue (some in-degrees never hit 0), there is a cycle.
- **Any "must-start-here" question on a DAG:** in-degree 0 is your friend.

### Next-time checklist
1. **"Reach all nodes with fewest starts" on a DAG →** answer is nodes with in-degree 0. Do not overthink it.
2. **Confirm the graph is a DAG first.** If cycles are possible, this trick is wrong.
3. **Edge list of directed edges + question about starts →** think in-degree. Do not immediately reach for adjacency list + DFS.
4. **`boolean hasIncoming[]` beats `int inDegree[]` when you only need "zero or not".** Save yourself the arithmetic.
5. **Kahn / topological sort family** all sit on the same "start from in-degree 0" foundation. Recognise the pattern.

### Gotchas I missed
- **This is a graph problem with no graph traversal.** Genuinely surprising the first time. The problem is essentially a set-membership question dressed up in graph vocabulary. Building an adjacency list would be pure waste.
- **Isolated nodes (in-degree 0 AND out-degree 0) are still roots.** They must be in the answer because nothing reaches them. The one-boolean approach handles this for free — an isolated node never appears as an edge destination, so `hasIncoming[v]` stays `false`.
- **What about out-degree?** Irrelevant for this problem. The question is "can this node be reached from elsewhere?", which is purely an in-degree question.
- Correction on my mental framing: I initially wanted to "traverse from every candidate root and see what happens." That would work but is O(V·(V+E)) or worse — massively wasteful. The in-degree trick converts the whole thing into a single linear scan by using the DAG property to skip the exploration entirely.
- Vocabulary reminder for future me: **DAG = Directed + Acyclic + (Graph).** Not "directed" alone. The "acyclic" is doing all the load-bearing here; a general directed graph would need SCC contraction first.

---
