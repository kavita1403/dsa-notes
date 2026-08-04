# Grid as Graph — Revision Notes

Grid problems (2D matrix of cells) are just graph problems in disguise. Every cell is a node; every "step to an adjacent cell" is an edge. Once that click happens, every technique from `bfs-dfs-traversal.md` transfers over — with a few grid-specific tricks stacked on top.

> **Tip:** Code blocks are collapsed by default (click the ▸ arrow to expand). Same convention as the traversal notes.

---

## Meta: the four grid-as-graph patterns

After solving the eight problems below, four reusable solution shapes emerged. Recognise which one a new problem belongs to and half the work is done.

| Pattern | Idea in one line | Canonical LC problems |
|---------|------------------|-----------------------|
| **1. Normal DFS/BFS from a source** | Pick a cell, flood outward, mark visited as you go. | Flood Fill (733), Number of Islands (200), Max Area of Island (695) |
| **2. Multi-source BFS** | Put **all** sources into the queue at once, then BFS level-by-level. Levels = distance/time. | Rotting Oranges (994), Walls and Gates (286), 01 Matrix (542) |
| **3. Reverse DFS/BFS** | Instead of "can I get from cell X to target?", walk **from the target outward** and mark whatever it can reach. Turns O((mn)²) into O(mn). | Pacific Atlantic (417) |
| **4. Boundary DFS/BFS** | Start from cells on the border, mark whatever they touch as "safe/escaped", then treat everything unmarked as trapped. | Surrounded Regions (130), Number of Enclaves (1020) |

There is also **Pattern 0 — no traversal at all** (e.g. Island Perimeter, LC 463), where you just look at each cell's neighbours locally without exploring the graph. Always ask "do I actually need to traverse?" before writing DFS.

---

## Grid boilerplate you will re-type on every problem

### The universal 4-direction array
```java
int[][] dirs = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};   // up, down, left, right
```
For **8-direction** problems (rare — usually stated explicitly), add the four diagonals: `{-1,-1}, {-1,1}, {1,-1}, {1,1}`.

### The bounds-check pattern
```java
for (int[] d : dirs) {
    int nr = r + d[0], nc = c + d[1];
    if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;   // out of bounds
    if (visited[nr][nc]) continue;                          // already seen
    // ... explore (nr, nc) ...
}
```
Combine both checks into one guard clause at the top of your DFS if you prefer — style choice.

---

## Marking cells as visited — three strategies

You **must** mark visited on grids. Cycles are guaranteed because every edge is bidirectional (going up then down puts you right back). Without a visited mechanism, you get infinite recursion or wildly wrong counts. Three ways to do it:

| Strategy | Space | Pros | Cons |
|----------|-------|------|------|
| **`boolean[][] visited`** | O(m·n) extra | Clean, non-destructive | Extra memory |
| **Mutate grid in place** (e.g. `'1' → '0'` in Number of Islands) | O(1) extra | Zero extra space | Destroys the input; not safe if caller reuses it |
| **Sentinel value** (e.g. mark visited land as `'#'` or `2`) | O(1) extra | Cheap; sometimes semantically meaningful (e.g. Surrounded Regions uses this to mean "safe") | Same destructive concern; needs a value that cannot collide with real data |

**Rule of thumb:** default to `boolean[][] visited`. Switch to in-place mutation only when the problem explicitly allows it or when memory is tight. Sentinel is often the cleanest choice for boundary-DFS problems (Q5).

---

## DFS vs BFS on grids — quick guidance

- **Reachability / count / area / perimeter:** DFS is usually shorter. Recursion depth is bounded by grid size (m·n), and typical LC constraints keep this manageable.
- **Shortest-path / minimum-time / "how many steps":** BFS. Every cell is one step, so BFS visits in distance order. This is why Rotting Oranges (994) and Walls and Gates (286) are BFS, not DFS.
- **Multi-source (many starts):** BFS again — you seed the queue with all sources up front, then peel off layers.

For DFS↔BFS mechanics on grids, all three rules from [`dfs-to-bfs-conversion.md`](./dfs-to-bfs-conversion.md) still apply (mark on enqueue, pack per-frame state into queue elements, keep answer logic in the same structural spot).

---

## 1. Flood Fill — LC 733

**Link:** https://leetcode.com/problems/flood-fill/

**Problem (1-liner):** Given an `m x n` grid `image`, a start pixel `(sr, sc)`, and a `color`, replace every pixel that is 4-connected to `(sr, sc)` and shares its original colour with `color`. Return the modified image.

### What tripped me up
- Nothing structural — this is the canonical "flood a connected component" problem.
- One subtle trap I nearly missed: **if the new colour equals the original colour**, a naive DFS loops forever because the "am I the right colour?" check never fires. Guard against it upfront.

### Key insight
- Classic **Pattern 1 (normal DFS from a source)**. Remember the source pixel's original colour, DFS from it, only recurse into 4-neighbours that still hold the original colour, and recolour on entry.
- The recolouring itself doubles as the visited marker — once a cell is repainted, it no longer matches the "original colour" guard, so it will not be re-entered. No extra `visited[][]` needed.

### Approach — DFS

<details>
<summary><b>Show DFS code (Java)</b></summary>

```java
public int[][] floodFill(int[][] image, int sr, int sc, int color) {
    int original = image[sr][sc];
    if (original == color) return image;               // guard: infinite loop otherwise
    dfs(image, sr, sc, original, color);
    return image;
}

void dfs(int[][] img, int r, int c, int original, int color) {
    if (r < 0 || r >= img.length || c < 0 || c >= img[0].length) return;
    if (img[r][c] != original) return;                 // wrong colour or already repainted
    img[r][c] = color;                                 // paint = mark visited
    dfs(img, r - 1, c, original, color);
    dfs(img, r + 1, c, original, color);
    dfs(img, r, c - 1, original, color);
    dfs(img, r, c + 1, original, color);
}
```

</details>

- **Time:** O(m · n) — each cell is visited at most once.
- **Space:** O(m · n) worst-case recursion stack (if the whole grid is one giant connected region).

### Next-time checklist
1. **Same-colour guard is mandatory.** `if (original == color) return image;` at the top.
2. **Recolouring IS the visited marker** — do not add a redundant `visited[][]`.
3. Four recursive calls, one per direction. Or use the `dirs` array if you prefer the loop.

### Gotchas I missed
- **Original vs new colour comparison** — always compare the neighbour cell against the *stored `original`*, not against the source pixel's current colour (which is now the new colour after you painted).
- **Grid boundaries:** the `r < 0 || ...` check must come **before** you index into `img[r][c]`, otherwise `ArrayIndexOutOfBoundsException`.

---

## 2. Number of Islands — LC 200

**Link:** https://leetcode.com/problems/number-of-islands/

**Problem (1-liner):** Given an `m x n` grid of `'1'` (land) and `'0'` (water), count the number of 4-connected land islands.

### What tripped me up
- Same DFS pattern as Flood Fill, but with an **outer loop** over every cell (because we do not know where the islands are). The outer loop + `if unvisited land` pattern is identical to LC 547 Provinces — that "each launch = one component" idea.

### Key insight
- **Pattern 1** again, wrapped in an outer loop. Each DFS launch drowns one entire island (marks all its cells as visited); count the launches.
- **In-place mutation:** turn each visited `'1'` into `'0'` to save the `visited[][]` array. Standard trick for grid problems that let you destroy the input.

### Approach — DFS (in-place)

<details>
<summary><b>Show DFS code (Java)</b></summary>

```java
public int numIslands(char[][] grid) {
    int m = grid.length, n = grid[0].length, count = 0;
    for (int r = 0; r < m; r++) {
        for (int c = 0; c < n; c++) {
            if (grid[r][c] == '1') {
                sink(grid, r, c);
                count++;
            }
        }
    }
    return count;
}

void sink(char[][] g, int r, int c) {
    if (r < 0 || r >= g.length || c < 0 || c >= g[0].length) return;
    if (g[r][c] != '1') return;                        // water or already sunk
    g[r][c] = '0';                                     // sink = mark visited
    sink(g, r - 1, c);
    sink(g, r + 1, c);
    sink(g, r, c - 1);
    sink(g, r, c + 1);
}
```

</details>

- **Time:** O(m · n) — every cell is inspected once by the outer loop and at most once by DFS.
- **Space:** O(m · n) worst-case recursion (one giant island). BFS variant avoids the stack.

### Next-time checklist
1. **Outer loop scans every cell**; DFS only launches on land.
2. **Sink land as you visit it** — cheapest way to mark visited on grid problems.
3. If the caller cares about preserving `grid`, use `boolean[][] visited` instead of mutation.

### Gotchas I missed
- The grid is `char[][]`, not `int[][]` — compare against `'1'` and `'0'` (chars), not `1` and `0` (ints).
- **BFS conversion** is trivial — swap `sink` recursion for a queue that pushes 4-neighbours. Complexity unchanged.

---

## 3. Max Area of Island — LC 695

**Link:** https://leetcode.com/problems/max-area-of-island/

**Problem (1-liner):** Given an `m x n` grid of `1` (land) and `0` (water), return the area (cell count) of the largest 4-connected island. Return `0` if none.

### What tripped me up
- Recognised it as "Number of Islands, but instead of counting launches, sum up the cells inside each launch and track the max."
- Small twist: DFS now needs to **return an int** (area of the component it explored) rather than mutating a shared counter.

### Key insight
- **Pattern 1** with a per-launch accumulator. Each DFS call returns `1 + sum of areas from its four recursive calls`. The outer loop tracks the max across launches.
- This is the same shape as tree recursion problems that return "size of subtree rooted here".

### Approach — DFS returning area

<details>
<summary><b>Show DFS code (Java)</b></summary>

```java
public int maxAreaOfIsland(int[][] grid) {
    int m = grid.length, n = grid[0].length, best = 0;
    for (int r = 0; r < m; r++) {
        for (int c = 0; c < n; c++) {
            if (grid[r][c] == 1) best = Math.max(best, area(grid, r, c));
        }
    }
    return best;
}

int area(int[][] g, int r, int c) {
    if (r < 0 || r >= g.length || c < 0 || c >= g[0].length) return 0;
    if (g[r][c] != 1) return 0;
    g[r][c] = 0;                                       // sink to prevent re-visit
    return 1
         + area(g, r - 1, c)
         + area(g, r + 1, c)
         + area(g, r, c - 1)
         + area(g, r, c + 1);
}
```

</details>

- **Time:** O(m · n). **Space:** O(m · n) recursion worst case.

### Next-time checklist
1. **DFS returns area** (an int), not void.
2. `return 1 + sum(four recursive calls)` — the `+1` counts the current cell.
3. Bounds-fail and water-fail both return `0` so they contribute nothing to the sum.

### Gotchas I missed
- Do not forget to sink the current cell (`g[r][c] = 0`) **before** the four recursive calls. Sinking after would let neighbours recurse back into this cell, infinite loop.
- Empty grid or all-water grid → answer is `0`. The outer loop simply never launches DFS, so `best` stays `0`. Handled implicitly.

---

## 4. Island Perimeter — LC 463

**Link:** https://leetcode.com/problems/island-perimeter/

**Problem (1-liner):** Given a grid with exactly **one** island, return its perimeter (the number of edges between land cells and water/boundary).

### What tripped me up
- Kept reaching for DFS out of habit. This is actually **Pattern 0 — no traversal needed**. A pure counting problem.

### Key insight
- Every land cell contributes up to 4 perimeter edges. For each of its four sides, subtract 1 if that side touches another land cell (shared edge, not on perimeter).
- Equivalently: perimeter = `4 · (land cells) − 2 · (adjacent land pairs)`. Each adjacent pair cancels two edges (one from each cell's perspective).
- Iterate the whole grid once; no DFS, no BFS, no `visited[][]`.

### Approach — direct count

<details>
<summary><b>Show count code (Java)</b></summary>

```java
public int islandPerimeter(int[][] grid) {
    int m = grid.length, n = grid[0].length, perim = 0;
    int[][] dirs = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};

    for (int r = 0; r < m; r++) {
        for (int c = 0; c < n; c++) {
            if (grid[r][c] != 1) continue;
            for (int[] d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                boolean neighbourIsWaterOrOutside =
                    nr < 0 || nr >= m || nc < 0 || nc >= n || grid[nr][nc] == 0;
                if (neighbourIsWaterOrOutside) perim++;
            }
        }
    }
    return perim;
}
```

</details>

- **Time:** O(m · n). **Space:** O(1) extra.

### Next-time checklist
1. **"Perimeter" and "boundary count" problems usually do not need traversal.** Look for a local per-cell contribution formula first.
2. **Out-of-bounds counts as water** for perimeter purposes — the edge of the grid IS a perimeter edge.
3. DFS variant exists (return `1` on OOB or water, `0` on already-visited land, and sum recursively) but adds no clarity and burns stack.

### Gotchas I missed
- Do not use `visited[][]` — you WANT to inspect every land cell independently. Marking visited would double-count nothing and just add code.
- The problem guarantees exactly one island; if it did not, this formula still works (it computes total perimeter across all islands combined).

---

## 5. Surrounded Regions — LC 130

**Link:** https://leetcode.com/problems/surrounded-regions/

**Problem (1-liner):** Given an `m x n` grid of `'X'` and `'O'`, flip every `'O'` region that is **completely surrounded** by `'X'` to `'X'`. An `'O'` region is not surrounded if any of its cells touches the border.

### What tripped me up
- Instinct was "find every `'O'` region, check whether any of its cells touch the border, if not flip it." Doable but ugly — you have to remember every cell of every region in case you decide to flip it.
- The elegant fix is a **reverse framing**: instead of hunting for surrounded regions, hunt for the **UN-surrounded** ones and mark them safe. Whatever is left is surrounded.

### Key insight — Pattern 4 (Boundary DFS)
- An `'O'` region escapes flipping **iff** it contains at least one border cell. So:
  1. Walk all border cells. For every `'O'` on the border, DFS/BFS across connected `'O'`s and mark them with a sentinel value like `'#'` (means "safe, do not flip").
  2. Sweep the whole grid: every remaining `'O'` is surrounded → flip to `'X'`. Every `'#'` → restore to `'O'`.
- This turns a hard "region membership" question into a simple two-pass mark-and-sweep.

The sentinel value pulls double duty (see the "marking cells as visited" table): it is both the visited marker AND the semantic label "this cell survives the flip."

### Approach — Boundary DFS with sentinel

<details>
<summary><b>Show DFS code (Java)</b></summary>

```java
public void solve(char[][] board) {
    int m = board.length, n = board[0].length;

    // 1. Mark border-connected 'O's as '#' (safe)
    for (int r = 0; r < m; r++) {
        markSafe(board, r, 0);
        markSafe(board, r, n - 1);
    }
    for (int c = 0; c < n; c++) {
        markSafe(board, 0, c);
        markSafe(board, m - 1, c);
    }

    // 2. Flip: 'O' → 'X' (surrounded); '#' → 'O' (restore safe)
    for (int r = 0; r < m; r++) {
        for (int c = 0; c < n; c++) {
            if (board[r][c] == 'O') board[r][c] = 'X';
            else if (board[r][c] == '#') board[r][c] = 'O';
        }
    }
}

void markSafe(char[][] b, int r, int c) {
    if (r < 0 || r >= b.length || c < 0 || c >= b[0].length) return;
    if (b[r][c] != 'O') return;                        // wall, water, or already marked
    b[r][c] = '#';
    markSafe(b, r - 1, c);
    markSafe(b, r + 1, c);
    markSafe(b, r, c - 1);
    markSafe(b, r, c + 1);
}
```

</details>

- **Time:** O(m · n) — boundary scan is O(m + n) launches, each launch touches each cell at most once. Final sweep is O(m · n).
- **Space:** O(m · n) recursion worst case.

### Next-time checklist
1. **"Something at the border escapes / survives" → Boundary DFS.** Start from the edges, not from the interior.
2. **Sentinel value** for "safe" is cleaner than a separate `visited[][]` here — you need to know which `'O'`s to keep anyway.
3. Two-pass sweep: mark first, flip second. Do not try to interleave.

### Gotchas I missed
- Only iterate the **four borders** in step 1, not the whole grid. Iterating the whole grid would launch DFS on interior `'O'`s too and mark them safe by mistake.
- The two corners `(0, 0)` etc. get visited twice by the border loops. Harmless — the second visit sees `'#'` and immediately returns.
- Do not forget the **restore step** (`'#' → 'O'`). Without it, you leave garbage sentinels in the output and fail the tests.

---

## 6. Rotting Oranges — LC 994

**Link:** https://leetcode.com/problems/rotting-oranges/

**Problem (1-liner):** Given an `m x n` grid where `0` = empty, `1` = fresh orange, `2` = rotten orange, return the **minimum number of minutes** until all fresh oranges rot. Every minute, each rotten orange rots its 4-neighbours. Return `-1` if some fresh orange can never rot.

### What tripped me up
- Trying to solve it with per-source BFS ("BFS from each rotten orange, take the min time each fresh orange rots at") is way too much work and wrong-in-spirit.
- The unlock: **all rotten oranges spread simultaneously**. So seed the BFS queue with **every** rotten orange up front, then process the queue level by level. Each BFS level = one minute.

### Key insight — Pattern 2 (Multi-source BFS)
- Multi-source BFS is single-source BFS with a fatter starting frontier. You still fan out one layer at a time; you just start with many roots instead of one.
- **Level counting:** track the number of nodes at the current level, drain them all, then increment the minute counter. Do this at every level *except the initial one* (minute 0 = the state you were handed).
- **Answer validity check:** after BFS, if any fresh oranges remain, they were unreachable → return `-1`.

### Approach — Multi-source BFS with fresh-count

<details>
<summary><b>Show BFS code (Java)</b></summary>

```java
public int orangesRotting(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    Queue<int[]> q = new ArrayDeque<>();
    int fresh = 0;

    // Seed queue with ALL rotten oranges; count fresh
    for (int r = 0; r < m; r++) {
        for (int c = 0; c < n; c++) {
            if (grid[r][c] == 2) q.offer(new int[]{r, c});
            else if (grid[r][c] == 1) fresh++;
        }
    }

    if (fresh == 0) return 0;                          // nothing to rot

    int minutes = 0;
    int[][] dirs = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};

    while (!q.isEmpty() && fresh > 0) {
        int size = q.size();                           // freeze this level's size
        for (int i = 0; i < size; i++) {
            int[] cell = q.poll();
            for (int[] d : dirs) {
                int nr = cell[0] + d[0], nc = cell[1] + d[1];
                if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
                if (grid[nr][nc] != 1) continue;       // not fresh: skip
                grid[nr][nc] = 2;                      // rot it (mark visited)
                fresh--;
                q.offer(new int[]{nr, nc});
            }
        }
        minutes++;                                     // one full level = one minute
    }

    return fresh == 0 ? minutes : -1;
}
```

</details>

- **Time:** O(m · n). Every cell is enqueued and dequeued at most once.
- **Space:** O(m · n) for the queue worst case.

### Next-time checklist
1. **"Spread from many sources simultaneously" → Multi-source BFS.** Seed the queue with all sources at once.
2. **Level counting** = process one level fully (`size = q.size()`; loop `size` times), then bump the timer.
3. **Track the remaining "unfinished" count** (`fresh` here) separately — it is both the "did we finish?" flag and an early-exit condition.
4. Rot as you enqueue (mark visited before neighbour re-add).

### Gotchas I missed
- **Snapshot the level size** with `int size = q.size();` **before** the inner loop. Do not use `q.size()` inside the loop — the queue grows as you enqueue new rotten oranges, and you would keep processing forever.
- **`fresh == 0` early return** matters: if the grid had no fresh oranges to begin with, the answer is `0`, not "however long BFS took."
- Multi-source BFS still uses the same "mark on enqueue" rule from the conversion notes — mark the cell as rotten before adding it to the queue.

---

## 7. Pacific Atlantic Water Flow — LC 417

**Link:** https://leetcode.com/problems/pacific-atlantic-water-flow/

**Problem (1-liner):** Given an `m x n` height map, water flows from a cell to any 4-neighbour with equal or lower height. The Pacific touches the top and left edges; the Atlantic touches the bottom and right edges. Return every cell from which water can reach **both** oceans.

### What tripped me up
- Instinct: "for each cell, DFS downhill and see if we reach both oceans." That is O((m·n)²) — every cell as source, each DFS is O(m·n). Times out.
- The unlock: **reverse the flow.** Instead of "which cells can reach the ocean?", ask "starting from the ocean, which cells can water climb *uphill* to reach?" Reverse DFS turns O((m·n)²) into O(m·n).

### Key insight — Pattern 3 (Reverse DFS)
- Do **two** DFS floods, one per ocean:
  1. Start from every Pacific-edge cell. Walk **uphill or equal** (i.e. `neighbourHeight >= currentHeight`). Mark every cell reached as "Pacific-reachable."
  2. Same thing from every Atlantic-edge cell into an "Atlantic-reachable" grid.
- Intersect the two reachable sets. Every cell in the intersection can flow to both oceans.
- Why reverse works: "A → B is downhill" is equivalent to "B → A is uphill or equal." Walking uphill from the ocean explores exactly the set of cells that can flow down to the ocean. Same set, computed in one linear sweep instead of one-per-source.

### Approach — Two reverse DFS, then intersect

<details>
<summary><b>Show DFS code (Java)</b></summary>

```java
public List<List<Integer>> pacificAtlantic(int[][] heights) {
    int m = heights.length, n = heights[0].length;
    boolean[][] pac = new boolean[m][n];
    boolean[][] atl = new boolean[m][n];

    // Seed the two floods from their respective ocean borders
    for (int r = 0; r < m; r++) {
        dfs(heights, pac, r, 0);                       // left edge → Pacific
        dfs(heights, atl, r, n - 1);                   // right edge → Atlantic
    }
    for (int c = 0; c < n; c++) {
        dfs(heights, pac, 0, c);                       // top edge → Pacific
        dfs(heights, atl, m - 1, c);                   // bottom edge → Atlantic
    }

    List<List<Integer>> result = new ArrayList<>();
    for (int r = 0; r < m; r++) {
        for (int c = 0; c < n; c++) {
            if (pac[r][c] && atl[r][c]) result.add(List.of(r, c));
        }
    }
    return result;
}

void dfs(int[][] h, boolean[][] seen, int r, int c) {
    seen[r][c] = true;
    int[][] dirs = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    for (int[] d : dirs) {
        int nr = r + d[0], nc = c + d[1];
        if (nr < 0 || nr >= h.length || nc < 0 || nc >= h[0].length) continue;
        if (seen[nr][nc]) continue;
        if (h[nr][nc] < h[r][c]) continue;             // uphill/equal only (reversed)
        dfs(h, seen, nr, nc);
    }
}
```

</details>

- **Time:** O(m · n). Two DFS floods, each touches each cell at most once.
- **Space:** O(m · n) for the two `seen` grids + O(m · n) recursion worst case.

### Next-time checklist
1. **"For each cell, can I reach X?" on a grid → try reversing.** Walking from X outward is usually O(m·n) vs O((m·n)²).
2. **Two-flood intersection pattern** shows up whenever the question is "reachable from both A and B".
3. Reverse the comparison operator too. Downhill in forward DFS becomes "uphill or equal" in reverse DFS.

### Gotchas I missed
- The condition is `>=`, not `>`. Water flows to **equal or lower**, so reverse is **equal or higher**. Miss the equal case and you drop a chunk of the answer.
- **Two separate `visited` grids** (`pac` and `atl`), not one shared. A cell can be Pacific-reachable, Atlantic-reachable, both, or neither — all four states must be distinguishable.
- Corner cells (`(0,0)`, `(0,n-1)`, etc.) are seeded twice by the two border loops. Harmless — DFS returns immediately on the second visit.

---

## 8. Walls and Gates — LC 286

**Link:** https://leetcode.com/problems/walls-and-gates/

**Problem (1-liner):** Given an `m x n` grid where `-1` = wall, `0` = gate, `INF` = empty room, fill every empty room with its distance to the **nearest** gate. If no gate is reachable, leave `INF`.

### What tripped me up
- Instinct: "BFS from each empty room, find the nearest gate." Same trap as Pacific Atlantic — O((m·n)²) and way too much work.
- Same fix: **flip the direction.** Multi-source BFS from all gates simultaneously. First time BFS reaches a room = shortest distance from *some* gate to that room (BFS visits in distance order).

### Key insight — Pattern 2 again (Multi-source BFS)
- Seed the queue with **every gate** up front (each at distance 0). BFS outward. Each time you visit an empty room, its distance from the nearest gate is `current distance + 1`.
- The room's own cell value doubles as visited marker: if `grid[r][c] != INF`, it is either a wall, gate, or already-filled room → do not re-enter.
- No level-counting needed because the distance is stored directly in the cell.

### Approach — Multi-source BFS with distance in cell

<details>
<summary><b>Show BFS code (Java)</b></summary>

```java
public void wallsAndGates(int[][] rooms) {
    final int EMPTY = Integer.MAX_VALUE;
    int m = rooms.length, n = rooms[0].length;
    Queue<int[]> q = new ArrayDeque<>();

    // Seed with every gate
    for (int r = 0; r < m; r++) {
        for (int c = 0; c < n; c++) {
            if (rooms[r][c] == 0) q.offer(new int[]{r, c});
        }
    }

    int[][] dirs = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    while (!q.isEmpty()) {
        int[] cell = q.poll();
        int r = cell[0], c = cell[1];
        for (int[] d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
            if (rooms[nr][nc] != EMPTY) continue;      // wall, gate, or already filled
            rooms[nr][nc] = rooms[r][c] + 1;           // distance = parent + 1
            q.offer(new int[]{nr, nc});
        }
    }
}
```

</details>

- **Time:** O(m · n). Every empty room is filled at most once, at which point it will never be revisited.
- **Space:** O(m · n) queue worst case.

### Next-time checklist
1. **"Distance to nearest X" on a grid → multi-source BFS from all X's.** Never per-source BFS from each starting point.
2. **Store the distance in the cell itself** when possible — kills two birds (visited + answer) with one write.
3. `rooms[nr][nc] = rooms[r][c] + 1` is the whole "distance calculation" — no separate `dist[][]` array needed.

### Gotchas I missed
- The `!= EMPTY` guard is the visited check. Walls (`-1`), gates (`0`), and already-filled rooms all fail this check → skipped. One condition covers three cases.
- **In-place mutation** is expected here — the problem literally asks you to fill the grid, not return a new one.
- Rooms that are unreachable from any gate stay as `INF`. The BFS never touches them, and the problem statement says to leave them alone.

---

## Summary — the four patterns, at a glance

| Pattern | When to reach for it | Key trick | Examples |
|---------|----------------------|-----------|----------|
| **Normal DFS/BFS** | "Explore this connected region." Answers about count, area, perimeter of a component. | Outer loop + `if !visited` for multi-component grids. | LC 733, 200, 695 |
| **Multi-source BFS** | "Spread from many sources simultaneously" OR "distance to nearest X." | Seed queue with all sources at once; level-count for time, or store distance directly. | LC 994, 286, 542 |
| **Reverse DFS/BFS** | "For each cell, can it reach X?" — beats the O((mn)²) per-cell approach. | Walk **from** X outward instead. Reverse the direction condition. | LC 417 |
| **Boundary DFS/BFS** | "Regions that escape/touch the border survive" OR "regions fully enclosed by X." | DFS from border cells, mark survivors with a sentinel, sweep and flip the rest. | LC 130, 1020 |
| **No traversal** | "Local per-cell contribution" (perimeter, corners, etc.). | Just scan the grid; no DFS needed. | LC 463 |

**The meta-question you should ask on any new grid problem:** *"Is my question about a single cell's reachability? A region's collective property? A distance? An escape condition?"* The answer maps directly to which pattern to use.
