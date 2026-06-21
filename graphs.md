# Graph Patterns — Reference Note

*Status: in progress — fill in each section as you work through it.*

## Quick Lookup Table

| Pattern | Key signal | Core idea |
|---|---|---|
| Graph representation | — | Adjacency list (sparse) vs. matrix (dense) vs. edge list |
| DFS & connected components | "is connected", "number of groups" | Recursive/iterative DFS, mark visited |
| BFS & multi-source BFS | "shortest path (unweighted)", "rotting oranges" style | Queue, level-by-level expansion from one or many sources |
| Cycle detection (undirected) | "redundant connection" | DFS + parent tracking, or Union-Find |
| Cycle detection (directed) | "course schedule" style | DFS + recursion-stack/coloring |
| Topological sort | "course order", "build order" | Kahn's (BFS + in-degree) or DFS postorder |
| Union-Find (DSU) | connectivity / grouping over time | Union by rank + path compression |
| Dijkstra's algorithm | "shortest path, weighted, non-negative" | Priority queue, greedily expand nearest unvisited |
| Bipartite check | "two-coloring", "can divide into two groups" | BFS/DFS coloring, alternate colors per level |
| MST (Prim's / Kruskal's) | "minimum cost to connect all" | Builds on Union-Find (Kruskal's) or priority queue (Prim's) |

---

## 1. Graph Representation

**When to use it:** Always your first decision before writing any traversal — pick adjacency list for sparse graphs (most interview problems), matrix for dense graphs or when O(1) edge-existence lookups matter.

**Template:**
```java
// Adjacency list
List<List<Integer>> graph = new ArrayList<>();
for (int i = 0; i < n; i++) graph.add(new ArrayList<>());
for (int[] edge : edges) {
    graph.get(edge[0]).add(edge[1]);
    graph.get(edge[1]).add(edge[0]); // omit this line if directed
}
```

**Why this shape:** *(fill in after working through a few problems)*

**Edge cases:** *(fill in)*

**Common pitfalls:** *(fill in)*

**Example problems:** *(fill in)*

---

## 2. DFS & Connected Components

*(fill in: template, why it works, edge cases, pitfalls, example problems)*

---

## 3. BFS & Multi-source BFS

*(fill in)*

---

## 4. Cycle Detection — Undirected vs. Directed

*(fill in — remember these are genuinely different techniques, don't conflate them)*

---

## 5. Topological Sort

*(fill in)*

---

## 6. Union-Find (DSU)

*(fill in)*

---

## 7. Dijkstra's Algorithm

*(fill in)*

---

## 8. Bipartite Check / Graph Coloring

*(fill in)*

---

## 9. Minimum Spanning Tree (Prim's / Kruskal's)

*(fill in)*
