# Graphs — Foundational Reference

## Adjacency List vs. Adjacency Matrix vs. Grid

Quick correction on terminology first: it's **adjacency** list/matrix, not "agency" — *adjacency* means "nearness / being next to," which is exactly what these structures represent: which nodes are next to each other.

---

## 1. Grid (2D matrix representing a spatial map)

An `n × m` matrix where **each cell is a node**, and connections are **implicit** — always to the cell's up/down/left/right (sometimes diagonal) neighbors. You never store the neighbor relationships explicitly; you derive them with fixed offset arrays.

```java
int[][] grid = {
    {1, 1, 0},
    {0, 1, 0},
    {0, 0, 1}
};

// neighbor of (i, j) is always (i±1, j) and (i, j±1) — no lookup needed
int[] dx = {1, -1, 0, 0};
int[] dy = {0, 0, 1, -1};
```

Used for grid-based problems like **Number of Islands**, **Flood Fill**, **Maze** problems.

---

## 2. Adjacency Matrix (graph representing arbitrary connections)

An `n × n` matrix where `n` = number of nodes, and `matrix[i][j] = 1` means node `i` is connected to node `j`. Unlike a grid, there's **no spatial meaning** — node 5 could be connected to node 0 even though they're "far apart" in index.

```java
// isConnected from LeetCode 547 — this IS an adjacency matrix
int[][] isConnected = {
    {1, 1, 0},
    {1, 1, 0},
    {0, 0, 1}
};

// to find neighbors of node i, scan the whole row: O(n)
for (int j = 0; j < n; j++) {
    if (isConnected[i][j] == 1) {
        // i and j are connected
    }
}
```

This is **547's input**. Finding neighbors here costs O(n) per node (full row scan), unlike the grid's O(1).

---

## 3. Adjacency List (graph representing arbitrary connections, sparse)

Instead of an `n × n` matrix (which wastes space if most nodes *aren't* connected to each other), you store **only the actual edges**, per node, as a list.

```java
// 323's edges = [[0,1],[1,2],[3,4]] converted to adjacency list
List<List<Integer>> adj = new ArrayList<>();
for (int i = 0; i < n; i++) adj.add(new ArrayList<>());

for (int[] edge : edges) {
    adj.get(edge[0]).add(edge[1]);
    adj.get(edge[1]).add(edge[0]); // undirected
}

// adj now looks like:
// 0: [1]
// 1: [0, 2]
// 2: [1]
// 3: [4]
// 4: [3]

// neighbors of node i = adj.get(i) directly, no scanning: O(degree of i)
```

Note: it's not just "how many neighbors a node has," it's the actual **list of *which* nodes** are its neighbors.

---

## Side-by-Side Comparison

|                                     | Grid                                  | Adjacency Matrix                              | Adjacency List                                |
| ----------------------------------- | ------------------------------------- | --------------------------------------------- | --------------------------------------------- |
| **Shape**                           | n × m                                 | n × n                                         | n entries, each a list                        |
| **Neighbor relationship**           | Implicit (spatial: up/down/left/right)| Explicit, stored as 0/1 in every cell         | Explicit, stored only for actual edges        |
| **Cost to find neighbors of a node**| O(1) — fixed 4 offsets                | O(n) — must scan the row                      | O(degree) — just read the list                |
| **Space**                           | O(n·m)                                | O(n²) — even if graph is sparse               | O(V + E) — only actual edges stored           |
| **Best for**                        | Spatial/grid problems                 | Dense graphs, or when given directly          | Sparse graphs, most general-purpose problems  |
| **Example problem**                 | 200. Number of Islands                | 547. Number of Provinces                      | 323. Number of Connected Components           |

---

## The One-Line Mental Model

Matrix and list both represent *arbitrary* graphs (any node can connect to any other). The difference is just **how you're allowed to check "is X connected to Y"**:

- **Adjacency matrix:** O(1) direct lookup
- **Adjacency list:** O(degree) scan through a list

A **grid isn't really a third *representation* of the same idea** — it's a special case where the graph structure is so predictable (always 4 neighbors) that you don't need to store it at all.
