# Cycle Detection & Topological Ordering — Revision Notes

Notes for graph problems where the core question is "is there a cycle?" or "give me an order that respects prerequisites?" or "can we 2-color this?". These are all secretly the same family — cycle detection with different framings.

> **Tip:** Code blocks are collapsed by default. Same convention as the other graph notes.

---

## Meta: how to pick the right cycle-detection tool

The right algorithm depends on **two things**: whether the graph is directed or undirected, and what the question is really asking. Here is the decision table.

| Graph type | Question | Best tool | Why |
|------------|----------|-----------|-----|
| **Undirected** | "Is there any cycle?" / "Which edge creates a cycle?" | **Union-Find** | Process edges one by one; if `union` fails (endpoints already connected), you found the cycle-creating edge. O(E · α(V)). |
| **Undirected** | Cycle check but you want to write it recursively | DFS with **parent tracking** | Visited neighbour that is NOT your parent = back edge = cycle. |
| **Directed** | "Can we complete all tasks?" / "Is there a topological order?" | **Kahn's algorithm** (BFS with in-degree) | Process nodes with in-degree 0. If some are left over, cycle exists. |
| **Directed** | Same, but recursive-feeling | DFS with **3 colors** (white / gray / black) | Gray-to-gray edge = back edge = cycle. |
| **Directed rooted tree + 1 extra edge** | "Find the extra edge" | Union-Find + **in-degree check** | Split into three cases: 2-parent, cycle, both. |
| **Any** | "Can we 2-color the graph?" | **BFS/DFS with 2 colors** OR **Union-Find on neighbours** | Bipartite iff no odd cycle. Both approaches are O(V + E). |

**One-line summary:** *Undirected → Union-Find. Directed → Kahn's. 2-color → BFS coloring (or Union-Find if the colors feel unintuitive).*

---

## The three universal recipes (memorise these)

### Recipe A — Union-Find for undirected cycle detection

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
        if (ra == rb) return false;                        // already connected → cycle
        if (rank[ra] < rank[rb]) parent[ra] = rb;
        else if (rank[ra] > rank[rb]) parent[rb] = ra;
        else { parent[rb] = ra; rank[ra]++; }
        return true;
    }
}
```

For every edge `(u, v)` in input order: `if (!dsu.union(u, v)) return that edge as the cycle-creator`.

### Recipe B — Kahn's algorithm for directed cycle detection / topological sort

```java
// Given int n and int[][] edges where edges[i] = [from, to]
int[] indegree = new int[n];
List<List<Integer>> adj = new ArrayList<>();
for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
for (int[] e : edges) {
    adj.get(e[0]).add(e[1]);        // DIRECTED: from → to only, not both ways
    indegree[e[1]]++;
}

Queue<Integer> q = new ArrayDeque<>();
for (int i = 0; i < n; i++) {
    if (indegree[i] == 0) q.offer(i);   // seed ALL zero-in-degree nodes, not just one
}

int processed = 0;
List<Integer> order = new ArrayList<>();
while (!q.isEmpty()) {
    int u = q.poll();
    order.add(u);
    processed++;
    for (int v : adj.get(u)) {
        if (--indegree[v] == 0) q.offer(v);
    }
}

boolean hasCycle = (processed != n);
// order is a valid topological order iff !hasCycle
```

### Recipe C — 2-coloring for bipartite check

```java
// color[i] = 0 (uncoloured), 1, or -1
int[] color = new int[n];
for (int start = 0; start < n; start++) {         // outer loop for DISCONNECTED graph
    if (color[start] != 0) continue;
    Queue<Integer> q = new ArrayDeque<>();
    q.offer(start);
    color[start] = 1;
    while (!q.isEmpty()) {
        int u = q.poll();
        for (int v : adj.get(u)) {
            if (color[v] == 0) {
                color[v] = -color[u];             // opposite colour
                q.offer(v);
            } else if (color[v] == color[u]) {
                return false;                     // same colour on adjacent nodes → odd cycle
            }
        }
    }
}
return true;
```

---

## Common bugs and misconceptions (my personal collection)

### Bug 1 — "I only pushed the first zero-in-degree course into the queue"
This one killed me on Course Schedule. The initial Kahn's queue must contain **every** node with `indegree == 0`, not just the first one you find. If two courses have no prerequisites, both need to seed the queue. Iterate `0..n-1` fully and push all zeros.

### Bug 2 — "I forgot Kahn's doesn't need `visited[]`"
The `indegree` array IS the visited mechanism. A node is enqueued exactly once — at the moment its in-degree hits zero. After that, decrementing its neighbours never brings it back into the queue because we do not touch its own in-degree again. So no `visited[]` needed; adding one is redundant work.

### Bug 3 — "I built an undirected adjacency list for a directed problem"
Rule: **directed problem → add edge ONE way**; **undirected problem → add edge BOTH ways**. Getting this wrong is the single most common bug across graph problems. Always check the problem statement (words like "prerequisite", "flows to", "depends on" = directed; "connected", "friends with", "adjacent" = undirected).

### Bug 4 — "I forgot the graph can be disconnected"
Both bipartite problems (LC 785, 886) test this: not every node is reachable from node 0. If you only BFS from node 0, you miss disconnected pieces and return the wrong answer. Wrap the BFS/DFS in an outer `for (int i = 0; i < n; i++)` loop that starts a new traversal whenever a node is still unvisited (or uncoloured). Same pattern as Number of Provinces (LC 547).

### Bug 5 — "I only reset the visited/color state per-component, but the DSU is global"
When using Union-Find, there is nothing to reset — the DSU spans the whole graph. This is actually a strength: it naturally handles disconnected components without any outer loop tricks.

---

## Time Complexity of cycle-detection problems — how to calculate it

Every problem in this file lands at **O(V + E) time** and **O(V + E) space**. That is not a coincidence — it falls out of the same 3-step reasoning that grid problems use. This section is the framework for deriving TC on any new problem in this family.

### Step 0: identify V and E

| Problem input | V | E |
|---------------|---|---|
| `numCourses` + `prerequisites[][]` (LC 207, 210) | `numCourses` | `prerequisites.length` |
| `int[][] edges` for a tree + 1 (LC 684, 685) | `edges.length` (tree has `n-1` edges so `V = E`) | `edges.length` |
| `int[][] graph` adjacency list (LC 785) | `graph.length` | sum of `graph[i].length` / 2 (undirected → each edge listed twice) |
| `n` + `dislikes[][]` (LC 886) | `n` | `dislikes.length` |

### Step 1: how many times is each edge/node touched?

This is the whole game, same as in grid problems.

| Algorithm | Node touches | Edge touches | Why |
|-----------|--------------|--------------|-----|
| Kahn's (BFS with in-degree) | 1 (enqueued exactly when in-degree hits 0) | 1 (each edge decrements one in-degree, once) | In-degree counter IS the visited mechanism — each node hits zero once, then never again |
| Union-Find single pass | — | 1 per edge (one `union` call) | We iterate the edge stream once, no back-tracking |
| Union-Find + adjacency list (bipartite variant) | 1 per node in outer loop | 1 per neighbour entry (2 per undirected edge) | Same as BFS traversal cost |
| 2-color BFS/DFS | 1 (colour set once, checked on neighbour discovery) | 2 per undirected edge (once from each endpoint) | Standard BFS accounting |

**Key insight:** in every one of these, a node or edge is processed a **small constant** number of times. That constant multiplied by `V` or `E` gives the linear bound.

### Step 2: cost per operation

| Operation | Cost | Notes |
|-----------|------|-------|
| `queue.offer`, `queue.poll` | O(1) | `ArrayDeque` is amortised O(1) |
| `indegree[v]--` | O(1) | Array write |
| `adj.get(u).add(v)` | O(1) amortised | ArrayList append |
| `dsu.find(x)` | **O(α(V))** | Inverse Ackermann — effectively `< 5` for any realistic input, i.e. essentially O(1) |
| `dsu.union(a, b)` | O(α(V)) | Two finds + a constant-time link |
| `color[v] = -color[u]` | O(1) | Array write |

### Step 3: multiply

**Time = (unit of work) × (times per unit) × (cost per unit)**

For Kahn's: `V nodes × 1 dequeue × O(1)` + `E edges × 1 decrement × O(1)` = **O(V + E)**.

For Union-Find on E edges: `E unions × O(α(V))` + `V init × O(1)` = **O(V + E·α(V))** ≈ **O(V + E)**.

For 2-color BFS: same as any BFS = **O(V + E)**.

Every algorithm in this file has the exact same linear bound. This is the payoff of using the right tool — no algorithm here is more expensive than reading the input.

### Why α(V) is treated as O(1)

`α(n)` is the inverse Ackermann function. It grows so slowly that for any `n ≤ 2^65536` (basically the number of atoms in the observable universe, cubed), `α(n) < 5`. So in practice, Union-Find with path compression + union by rank is treated as O(1) per operation. Some textbooks write the bound as O(E · α(V)) for precision; most people just say O(E) because it is honest for any real input.

**Warning:** you MUST have both optimisations — path compression AND union by rank — to get the near-O(1) bound. Skipping either drops you to O(log V) per operation, which is still fast but not textbook-optimal.

### Space complexity breakdown

| Contributor | Size | When present |
|-------------|------|--------------|
| Adjacency list | O(V + E) | Kahn's, 2-color BFS, bipartite Union-Find on adj list |
| `indegree[]` array | O(V) | Kahn's only |
| BFS queue | O(V) worst case | Kahn's, 2-color BFS |
| `color[]` array | O(V) | 2-color BFS |
| DSU `parent[]` + `rank[]` | O(V) | Any Union-Find problem |
| Recursion stack (if using DFS variant) | O(V) worst case | 3-color DFS for LC 207/210 |

**Rule of thumb:** every problem in this file uses **O(V + E) space** total (dominated by the adjacency list). Union-Find-only problems (LC 684) that operate directly on the edge stream drop the adj list entirely and use just **O(V)** aux space.

### Common misconceptions killed

**"Kahn's algorithm looks like it might loop — every edge could re-enqueue its target?"**
No. Each node's in-degree hits zero **at most once** (in-degrees only decrease, and once a node is enqueued its in-degree is never inspected again). So each node is enqueued once, dequeued once, and each of its outgoing edges triggers exactly one decrement. Total edge work = E. Total node work = V. Sum = O(V + E).

**"Union-Find on n edges must be O(n log n) because trees are involved?"**
With path compression + union by rank, it collapses to O(α(n)) per operation, which is functionally O(1). The tree gets so flat so fast that any depth-based reasoning is misleading. Just remember: `parent[x] = find(parent[x])` (path compression) and rank-based union together produce the near-O(1) magic.

**"2-coloring might revisit nodes if we colour then un-colour?"**
We never un-colour. A node is coloured exactly once when it is first discovered, and any later encounter only reads the colour to check consistency. So each node is coloured once (O(V)) and each edge is inspected once from each endpoint (O(2E) = O(E)).

### TC quick-derivation checklist

For any new cycle-detection / topological / bipartite problem:

1. **What is V and what is E?** Extract from the input format.
2. **Which recipe applies?** Kahn's, Union-Find, or 2-color BFS?
3. **Does the problem require building an adjacency list first?** If yes, add an O(V + E) preprocessing step.
4. **How many operations does the recipe do?** Kahn's: `V + E` operations at O(1) each. Union-Find: `E` unions at O(α(V)) each. 2-color BFS: `V + E` operations at O(1) each.
5. **Multiply.** The answer is almost always **O(V + E)**.

If you get something bigger (log factors, quadratic, etc.) on a problem in this family, you either forgot path compression, or you are running an inner loop that scans edges/nodes repeatedly — usually a design bug.

### Applied to every problem in this note

| Problem | Time | Aux space | Why |
|---------|------|-----------|-----|
| Course Schedule (207) | O(V + E) | O(V + E) | Kahn's: build adj (O(E)) + BFS (O(V + E)) |
| Course Schedule II (210) | O(V + E) | O(V + E) | Same as 207 + O(V) for output order |
| Redundant Connection (684) | O(E · α(V)) ≈ O(E) | O(V) | Pure Union-Find on the edge stream, no adj list |
| Redundant Connection II (685) | O(E · α(V)) ≈ O(E) | O(V) | Two-parent scan (O(E)) + Union-Find pass (O(E · α(V))) |
| Is Graph Bipartite (785) | O(V + E) | O(V) | 2-color BFS or Union-Find on given adj list (no build cost) |
| Possible Bipartition (886) | O(V + E) | O(V + E) | Build adj list from `dislikes` (O(E)) + bipartite check (O(V + E)) |

**Meta-observation:** LC 684 and 685 are the outliers with O(V) aux space instead of O(V + E), because Union-Find on a raw edge stream never needs an adjacency list. Whenever you see a problem where you can iterate edges in input order without needing per-node neighbour lookups, Union-Find gives you the smaller footprint for free.

---

## 1. Course Schedule — LC 207

**Link:** https://leetcode.com/problems/course-schedule/

**Problem (1-liner):** Given `numCourses` and `prerequisites[i] = [a, b]` meaning "to take `a` you must first take `b`" (i.e. directed edge `b → a`), return `true` if you can finish every course.

### What tripped me up
- **I only pushed the first course with no prerequisites into the queue.** Big miss. Every course with `indegree == 0` must be seeded. Two independent "starter" courses? Both go in.
- Nearly added a `visited[]` array out of habit. Not needed — the in-degree hitting zero is what marks a node as "ready", and each node hits zero exactly once.
- Nearly built an **undirected** adjacency list for what is a **directed** prerequisite graph. Add the edge one way only: `b → a`, not both.

### Key insight — this is directed cycle detection dressed up as scheduling
- If the prerequisite graph has a **cycle**, you can never take any course in that cycle (they all depend on each other transitively).
- If it has **no cycle** (it is a DAG), Kahn's algorithm will drain the queue and every node gets processed.
- **The cycle check** = "did Kahn's process all `n` nodes?" If yes → no cycle → return true. If some nodes are left with `indegree > 0` → they are trapped in a cycle → return false.

### Approach — Kahn's algorithm

<details>
<summary><b>Show BFS code (Java)</b></summary>

```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    int[] indegree = new int[numCourses];
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());

    for (int[] p : prerequisites) {
        int course = p[0], prereq = p[1];
        adj.get(prereq).add(course);            // DIRECTED: prereq → course
        indegree[course]++;
    }

    Queue<Integer> q = new ArrayDeque<>();
    for (int i = 0; i < numCourses; i++) {
        if (indegree[i] == 0) q.offer(i);       // seed ALL zero-in-degree courses
    }

    int processed = 0;
    while (!q.isEmpty()) {
        int u = q.poll();
        processed++;
        for (int v : adj.get(u)) {
            if (--indegree[v] == 0) q.offer(v);
        }
    }

    return processed == numCourses;
}
```

</details>

- **Time:** O(V + E). Build adjacency list = O(E). Kahn's processes each node once (O(V)) and each edge once (O(E)).
- **Space:** O(V + E) adjacency list + O(V) in-degree + O(V) queue.

### Next-time checklist
1. **Seed the queue with ALL zero-in-degree nodes**, not just the first one. Iterate `0..n-1` fully.
2. **No `visited[]`.** The `indegree[v]--` is the visited mechanism; each node hits zero at most once.
3. **Directed adjacency list** — add the edge one way. Prerequisite `b` unlocks course `a` means edge `b → a`.
4. **Cycle check = `processed == n`**. If some nodes never got their in-degree to zero, they are stuck in a cycle.

### Gotchas I missed
- The problem's `[a, b]` format is easy to misread — it means "to take `a` first take `b`", so the edge is `b → a` (from prereq TO course). Getting the direction backwards silently returns wrong answers on any non-trivial input.
- Empty prerequisites list → every course has in-degree 0 → queue is seeded with all courses → all get processed → return true. This works "for free" with the standard code; do not add a special case.
- **Alternative approach: DFS with 3 colors** (white/gray/black). Same TC. Gray = currently in the recursion stack. Seeing gray from gray = back edge = cycle. Mentioned for completeness; Kahn's is usually cleaner.

---

## 2. Course Schedule II — LC 210

**Link:** https://leetcode.com/problems/course-schedule-ii/

**Problem (1-liner):** Same as LC 207, but instead of a boolean, return an actual valid course order. Empty array if impossible.

### What tripped me up
- Almost nothing new after LC 207 — the algorithm is identical. Just record the pop-order.
- **Same bug potential:** seeding only one zero-in-degree course still bites here, and returns a partial (wrong) order.

### Key insight — Kahn's *is* topological sort
- Every time you pop a node from Kahn's queue, that node has no remaining unmet prerequisites → it can be taken NEXT in a valid order.
- Recording the pop-order as you go builds a topological order for free. No second pass needed.
- If Kahn's completes with `processed < n` → cycle → return empty array.

### Approach — Kahn's with pop-order recording

<details>
<summary><b>Show BFS code (Java)</b></summary>

```java
public int[] findOrder(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    int[] indegree = new int[numCourses];
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());

    for (int[] p : prerequisites) {
        adj.get(p[1]).add(p[0]);
        indegree[p[0]]++;
    }

    Queue<Integer> q = new ArrayDeque<>();
    for (int i = 0; i < numCourses; i++) {
        if (indegree[i] == 0) q.offer(i);
    }

    int[] order = new int[numCourses];
    int idx = 0;
    while (!q.isEmpty()) {
        int u = q.poll();
        order[idx++] = u;                      // record in pop-order
        for (int v : adj.get(u)) {
            if (--indegree[v] == 0) q.offer(v);
        }
    }

    return idx == numCourses ? order : new int[0];
}
```

</details>

- **Time:** O(V + E). Same as 207.
- **Space:** O(V + E) + output.

### Next-time checklist
1. Everything from LC 207 checklist applies.
2. **Record on pop, not on enqueue.** The pop-time is when the node is "committed" as the next in the order.
3. **`idx == numCourses` is your cycle check.** If not all nodes were processed, return an empty array.

### Gotchas I missed
- The output must have exactly `numCourses` entries. Returning `order` with trailing zeros when a cycle exists would silently pass on trivial tests but fail asymptotic ones.
- **Multiple valid orders exist** — the judge accepts any of them. No need to sort or normalise.

---

## 3. Redundant Connection — LC 684

**Link:** https://leetcode.com/problems/redundant-connection/

**Problem (1-liner):** Start with a tree of `n` nodes (`n` nodes, `n - 1` edges, no cycles). One extra edge is added, producing a graph with exactly one cycle. Return that redundant edge. If multiple could be removed, return the one that appears LAST in the input.

### What tripped me up
- This is where Union-Find really clicks for me. The problem is literally "process edges in order, tell me the first one that closes a cycle."
- Kept trying to think of a DFS solution — possible but way messier (find any cycle, then walk the input list to find the last edge in it).

### Key insight — Union-Find is purpose-built for this
- Iterate edges in the order given. For each `(u, v)`, try `dsu.union(u, v)`.
- If `union` succeeds → this edge connects two previously separate components → not redundant.
- If `union` **fails** → both endpoints are already in the same component → this edge closes a cycle → return it.
- Because we process in input order and return the first failure, we automatically return the LAST edge that could be removed (the tree's `n-1` non-redundant edges will all union successfully; the only failure comes when the cycle-closing edge is processed).

### Approach — Union-Find, single pass

<details>
<summary><b>Show code (Java)</b></summary>

```java
public int[] findRedundantConnection(int[][] edges) {
    int n = edges.length;                       // n edges, n nodes labelled 1..n
    DSU dsu = new DSU(n + 1);                   // +1 because nodes are 1-indexed

    for (int[] e : edges) {
        if (!dsu.union(e[0], e[1])) return e;   // first failure = redundant edge
    }
    return new int[0];                          // unreachable per problem constraints
}

// DSU class from Recipe A above
```

</details>

- **Time:** O(E · α(V)) ≈ O(E) with path compression + union by rank.
- **Space:** O(V) for `parent` and `rank`.

### Next-time checklist
1. **"Extra edge in a tree" → Union-Find, single pass, return first union-failure.**
2. Nodes are 1-indexed in this problem — size the DSU as `n + 1` to avoid off-by-one.
3. Do NOT build an adjacency list — Union-Find operates directly on the edge stream.

### Gotchas I missed
- 1-indexed nodes bite here. Sizing DSU as `n` and doing `parent[i] = i` for `i in 0..n-1` will fail when you try to `find(n)`.
- The problem promises exactly one redundant edge, so exactly one `union` will fail. No need to iterate further after finding it.
- **Contrast with LC 685 (below):** this is undirected. Life is simple. Directed changes everything.

---

## 4. Redundant Connection II — LC 685

**Link:** https://leetcode.com/problems/redundant-connection-ii/

**Problem (1-liner):** Same setup but the graph is **directed** — it was originally a **rooted tree** (every node has exactly one parent except the root, which has none), plus one extra directed edge. Return the redundant edge; if multiple options, return the LAST in input.

### What tripped me up
- This one is genuinely tricky. Union-Find alone is not enough because a directed tree has an extra constraint: **each node has exactly one incoming edge (in-degree 1)**, except the root (in-degree 0). The extra edge can break that constraint OR create a cycle OR both.

### Key insight — three cases to handle
Adding one extra directed edge to a rooted tree produces exactly one of these:

- **Case A: A node has TWO parents, no cycle.** Example: root `1 → 2`, root `1 → 3`, then extra edge `3 → 2` gives `2` two parents. Answer: return the LAST of the two conflicting incoming edges.
- **Case B: There is a cycle, but every node still has in-degree 1.** The root's original incoming edge from another node creates a cycle. Answer: run Union-Find and return the edge that closes the cycle.
- **Case C: A node has two parents AND there is a cycle.** Answer: return the FIRST of the two conflicting edges (the one that is part of the cycle).

The algorithm handles all three cases in one pass:

1. **Scan for a two-parent node.** Record the two candidate edges as `cand2` (in input order — `cand1` is earlier). If no such node exists, we are in Case B (pure cycle) — skip to step 3.
2. **Try Union-Find while pretending `cand2` does not exist** (skip it during the union pass). If union succeeds on every other edge → Case A: return `cand2`.
3. **If a union failure happens during the pass:**
   - If `cand2` exists (i.e. we detected a two-parent case AND a cycle) → Case C: return `cand1`.
   - If no two-parent was found → Case B: return the union-failing edge.

### Approach — indegree scan + Union-Find

<details>
<summary><b>Show code (Java)</b></summary>

```java
public int[] findRedundantDirectedConnection(int[][] edges) {
    int n = edges.length;
    int[] parent = new int[n + 1];              // for the two-parent scan (1-indexed)
    int[] cand1 = null, cand2 = null;

    // Step 1: find two-parent conflict, if any
    for (int[] e : edges) {
        int u = e[0], v = e[1];
        if (parent[v] == 0) {
            parent[v] = u;
        } else {
            cand1 = new int[]{parent[v], v};    // earlier edge into v
            cand2 = e;                          // later edge into v (this one)
            break;
        }
    }

    // Step 2: Union-Find pass, SKIPPING cand2
    DSU dsu = new DSU(n + 1);
    for (int[] e : edges) {
        if (e == cand2) continue;               // pretend cand2 does not exist
        if (!dsu.union(e[0], e[1])) {
            // Union failed → cycle detected among edges we DID include
            return cand1 == null ? e : cand1;   // Case B or Case C
        }
    }

    // No cycle in the pruned graph → Case A: cand2 is the culprit
    return cand2;
}

// DSU class from Recipe A above
```

</details>

- **Time:** O(E · α(V)) ≈ O(E).
- **Space:** O(V) for `parent`, `DSU`.

### Next-time checklist
1. **Directed version → three cases.** Do not try to force the undirected Union-Find pattern; you WILL get wrong answers on Case A.
2. **Two-parent scan first, Union-Find second.** Skip `cand2` during Union-Find to isolate whether a cycle exists without it.
3. **Case decision tree:**
   - No two-parent, cycle exists → Case B → return the union-failing edge.
   - Two-parent found, no cycle without `cand2` → Case A → return `cand2`.
   - Two-parent found AND cycle without `cand2` → Case C → return `cand1`.

### Gotchas I missed
- **Case C is the trap.** If you naively return `cand2` whenever a two-parent conflict exists, you fail on graphs where the cycle involves `cand1`.
- **1-indexed again** — size DSU and `parent` as `n + 1`.
- The `e == cand2` reference comparison works here because `cand2` is a reference to the exact `int[]` object from the input array. If you built a copy, you would need `Arrays.equals(e, cand2)` instead.

---

## 5. Is Graph Bipartite? — LC 785

**Link:** https://leetcode.com/problems/is-graph-bipartite/

**Problem (1-liner):** Given an undirected graph as an adjacency list `graph[i]` = neighbours of node `i`, return `true` iff the graph can be split into two sets `A` and `B` such that every edge connects a node in `A` to a node in `B`. Equivalently: can we 2-color the graph?

### What tripped me up
- **Forgot the graph can be disconnected.** BFS from node 0 alone misses whole components. Answer: wrap the coloring loop in `for (int i = 0; i < n; i++) if (color[i] == 0) ...`.
- Found the 2-coloring approach unintuitive at first — kept second-guessing which colour to assign next.
- Union-Find felt more natural to me for this problem (surprise: it works!). If a node and any of its neighbours end up in the same DSU component, they must have the same color → not bipartite.

### Key insight — bipartite ⟺ no odd cycle
- A graph is bipartite iff it contains **no odd-length cycles**. Both 2-coloring and Union-Find detect exactly this.
- **2-color approach:** start with any node, color it 1. BFS: colour every neighbour with the opposite colour. If you ever try to colour a node with a colour it already has that disagrees with what you were about to assign → conflict → odd cycle exists → not bipartite.
- **Union-Find approach:** for each node `u`, union all of its neighbours together (they all must share a colour since they are all "one hop from u", opposite to u). If `u` itself is already in the same DSU component as any of its neighbours → they must all share u's colour AND u's opposite colour → contradiction → not bipartite.

### Approach 1 — BFS 2-coloring

<details>
<summary><b>Show 2-color BFS (Java)</b></summary>

```java
public boolean isBipartite(int[][] graph) {
    int n = graph.length;
    int[] color = new int[n];                       // 0 uncoloured, 1 and -1 are the two colours

    for (int start = 0; start < n; start++) {       // outer loop for DISCONNECTED graph
        if (color[start] != 0) continue;
        Queue<Integer> q = new ArrayDeque<>();
        q.offer(start);
        color[start] = 1;
        while (!q.isEmpty()) {
            int u = q.poll();
            for (int v : graph[u]) {
                if (color[v] == 0) {
                    color[v] = -color[u];
                    q.offer(v);
                } else if (color[v] == color[u]) {
                    return false;                   // same colour on adjacent → odd cycle
                }
            }
        }
    }
    return true;
}
```

</details>

### Approach 2 — Union-Find (my personal favourite here)

<details>
<summary><b>Show Union-Find (Java)</b></summary>

```java
public boolean isBipartite(int[][] graph) {
    int n = graph.length;
    DSU dsu = new DSU(n);

    for (int u = 0; u < n; u++) {
        for (int v : graph[u]) {
            if (dsu.find(u) == dsu.find(v)) return false;   // same component as a neighbour → not bipartite
            dsu.union(graph[u][0], v);                       // union all neighbours of u together
        }
    }
    return true;
}

// DSU class from Recipe A above
```

</details>

- **Time (both):** O(V + E). BFS visits each node/edge once. Union-Find: E unions with near-constant α(V) per op.
- **Space:** O(V) for `color` or `DSU`.

### Next-time checklist
1. **ALWAYS wrap traversal in an outer loop over all nodes** for bipartite / connected-components on potentially-disconnected graphs. The problem statement rarely promises connectivity.
2. **2-coloring:** colour on enqueue, check on discovery. Mismatch = not bipartite.
3. **Union-Find alternative:** for each node, union all its neighbours together. If a node is ever in the same component as a neighbour → not bipartite. Great mental model if colouring feels off.

### Gotchas I missed
- Trying to bipartite-check without the outer loop: fails silently on any test case where node 0 is disconnected from the problematic component. Was my exact bug the first time.
- The 2-color check must be strict inequality (`color[v] == color[u]` → bad), NOT `color[v] != -color[u]` — same thing but easier to get right with the direct comparison.
- **Union-Find over neighbours** — `graph[u][0]` (the first neighbour) is the anchor; every other neighbour of `u` unions to it. Any neighbour will do, but be consistent.

---

## 6. Possible Bipartition — LC 886

**Link:** https://leetcode.com/problems/possible-bipartition/

**Problem (1-liner):** Given `n` people (1-indexed) and `dislikes[i] = [a, b]` meaning person `a` dislikes person `b`, split them into two groups so no two people who dislike each other are in the same group. Return `true` if possible.

### What tripped me up
- Identical to LC 785 in disguise. Once I saw that the `dislikes` list is just an edge list of an **undirected** graph, and "split into two groups so no edge is intra-group" is the definition of bipartite, the whole problem collapsed.
- **Same disconnected-graph mistake** — again forgot the outer loop the first time. Some people might have no dislikes and form their own trivial component.
- Again Union-Find felt more natural than 2-coloring.

### Key insight — this IS LC 785 with a wrapper
- Step 1: build undirected adjacency list from `dislikes` (add BOTH directions).
- Step 2: run either bipartite algorithm (2-color BFS or Union-Find on neighbours).
- Done.

### Approach — Union-Find (mirror of LC 785)

<details>
<summary><b>Show code (Java)</b></summary>

```java
public boolean possibleBipartition(int n, int[][] dislikes) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i <= n; i++) adj.add(new ArrayList<>());   // 1-indexed

    for (int[] d : dislikes) {
        adj.get(d[0]).add(d[1]);
        adj.get(d[1]).add(d[0]);                    // UNDIRECTED: both ways
    }

    DSU dsu = new DSU(n + 1);
    for (int u = 1; u <= n; u++) {
        if (adj.get(u).isEmpty()) continue;
        int anchor = adj.get(u).get(0);
        for (int v : adj.get(u)) {
            if (dsu.find(u) == dsu.find(v)) return false;
            dsu.union(anchor, v);
        }
    }
    return true;
}

// DSU class from Recipe A above
```

</details>

- **Time:** O(V + E · α(V)) ≈ O(V + E).
- **Space:** O(V + E) adjacency list + O(V) DSU.

### Next-time checklist
1. **"Split into two groups so X" → bipartite check.** Same tool as LC 785.
2. **`dislikes` is an UNDIRECTED edge list** — add both directions when building the adjacency list.
3. Same disconnected-graph gotcha: outer loop over ALL people, not just person 1.
4. 1-indexed → size DSU and adjacency list as `n + 1`.

### Gotchas I missed
- **The problem gives you an edge list, not an adjacency list.** You must build the adjacency list yourself (Step 1 above). Skipping this step and trying to iterate `dislikes` directly makes the outer bipartite loop awkward.
- **1-indexed nodes** — always size arrays as `n + 1` and start loops at 1.
- If `dislikes` is empty, every person can go in group A → return true. This falls out of the standard code for free.

---

## Summary — cycle detection at a glance

| Problem | Graph type | Tool | Cycle check | Why |
|---------|------------|------|-------------|-----|
| LC 207 Course Schedule | Directed | Kahn's BFS | `processed != n` | Nodes left over = trapped in a cycle |
| LC 210 Course Schedule II | Directed | Kahn's BFS + pop-order | Same as 207 | Pop-order IS the topological order |
| LC 684 Redundant Connection | Undirected | Union-Find single pass | First `union` failure | Tree + 1 edge = exactly one cycle-creator |
| LC 685 Redundant Connection II | Directed rooted tree + 1 | Indegree scan + Union-Find | Three cases (2-parent / cycle / both) | Directed constraint: in-degree 1 for non-root |
| LC 785 Is Graph Bipartite? | Undirected | 2-color BFS or Union-Find | Same colour on adjacent nodes | Bipartite ⟺ no odd cycle |
| LC 886 Possible Bipartition | Undirected | 2-color BFS or Union-Find | Same as 785 | LC 785 in disguise |

**Reusable meta-lessons:**
1. **Kahn's queue must include EVERY zero-in-degree node.** Not just the first one you see.
2. **No `visited[]` needed for Kahn's** — the in-degree counter IS the visited mechanism.
3. **Always check directed vs undirected** before building the adjacency list. Wrong direction is the #1 bug across this whole family.
4. **Disconnected graphs need an outer loop** — bipartite / colouring / component-counting problems all bite here.
5. **Union-Find is often the shortest correct answer** when you are processing edges one at a time, or when 2-coloring feels unintuitive. Its cache-hit branch (`find(a) == find(b)`) is a natural cycle detector.
