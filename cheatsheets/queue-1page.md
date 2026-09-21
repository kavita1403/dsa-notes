# Queue — Cheat Sheet

**Default choice:** `Queue<Integer> queue = new ArrayDeque<>();` — `offer()`/`poll()`/`peek()` (safe, return sentinel) over `add()`/`remove()`/`element()` (throw on failure).

**Why queue exists:** FIFO — first to arrive is first served. BFS, message queues, task scheduling, round-robin CPU scheduling all reduce to this one shape.

**Gotchas:** `ArrayDeque` does NOT allow `null` (throws NPE) — `LinkedList` does. `PriorityQueue` is a min-heap, NOT sorted order on iteration — only `peek()`/`poll()` are ordered.

## BFS / Level Order
```java
int size = queue.size(); // snapshot BEFORE inner loop — the #1 BFS bug if skipped
for (int i = 0; i < size; i++) {
    var node = queue.poll();
    // process, then enqueue children
}
```

## Multi-Source BFS
```java
// seed ALL sources into the queue up front, THEN run BFS layer by layer
for (source : sources) queue.offer(source);
```

## Topological Sort (Kahn's)
```java
for (i : nodes) if (inDegree[i] == 0) queue.offer(i);
while (!queue.isEmpty()) {
    int node = queue.poll();
    for (next : graph.get(node)) if (--inDegree[next] == 0) queue.offer(next);
}
// fewer than N nodes processed at the end => cycle exists
```

## Monotonic Deque (Sliding Window Max)
```java
if (!deque.isEmpty() && deque.peekFirst() <= i - k) deque.pollFirst(); // expired front
while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i]) deque.pollLast(); // useless back
deque.offerLast(i);
```
Full detail: `stack.md` pattern #11.

## Top K Elements (heap capped at size K)
```java
minHeap.offer(num);
if (minHeap.size() > k) minHeap.poll(); // evict smallest — keeps top K seen so far
```
Min-heap for "K largest", max-heap for "K smallest" — counter-intuitive, memorize it.

## Merge K Sorted Lists
```java
PriorityQueue<ListNode> minHeap = new PriorityQueue<>((a, b) -> a.val - b.val);
// seed one node per list, then: poll smallest, append, re-offer its .next
```

## Two Heaps — Running Median
```java
lowerHalf.offer(num);                    // max-heap
upperHalf.offer(lowerHalf.poll());       // always push winner across
if (upperHalf.size() > lowerHalf.size()) lowerHalf.offer(upperHalf.poll()); // rebalance
```

## Task Scheduler (max-heap + cooldown queue)
```java
maxHeap.offer(freq); // most frequent task first
cooldown.offer(new int[]{remaining, time + n}); // parked until cooldown expires
if (cooldown.peek()[1] == time) maxHeap.offer(cooldown.poll()[0]); // back into contention
```

## Circular Queue (array + modulo)
```java
buffer[(head + size) % capacity] = value; size++;      // enqueue
head = (head + 1) % capacity; size--;                   // dequeue
```
Track `size` explicitly — `head == tail` alone can't tell empty from full.

## Stack via Two Queues (single-queue rotate approach)
```java
queue.offer(x);
for (int i = 0; i < queue.size() - 1; i++) queue.offer(queue.poll()); // rotate old elements behind x
```
Push O(n), pop O(1) — opposite trade-off from Queue-via-Two-Stacks (`stack.md` pattern #8).

## Producer-Consumer
```java
BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(capacity);
queue.put(item);  // blocks if full, doesn't throw
queue.take();     // blocks if empty, doesn't throw
```
