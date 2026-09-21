# Queue Patterns — Master Reference Note

```java
// Java's inbuilt queue support — pick based on what you actually need
Queue<Integer> queue = new ArrayDeque<>();       // plain FIFO queue, preferred implementation
Deque<Integer> deque = new ArrayDeque<>();        // double-ended, needed for monotonic-deque patterns
PriorityQueue<Integer> minHeap = new PriorityQueue<>(); // NOT FIFO — priority-ordered, min-heap by default
```

---

# Part 1 — Foundations (read this before Part 2)

## Why does Queue exist as a data structure?

A queue models exactly one idea: **the first thing that arrived is the first thing that
leaves.** FIFO — First In, First Out. Picture a line of people at a checkout counter:
whoever joined the line first gets served first, and new arrivals join at the back,
never cutting in. That fairness/ordering guarantee is the entire point — it's the
mirror-image restriction of a stack's LIFO rule, and it turns out to be exactly the
shape needed any time "order of arrival" must be preserved and processed fairly.

**The core insight:** any time a problem needs you to process things "in the order they
showed up," or explore a graph/tree "layer by layer, nearest things first," that's a
queue — even if the problem statement never uses the word.

## Real-world examples (where this already runs, right now, without you thinking about it)

1. **Print queue / job scheduling.** Documents print, or CPU tasks run, in the order
   they were submitted (or with priorities layered on top — see `PriorityQueue` below).
2. **Message queues / event streaming** (Kafka, SQS, RabbitMQ). Producers push events
   onto one end; consumers pull them off the other, in order, often across many
   consumers pulling from the same ordered stream.
3. **Breadth-First Search (BFS)** on trees/graphs. Exploring "everything at distance 1
   before anything at distance 2" is only possible because a queue preserves arrival
   order — this is *why* BFS finds shortest paths in unweighted graphs and DFS doesn't.
4. **CPU round-robin scheduling / IO buffering.** Processes/bytes are serviced in the
   order they arrived, in fixed-size time slices, cycling back to the queue's tail.
5. **Rate limiting / request buffering.** Incoming requests queue up and get processed
   at a sustainable rate rather than dropped or processed out of order.
6. **The "waiting line" experience itself** — customer service call queues, hospital
   triage (a `PriorityQueue` in disguise — urgent cases jump the FIFO line).

## Java's inbuilt queue-related tools

### 1. `Queue<T>` — the interface, not a concrete class

`Queue` is an interface. You never `new Queue<>()` — you back it with a concrete
implementation, most commonly `ArrayDeque`.

### 2. `ArrayDeque<T>` — the modern default choice for a plain FIFO queue

- Use as a queue with `offer(e)` (add to tail) and `poll()` (remove from head).
- **Why preferred over `LinkedList`:** resizable circular-array backing, no per-node
  object allocation overhead, better cache locality. The official Java docs recommend
  `ArrayDeque` over both `Stack` (for stack use) and `LinkedList` (for queue use).
- **Gotcha:** `ArrayDeque` does **not** allow `null` elements — `offer(null)` throws
  `NullPointerException`. This trips people up coming from `LinkedList`, which does
  allow nulls.

### 3. `LinkedList<T>` — implements both `Queue` and `Deque`

- Allows `null` elements, which `ArrayDeque` doesn't — occasionally the deciding factor.
- Generally slower than `ArrayDeque` for pure queue/stack usage due to per-node
  allocation, but useful if you also need arbitrary list operations on the same object.

### 4. `PriorityQueue<T>` — NOT a FIFO queue, a binary heap in disguise

- Orders elements by priority (natural ordering or a supplied `Comparator`), not by
  arrival order. Default is a **min-heap** (`peek()`/`poll()` return the smallest).
- For a max-heap: `new PriorityQueue<>(Comparator.reverseOrder())`.
- Complexity: O(log n) insert (`offer`) and remove-min (`poll`), O(1) peek-min.
- **Gotcha:** iterating a `PriorityQueue` directly (e.g., with a for-each loop) does
  **not** yield elements in sorted order — only the head (`peek()`) is guaranteed to be
  the min/max. You must repeatedly `poll()` to drain it in priority order.

### 5. Concurrent queues (brief mention — know these exist)

- `ConcurrentLinkedQueue` — non-blocking, thread-safe FIFO queue.
- `LinkedBlockingQueue` / `ArrayBlockingQueue` — blocking queues that make threads wait
  when empty (on `take()`) or full (on `put()`), the backbone of the classic
  producer-consumer pattern (pattern #11 below).

### Quick comparison table — safe vs. exception-throwing methods

Every `Queue` operation comes in two flavors: one that throws on failure, one that
returns a sentinel value instead. **Prefer the sentinel-returning versions** in
algorithm code — they let you write clean `while (!queue.isEmpty())` loops without
try/catch.

| Operation | Throws on failure | Returns sentinel on failure |
|---|---|---|
| Insert | `add(e)` → `IllegalStateException` if capacity-restricted and full | `offer(e)` → returns `false` |
| Remove | `remove()` → `NoSuchElementException` if empty | `poll()` → returns `null` |
| Examine | `element()` → `NoSuchElementException` if empty | `peek()` → returns `null` |

---

# Part 2 — Queue Patterns (Master Reference)

## Quick Lookup Table (read this first when revisiting after a gap)

| # | Pattern | Trigger phrases | Anchor problem | Core idea |
|---|---|---|---|---|
| 1 | BFS Traversal (Level Order) | "level by level", "shortest path (unweighted)" | Binary Tree Level Order Traversal | Queue preserves arrival order; snapshot size before the inner loop |
| 2 | Multi-Source BFS | "rotting oranges", "walls and gates", "01 matrix" | Rotting Oranges | Push ALL sources into the queue before the first BFS layer starts |
| 3 | Topological Sort (Kahn's Algorithm) | "course schedule", "build order", "kahn's algorithm" | Course Schedule | Queue of zero-in-degree nodes; decrement neighbors' in-degree as you go |
| 4 | Monotonic Deque (Sliding Window Max/Min) | "max/min in every window of size k" | Sliding Window Maximum | Deque holds indices in decreasing value order; evict both ends |
| 5 | Top K Elements (Heap of size K) | "kth largest", "top k frequent", "k closest points" | Kth Largest Element in an Array | Maintain a heap capped at size K, don't sort the whole input |
| 6 | Merge K Sorted Lists/Arrays | "merge k sorted lists", "merge k sorted arrays" | Merge k Sorted Lists | Min-heap holds one "current head" candidate per list |
| 7 | Two Heaps for Running Median | "find median from data stream" | Find Median from Data Stream | Max-heap for the lower half, min-heap for the upper half, keep them balanced |
| 8 | Task Scheduling with Cooldown | "task scheduler", "rearrange string k distance apart" | Task Scheduler | Max-heap picks the most-frequent task; a cooldown queue holds recently-run tasks |
| 9 | Circular Queue / Bounded Buffer Design | "design circular queue", "design hit counter", "moving average" | Design Circular Queue | Fixed-size array, head/tail indices wrap with modulo |
| 10 | Stack via Two Queues | "implement stack using queues" | Implement Stack using Queues | One queue does the work; rotate it after each push to fake LIFO order |
| 11 | Producer-Consumer (Blocking Queue) | "producer consumer problem", "bounded buffer", "thread-safe queue" | (concurrency design problem, not LeetCode-style) | `BlockingQueue.put()`/`take()` block instead of failing when full/empty |

---

## 1. BFS Traversal (Level Order)

**When to use it:** Any "level by level" exploration, or "shortest path in an unweighted
graph" — BFS finds shortest paths precisely because a queue explores in strict distance
order, unlike DFS.

**Trigger phrases:** "level order traversal", "shortest path (unweighted)", "minimum number of steps", "right/left side view".

**Anchor problem:** Binary Tree Level Order Traversal.

**Template:**
```java
List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;

    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int size = queue.size(); // snapshot BEFORE the inner loop — the #1 BFS bug if skipped
        List<Integer> level = new ArrayList<>();

        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
    }
    return result;
}
```

**Why this shape:** Snapshotting `queue.size()` before the inner loop is what separates
"one level" from "everything currently queued" — without it, children enqueued during
the current level's processing would get consumed in the same iteration, silently
merging levels together.

**Edge cases:**
- Empty tree/graph (`root == null`) → return immediately, queue never used.
- Disconnected graph (general graph, not tree) → need an outer loop over all unvisited
  nodes, same idea as the DFS version in `graphs/bfs-dfs-traversal.md`.

**Common pitfalls:**
- Forgetting the `size` snapshot (see above) — merges multiple levels into one.
- Marking nodes visited on **poll** instead of on **offer/enqueue** in general-graph
  BFS — allows the same node to be enqueued multiple times by different neighbors
  before it's ever processed, wasting work and occasionally causing incorrect distances.

**Example problems:** Binary Tree Level Order Traversal, Binary Tree Right Side View, Minimum Depth of Binary Tree, Word Ladder. See `graphs/bfs-dfs-traversal.md` for the full general-graph BFS treatment (adjacency lists/matrices, connected components).

---

## 2. Multi-Source BFS

**When to use it:** "Multiple starting points spread simultaneously" — rot spreading,
fire spreading, distance-to-nearest-gate problems.

**Trigger phrases:** "rotting oranges", "walls and gates", "01 matrix", "shortest distance from any of these cells".

**Anchor problem:** Rotting Oranges.

**Template:**
```java
int orangesRotting(int[][] grid) {
    Queue<int[]> queue = new ArrayDeque<>();
    int fresh = 0;

    for (int r = 0; r < grid.length; r++) {
        for (int c = 0; c < grid[0].length; c++) {
            if (grid[r][c] == 2) queue.offer(new int[]{r, c}); // seed ALL sources up front
            if (grid[r][c] == 1) fresh++;
        }
    }

    int minutes = 0;
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    while (!queue.isEmpty() && fresh > 0) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            int[] cell = queue.poll();
            for (int[] d : dirs) {
                int nr = cell[0] + d[0], nc = cell[1] + d[1];
                if (nr >= 0 && nr < grid.length && nc >= 0 && nc < grid[0].length && grid[nr][nc] == 1) {
                    grid[nr][nc] = 2;
                    fresh--;
                    queue.offer(new int[]{nr, nc});
                }
            }
        }
        minutes++;
    }
    return fresh == 0 ? minutes : -1;
}
```

**Why this shape:** Seeding every source into the queue *before* the first BFS layer
runs means all sources expand outward simultaneously, one shared "ring" per minute/step
— exactly matching real spreading behavior, instead of processing sources one at a time
(which would need K separate BFS passes and give wrong timing).

**Edge cases:**
- No sources at all (no rotten oranges) but some fresh ones exist → loop never runs,
  `fresh > 0` at the end → return `-1`.
- No fresh cells to begin with → `minutes` stays `0`, correct immediately.

**Common pitfalls:**
- Running BFS once per source sequentially instead of seeding all sources together —
  gives the wrong "time," not just slower code.
- Forgetting to decrement/track the remaining count (`fresh`) to detect the unreachable case.

**Example problems:** Rotting Oranges, Walls and Gates, 01 Matrix, Shortest Path in Binary Matrix.

---

## 3. Topological Sort (Kahn's Algorithm)

**When to use it:** Ordering tasks/courses with prerequisite (dependency) constraints
in a directed acyclic graph.

**Trigger phrases:** "course schedule", "build order", "task ordering with dependencies", "kahn's algorithm".

**Anchor problem:** Course Schedule (II).

**Template:**
```java
int[] findOrder(int numCourses, int[][] prerequisites) {
    List<List<Integer>> graph = new ArrayList<>();
    int[] inDegree = new int[numCourses];
    for (int i = 0; i < numCourses; i++) graph.add(new ArrayList<>());
    for (int[] p : prerequisites) {
        graph.get(p[1]).add(p[0]); // p[1] -> p[0]
        inDegree[p[0]]++;
    }

    Queue<Integer> queue = new ArrayDeque<>();
    for (int i = 0; i < numCourses; i++) if (inDegree[i] == 0) queue.offer(i);

    int[] order = new int[numCourses];
    int idx = 0;
    while (!queue.isEmpty()) {
        int node = queue.poll();
        order[idx++] = node;
        for (int next : graph.get(node)) {
            if (--inDegree[next] == 0) queue.offer(next); // only ready once ALL prereqs done
        }
    }
    return idx == numCourses ? order : new int[0]; // fewer than numCourses processed => cycle
}
```

**Why this shape:** A node is only "ready" once every prerequisite pointing into it has
been processed — tracking `inDegree` and only enqueuing at zero guarantees a node never
gets ordered before something it depends on. If a cycle exists, some nodes' in-degree
never reaches zero, so they're never enqueued — the queue drains early, which is exactly
how this pattern doubles as cycle detection for free.

**Edge cases:**
- A cycle exists → fewer than `numCourses` nodes get processed; return empty/false per
  the problem's contract.
- No dependencies at all → every node starts at in-degree 0, order is simply insertion order.

**Common pitfalls:**
- Getting the edge direction backwards (`prerequisite -> course` vs `course -> prerequisite`).
- Forgetting the final "did we process everything?" check — this is the only way to
  detect a cycle in this formulation.

**Example problems:** Course Schedule I/II, Alien Dictionary, Sequence Reconstruction, Minimum Height Trees.

---

## 4. Monotonic Deque (Sliding Window Max/Min)

**When to use it:** "Max or min in every contiguous window of size k."

**Trigger phrases:** "sliding window maximum", "max/min of every subarray of size k".

**Anchor problem:** Sliding Window Maximum.

> Full walkthrough (why-this-shape, edge cases, pitfalls) lives in `stack.md` pattern
> #11 — a `Deque` used at both ends is the natural generalization of the monotonic
> stack patterns covered there. Template repeated here since it's equally a core
> **queue-family** pattern:

```java
int[] maxSlidingWindow(int[] nums, int k) {
    Deque<Integer> deque = new ArrayDeque<>(); // indices, values DECREASING front→back
    int[] result = new int[nums.length - k + 1];

    for (int i = 0; i < nums.length; i++) {
        if (!deque.isEmpty() && deque.peekFirst() <= i - k) deque.pollFirst(); // expired front
        while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i]) deque.pollLast(); // useless back
        deque.offerLast(i);
        if (i >= k - 1) result[i - k + 1] = nums[deque.peekFirst()];
    }
    return result;
}
```

**Example problems:** Sliding Window Maximum, Shortest Subarray with Sum at Least K.

---

## 5. Top K Elements (Heap of Size K)

**When to use it:** "Find the K largest/smallest/most-frequent/closest" — without
needing the full sorted order of everything else.

**Trigger phrases:** "kth largest element", "top k frequent elements", "k closest points to origin".

**Anchor problem:** Kth Largest Element in an Array.

**Template (find the K largest — use a min-heap capped at size K):**
```java
int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>(); // default min-heap
    for (int num : nums) {
        minHeap.offer(num);
        if (minHeap.size() > k) minHeap.poll(); // evict the smallest — keep only the top K seen so far
    }
    return minHeap.peek(); // the smallest of the K largest == the Kth largest overall
}
```

**Why this shape:** A **min**-heap capped at size K is the trick for finding the K
**largest** elements (and vice versa) — counter-intuitive at first, but the heap only
ever needs to know "is this candidate bigger than my current worst-of-the-best?", which
is exactly what a min-heap's `peek()` tells you in O(1). This runs in O(n log k) instead
of O(n log n) for a full sort — a real complexity win when k is small relative to n.

**Edge cases:**
- `k == nums.length` → heap ends up holding everything, equivalent to a full sort.
- Duplicate values → heap handles them naturally, no special casing needed.

**Common pitfalls:**
- Using a max-heap of the *whole* input instead of a size-capped min-heap — technically
  correct but throws away the O(log k) benefit, degrading to O(n log n).
- Forgetting to `poll()` once size exceeds K — the heap silently grows unbounded instead
  of staying capped.

**Example problems:** Kth Largest Element in an Array, Top K Frequent Elements, K Closest Points to Origin, Kth Smallest Element in a Sorted Matrix.

---

## 6. Merge K Sorted Lists/Arrays

**When to use it:** Combining K already-sorted sequences into one sorted output
efficiently, without concatenating and re-sorting everything.

**Trigger phrases:** "merge k sorted lists", "merge k sorted arrays", "smallest range covering elements from k lists".

**Anchor problem:** Merge k Sorted Lists.

**Template:**
```java
ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> minHeap = new PriorityQueue<>((a, b) -> a.val - b.val);
    for (ListNode node : lists) if (node != null) minHeap.offer(node);

    ListNode dummy = new ListNode(-1);
    ListNode tail = dummy;

    while (!minHeap.isEmpty()) {
        ListNode smallest = minHeap.poll();
        tail.next = smallest;
        tail = tail.next;
        if (smallest.next != null) minHeap.offer(smallest.next); // advance that list's pointer
    }
    return dummy.next;
}
```

**Why this shape:** The heap only ever needs to hold **one candidate per list** — the
current head of each. Popping the global minimum and immediately offering that same
list's next node keeps exactly K candidates in the heap at all times, giving
O(N log K) total instead of O(N log N) from a full merge-and-sort (N = total elements
across all lists, K = number of lists).

**Edge cases:**
- Some input lists are empty/null → skip adding them to the heap initially.
- Only one list → heap degenerates to a simple linear pass.

**Common pitfalls:**
- Forgetting to re-offer `smallest.next` — silently drops the rest of that list.
- Comparator direction mistakes when merging in descending order instead of ascending.

**Example problems:** Merge k Sorted Lists, Merge k Sorted Arrays, Smallest Range Covering Elements from K Lists.

---

## 7. Two Heaps for Running Median

**When to use it:** Maintaining a running statistic (median) over a stream of numbers
as they arrive, without re-sorting on every insert.

**Trigger phrases:** "find median from data stream", "sliding window median".

**Anchor problem:** Find Median from Data Stream.

**Template:**
```java
class MedianFinder {
    private final PriorityQueue<Integer> lowerHalf = new PriorityQueue<>(Comparator.reverseOrder()); // max-heap
    private final PriorityQueue<Integer> upperHalf = new PriorityQueue<>(); // min-heap

    public void addNum(int num) {
        lowerHalf.offer(num);
        upperHalf.offer(lowerHalf.poll()); // always push the "winner" across to balance

        if (upperHalf.size() > lowerHalf.size()) {
            lowerHalf.offer(upperHalf.poll()); // rebalance if upperHalf grew too big
        }
    }

    public double findMedian() {
        if (lowerHalf.size() > upperHalf.size()) return lowerHalf.peek();
        return (lowerHalf.peek() + upperHalf.peek()) / 2.0;
    }
}
```

**Why this shape:** The median only ever depends on the one or two middle elements —
you never need the full sorted order. Splitting the stream into "everything below the
median" (a max-heap, so its largest is instantly accessible) and "everything above" (a
min-heap, so its smallest is instantly accessible), kept balanced in size, means both
middle candidates are always sitting at O(1)-accessible heap tops.

**Edge cases:**
- Even total count → median is the average of both heap tops.
- First element ever added → both heaps start empty, first `offer`/`poll` pair still
  works correctly (rebalance check handles the size-1 case fine).

**Common pitfalls:**
- Rebalancing only sometimes instead of after every insert — sizes silently drift more
  than 1 apart and the median formula breaks.
- Using the wrong heap type for either half (both must-have `lowerHalf` as max-heap,
  `upperHalf` as min-heap — swapping them silently gives nonsense results, not a crash).

**Example problems:** Find Median from Data Stream, Sliding Window Median, IPO (two-heap variant with a different pairing rule).

---

## 8. Task Scheduling with Cooldown

**When to use it:** Scheduling repeated tasks/characters with a mandatory minimum gap
between repeats of the same item.

**Trigger phrases:** "task scheduler", "rearrange string k distance apart", "reorganize string".

**Anchor problem:** Task Scheduler.

**Template:**
```java
int leastInterval(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char t : tasks) freq[t - 'A']++;

    PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
    for (int f : freq) if (f > 0) maxHeap.offer(f);

    int time = 0;
    Queue<int[]> cooldown = new ArrayDeque<>(); // {remaining count, time it becomes available again}

    while (!maxHeap.isEmpty() || !cooldown.isEmpty()) {
        time++;
        if (!maxHeap.isEmpty()) {
            int count = maxHeap.poll() - 1;
            if (count > 0) cooldown.offer(new int[]{count, time + n});
        }
        if (!cooldown.isEmpty() && cooldown.peek()[1] == time) {
            maxHeap.offer(cooldown.poll()[0]); // cooldown expired — back into contention
        }
    }
    return time;
}
```

**Why this shape:** The max-heap always greedily picks the most-frequent remaining
task (spreading out the tasks that need it most, since they have the least slack), while
the cooldown queue naturally holds tasks "on ice" in the exact order they'll become
available again — a queue is the right structure there because cooldown expiry is a
pure FIFO-by-time relationship.

**Edge cases:**
- `n == 0` (no cooldown required) → answer is simply the total task count.
- One task type dominates heavily → idle slots are unavoidable, answer exceeds task count.

**Common pitfalls:**
- Using a plain counter/array scan instead of a heap for "most frequent remaining" —
  works but is asymptotically worse and easy to get subtly wrong on ties.
- Forgetting idle time counts toward `time` even when both structures still have items
  but nothing is immediately schedulable.

**Example problems:** Task Scheduler, Rearrange String k Distance Apart, Reorganize String.

---

## 9. Circular Queue / Bounded Buffer Design

**When to use it:** "Design" problems needing a fixed-capacity queue backed by an array,
without the wasted space a naive array-shifting queue would cause.

**Trigger phrases:** "design circular queue", "design hit counter", "moving average from data stream", "design circular deque".

**Anchor problem:** Design Circular Queue.

**Template:**
```java
class MyCircularQueue {
    private final int[] buffer;
    private int head = 0, size = 0;
    private final int capacity;

    public MyCircularQueue(int k) {
        buffer = new int[k];
        capacity = k;
    }

    public boolean enQueue(int value) {
        if (size == capacity) return false;
        buffer[(head + size) % capacity] = value;
        size++;
        return true;
    }

    public boolean deQueue() {
        if (size == 0) return false;
        head = (head + 1) % capacity; // advance head, wrap via modulo
        size--;
        return true;
    }

    public int Front() { return size == 0 ? -1 : buffer[head]; }
    public int Rear()  { return size == 0 ? -1 : buffer[(head + size - 1) % capacity]; }
}
```

**Why this shape:** A naive array queue either wastes the freed-up front slots forever,
or needs an O(n) shift on every dequeue. Tracking `head` and `size` and using modulo
arithmetic to "wrap around" the fixed-size array reuses freed slots automatically,
giving true O(1) enqueue/dequeue with no wasted space and no shifting.

**Edge cases:**
- Queue completely full (`size == capacity`) → `enQueue` must reject, not overwrite.
- Queue completely empty (`size == 0`) → `deQueue`/`Front`/`Rear` must reject/return
  sentinel, not read garbage/stale data from the buffer.

**Common pitfalls:**
- Using two raw index pointers (`head`/`tail`) without a `size` counter — makes it
  genuinely ambiguous whether the queue is empty or completely full when `head == tail`.
- Forgetting the modulo wrap on `Rear()`'s index calculation.

**Example problems:** Design Circular Queue, Design Circular Deque, Design Hit Counter, Moving Average from Data Stream.

---

## 10. Stack via Two Queues

**When to use it:** "Implement a LIFO stack using only queue operations" — the mirror
image of `stack.md` pattern #8 (Queue via Two Stacks).

**Trigger phrases:** "implement stack using queues".

**Anchor problem:** Implement Stack using Queues.

**Template (single-queue approach — rotate after every push, simpler than a two-queue version):**
```java
class MyStack {
    private final Queue<Integer> queue = new ArrayDeque<>();

    public void push(int x) {
        queue.offer(x);
        // rotate everything BEFORE x to sit behind it, making x the "front" (i.e. the new top)
        for (int i = 0; i < queue.size() - 1; i++) {
            queue.offer(queue.poll());
        }
    }

    public int pop() { return queue.poll(); }
    public int top() { return queue.peek(); }
    public boolean empty() { return queue.isEmpty(); }
}
```

**Why this shape:** A queue can't naturally give you "the last thing added" — its front
is always the *oldest* item. The trick is to pay the reordering cost on **push**, by
rotating every previously-queued element behind the new one right away, so the queue's
front always happens to be the most-recently-pushed element. This makes `push` O(n) but
`pop`/`top` O(1) — the opposite trade-off from the Queue-via-Two-Stacks pattern, where
push is O(1) and pop is amortized O(1) but occasionally O(n).

**Edge cases:**
- Pushing onto an empty stack → the rotation loop runs zero times, correctly.
- Single-element stack → `pop`/`top` trivially return that element.

**Common pitfalls:**
- Rotating the *wrong* number of times (`queue.size()` instead of `queue.size() - 1`) —
  either under-rotates (top is wrong) or spins all the way back to the original order.

**Example problems:** Implement Stack using Queues, Implement Queue using Stacks (`stack.md` pattern #8, the mirror image).

---

## 11. Producer-Consumer (Blocking Queue)

**When to use it:** Concurrency/design questions about coordinating multiple threads
producing and consuming work items safely, without busy-waiting or manual `wait()`/`notify()`.

**Trigger phrases:** "producer consumer problem", "bounded buffer", "thread-safe queue between threads".

**Anchor problem:** (a concurrency design problem, not a typical LeetCode algorithm — comes up in system-design-flavored interviews.)

**Template:**
```java
BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(capacity);

// Producer thread
void produce(int item) throws InterruptedException {
    queue.put(item); // BLOCKS (doesn't fail) if the queue is full, until space frees up
}

// Consumer thread
int consume() throws InterruptedException {
    return queue.take(); // BLOCKS (doesn't fail) if the queue is empty, until an item arrives
}
```

**Why this shape:** `BlockingQueue` moves all the wait/notify coordination logic out of
your code and into a battle-tested standard-library implementation — `put()`/`take()`
handle full/empty conditions by blocking the calling thread, rather than you writing
manual `synchronized`/`wait()`/`notifyAll()` boilerplate that's easy to get subtly wrong
(lost wakeups, spurious wakeup handling, missed notifications).

**Edge cases:**
- Multiple producers and/or multiple consumers on the same queue — `BlockingQueue`
  implementations are internally thread-safe for exactly this multi-producer/multi-consumer case.
- Graceful shutdown — typically needs a sentinel "poison pill" value pushed through the
  queue to signal consumers to stop, since `take()` blocks forever otherwise.

**Common pitfalls:**
- Reaching for a plain `ArrayDeque`/`LinkedList` across threads without external
  synchronization — neither is thread-safe, unlike `BlockingQueue` implementations.
- Not having a shutdown/poison-pill strategy, leaving consumer threads blocked forever
  on `take()` when producers are done.

**Example problems:** Classic Producer-Consumer, Design a rate limiter, Design a thread pool's task queue.

---

## My Mistakes (fill in as you go)

_Add your own here when you debug something for more than 10 minutes — write down why
you got it wrong, not just the fix._
