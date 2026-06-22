# Linked List Patterns - Master Reference Note

```java
// Standard node definition assumed throughout
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}

// Doubly linked variant (used in LRU and a few problems)
class DListNode {
    int val;
    DListNode prev, next;
}

// THE most useful tool: sentinel / dummy node
ListNode dummy = new ListNode(0);
dummy.next = head;
// ... operate on dummy.next ...
return dummy.next;
```

---

## Decision Fork (ask yourself this before writing any code)

> "Detect cycle / find where cycle starts" -> **Slow/Fast Pointer (Floyd's tortoise & hare)**.
>
> "Find middle of list / Kth node from end" -> **Two Pointers** (one moves faster, or one starts later by K).
>
> "Reverse the whole list or a sublist" -> **Iterative reverse** with `prev`, `cur`, `next` triplet.
>
> "Reverse nodes in K-sized groups" -> **Reverse + reconnect** in chunks. Use a sentinel.
>
> "Merge two sorted lists" -> **Dummy node + walk both** with `<=` comparison.
>
> "Merge K sorted lists" -> **Min-heap of heads** OR **divide-and-conquer pairwise merge**.
>
> "Remove Nth from end (single pass)" -> **Two-pointer with gap of N**.
>
> "Intersection of two lists" -> **Length-difference walk** OR **"redirect at end" trick** (a + b = b + a).
>
> "Deep copy list with random pointer" -> **HashMap original->copy** (two passes) OR **interleave clones**.
>
> "Reorder list / palindrome check" -> **Find middle + reverse second half + merge/compare**.
>
> "Add two numbers (as linked lists)" -> **Walk both with carry**. Handle different lengths.
>
> "Partition list around a value" -> **Two dummy lists** (less + greater), then stitch.
>
> "Need to remove edge cases for first/last node" -> **Sentinel / dummy node**.

Everything else in linked-list-land is a variant or combination of these.

---

## Quick Lookup Table (read this first when revisiting after a gap)

| # | Pattern | Trigger phrases | Anchor problem | Core idea |
|---|---|---|---|---|
| 1 | Slow/Fast Pointer (Floyd's) | "detect cycle", "find cycle start", "happy number", "middle of list" | Linked List Cycle | Slow moves 1, fast moves 2; if they meet, cycle exists |
| 2 | Two Pointer with Gap | "Nth from end", "find middle", "Kth from end" | Remove Nth Node From End | First pointer advances N steps, then both move together |
| 3 | Iterative Reverse | "reverse linked list", "reverse sublist", "reverse between m and n" | Reverse Linked List | `prev`, `cur`, `next` triplet; cur.next = prev, then advance |
| 4 | Reverse in K-Groups | "reverse nodes in K-group", "swap nodes in pairs" | Reverse Nodes in k-Group | Verify K available; reverse K nodes; reconnect; recurse/iterate |
| 5 | Merge Two Sorted | "merge two sorted lists", "merge sorted list" | Merge Two Sorted Lists | Dummy node; walk both lists with `<=` |
| 6 | Merge K Sorted | "merge K sorted lists", "smallest from K sources" | Merge k Sorted Lists | Min-heap of heads (O(N log K)), or pairwise merge (O(N log K)) |
| 7 | Deep Copy w/ Random | "clone list with random pointer", "deep copy graph-like list" | Copy List with Random Pointer | HashMap original -> copy (two passes), OR interleave clones |
| 8 | Reorder / Palindrome | "reorder list (L0->Ln->L1->Ln-1...)", "palindrome linked list" | Reorder List | Find middle, reverse second half, merge / compare |
| 9 | Intersection of Two Lists | "find intersection node", "Y-shape lists" | Intersection of Two Linked Lists | Walk a+b vs b+a; meet at intersection or null |
| 10 | Add Numbers (Linked) | "add two numbers as linked lists", "multiply linked lists" | Add Two Numbers | Walk both; track carry; pad shorter with 0 |
| 11 | Partition List | "partition list around value X", "all less than X first" | Partition List | Two dummy lists (lessHead, greaterHead); stitch |
| 12 | Doubly Linked (LRU) | "design LRU cache", "constant-time insert and remove" | LRU Cache | Hash map (key -> node) + doubly linked list with sentinels |
| 13 | Sentinel / Dummy Node | "delete head", "modify boundary", "merge with optional first" | (used in 5, 11, others) | Avoids special-casing head; always return `dummy.next` |

---

## 1. Slow/Fast Pointer (Floyd's Tortoise & Hare)

**When to use it:** Cycle detection in a linked list (or any structure with `next`-style traversal - e.g., `nums[nums[i]]`). Also for finding the middle of a list in one pass, or detecting a cycle in number sequences (happy number).

**Trigger phrases:** "detect cycle in linked list", "find where cycle starts", "is happy number", "linked list cycle II", "middle of linked list".

**Anchor problem:** Linked List Cycle.

**Template (detect cycle - returns true if cycle exists):**
```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

**Template (find cycle start - Floyd's classic):**
```java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) break;
    }
    if (fast == null || fast.next == null) return null;     // no cycle

    // Phase 2: reset slow to head; walk both one step at a time
    slow = head;
    while (slow != fast) {
        slow = slow.next;
        fast = fast.next;
    }
    return slow;                                            // cycle start
}
```
**Intuition (why phase 2 works):** When slow and fast meet, slow has walked `L` steps where L = distance from head to cycle start + distance INTO the cycle. Fast has walked `2L`. The difference (L) equals some multiple of the cycle length. From the meeting point, walking L more steps lands back at the cycle start. So resetting one pointer to head and walking both at speed 1 -> they meet at the cycle start.

**Template (find middle of list):**
```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
}
return slow;        // for even count, slow lands at the SECOND middle (1->2->3->4 => 3)
```
For first-middle in even count (1->2->3->4 => 2): start `fast = head.next`.

**Variations:**
- **Happy Number**: same slow/fast on the `sumOfSquares` function instead of `.next`. Cycle = unhappy.
- **Find Duplicate Number** (n+1 integers in [1..n]): treat `nums` as a linked list with `nums[i]` as next pointer; Floyd's finds the duplicate as the cycle entry.
- **Linked List Cycle Length**: after detecting cycle, fix slow; advance fast and count until fast meets slow again.

**Common bugs:**
- Loop condition - need BOTH `fast != null && fast.next != null` to avoid NPE on `fast.next.next`.
- For "find middle", forgetting to specify first vs second middle - 4 nodes have TWO middles; pick deliberately.
- Starting slow and fast at different positions confuses Phase 2 of Floyd's. Keep both at `head`.

---

## 2. Two Pointer with Gap (Kth from end)

**When to use it:** Find / remove a node at a position counted FROM THE END, in a SINGLE pass. The first pointer advances K steps; then both move together until first reaches end.

**Trigger phrases:** "remove Nth node from end", "Kth from last", "find the Nth-to-last node".

**Anchor problem:** Remove Nth Node From End of List.

**Template:**
```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0, head);
    ListNode first = dummy, second = dummy;

    for (int i = 0; i < n + 1; i++) first = first.next;   // gap of n + 1
    while (first != null) {
        first = first.next;
        second = second.next;
    }
    second.next = second.next.next;                        // skip the target node
    return dummy.next;
}
```
**Why gap of n + 1, not n?** We want `second` to land at the node BEFORE the one to remove, so we can do `second.next = second.next.next`.

**Variations:**
- **Find Middle of Linked List**: gap of `length / 2` (or use Floyd's slow/fast).
- **Delete the Middle Node**: find middle (slow/fast), unlink it.

**Common bugs:**
- Off-by-one on the gap - `n` vs `n + 1`. Always use `dummy` + gap of `n + 1` for "remove Nth" to avoid head-deletion edge case.
- Not using a dummy when the first node might be removed (head deletion).

---

## 3. Iterative Reverse

**When to use it:** Reverse the entire list, or a sublist. THE bread-and-butter linked list manipulation; many higher-level patterns build on this.

**Trigger phrases:** "reverse linked list", "reverse list between m and n", "reverse second half".

**Anchor problem:** Reverse Linked List.

**Template (iterative - reverse the whole list):**
```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null, cur = head;
    while (cur != null) {
        ListNode next = cur.next;        // save
        cur.next = prev;                 // reverse
        prev = cur;                       // advance
        cur = next;
    }
    return prev;                          // new head
}
```

**Template (recursive - elegant but uses stack space):**
```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode newHead = reverseList(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;
}
```

**Template (reverse between positions m and n, 1-indexed):**
```java
public ListNode reverseBetween(ListNode head, int m, int n) {
    ListNode dummy = new ListNode(0, head);
    ListNode beforeM = dummy;
    for (int i = 1; i < m; i++) beforeM = beforeM.next;     // node before position m

    ListNode prev = null, cur = beforeM.next;
    for (int i = 0; i <= n - m; i++) {                       // reverse n - m + 1 nodes
        ListNode next = cur.next;
        cur.next = prev;
        prev = cur;
        cur = next;
    }

    // Reconnect: beforeM -> last of reversed (prev) -> ... -> first of reversed -> cur
    beforeM.next.next = cur;             // first-of-reversed (still pointing at original cur position)
    beforeM.next = prev;                 // beforeM now points to NEW first (last original)
    return dummy.next;
}
```

**Variations:**
- **Reverse Linked List II** (above): m to n inclusive.
- **Palindrome Linked List** (use this + slow/fast in pattern 8).
- **Reorder List** (use this + slow/fast in pattern 8).

**Common bugs:**
- Forgetting to save `next` BEFORE reversing - lose the rest of the list.
- For "reverse between m, n": forgetting to fix `beforeM.next.next` (the new tail's `next`) - the reversed segment becomes disconnected from the rest.
- Off-by-one in reverse-count loop: reversing nodes from m to n inclusive is `n - m + 1` nodes, so loop runs `n - m + 1` times (i.e., `i <= n - m` or `i < n - m + 1`).

---

## 4. Reverse in K-Groups

**When to use it:** Reverse linked list nodes in groups of size K. If fewer than K nodes remain at the end, leave them as-is.

**Trigger phrases:** "reverse nodes in K-group", "swap nodes in pairs" (K=2), "rotate list by K".

**Anchor problem:** Reverse Nodes in k-Group.

**Template (iterative):**
```java
public ListNode reverseKGroup(ListNode head, int k) {
    ListNode dummy = new ListNode(0, head);
    ListNode groupPrev = dummy;

    while (true) {
        ListNode kth = getKth(groupPrev, k);
        if (kth == null) break;                     // fewer than k remain - stop
        ListNode groupNext = kth.next;

        // Reverse this group: from groupPrev.next to kth
        ListNode prev = groupNext;
        ListNode cur = groupPrev.next;
        while (cur != groupNext) {
            ListNode next = cur.next;
            cur.next = prev;
            prev = cur;
            cur = next;
        }

        ListNode tmp = groupPrev.next;             // becomes new tail of reversed group
        groupPrev.next = kth;                       // new head of group
        groupPrev = tmp;                            // advance to next group's prev
    }
    return dummy.next;
}

private ListNode getKth(ListNode start, int k) {
    while (start != null && k > 0) { start = start.next; k--; }
    return start;
}
```

**Template (Swap Nodes in Pairs - K = 2, cleaner):**
```java
public ListNode swapPairs(ListNode head) {
    ListNode dummy = new ListNode(0, head);
    ListNode prev = dummy;
    while (prev.next != null && prev.next.next != null) {
        ListNode a = prev.next, b = a.next;
        a.next = b.next;
        b.next = a;
        prev.next = b;
        prev = a;
    }
    return dummy.next;
}
```

**Variations:**
- **Rotate List by K**: count length, find new tail at position `length - K`, splice.
- **Reverse Nodes in Even Length Groups**: variant of K-group with dynamic K per group.

**Common bugs:**
- Not checking that K nodes are actually available before reversing - reverses a partial group.
- Losing the `groupNext` pointer - have to walk to find it again.
- Mixing up which is `groupPrev` (before group) vs new tail (the old `groupPrev.next`).

---

## 5. Merge Two Sorted Lists

**When to use it:** Combine two sorted linked lists into one sorted list. Foundation for merge K (pattern 6) and merge sort on linked lists.

**Trigger phrases:** "merge two sorted lists", "merge sorted lists in place", "combine two ordered lists".

**Anchor problem:** Merge Two Sorted Lists.

**Template (iterative with dummy):**
```java
public ListNode mergeTwoLists(ListNode a, ListNode b) {
    ListNode dummy = new ListNode(0);
    ListNode tail = dummy;
    while (a != null && b != null) {
        if (a.val <= b.val) {
            tail.next = a;
            a = a.next;
        } else {
            tail.next = b;
            b = b.next;
        }
        tail = tail.next;
    }
    tail.next = (a != null) ? a : b;            // append remaining
    return dummy.next;
}
```

**Template (recursive - elegant but stack-heavy for long lists):**
```java
public ListNode mergeTwoLists(ListNode a, ListNode b) {
    if (a == null) return b;
    if (b == null) return a;
    if (a.val <= b.val) { a.next = mergeTwoLists(a.next, b); return a; }
    else                { b.next = mergeTwoLists(a, b.next); return b; }
}
```

**Variations:**
- **Sort List** (merge sort on linked list): use slow/fast to split, recurse, merge.
- **Merge Sorted Lists** is a building block; many higher-level problems decompose into it.

**Common bugs:**
- Using `<` instead of `<=` - for stable order with duplicates, `<=` keeps `a`'s element first (matters when nodes carry extra metadata).
- Forgetting `tail.next = (a != null) ? a : b` - leaves the merged list truncated.
- Recursive version on lists of 10000+ - StackOverflowError.

---

## 6. Merge K Sorted Lists

**When to use it:** Combine K sorted linked lists. Naive (concat + sort) is O(N log N). With min-heap: O(N log K). Pairwise divide-and-conquer: also O(N log K) and no heap overhead.

**Trigger phrases:** "merge K sorted lists", "merge K sorted arrays", "smallest element from K streams".

**Anchor problem:** Merge k Sorted Lists.

**Template (min-heap - O(N log K)):**
```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>(Comparator.comparingInt(n -> n.val));
    for (ListNode head : lists) {
        if (head != null) pq.offer(head);
    }

    ListNode dummy = new ListNode(0), tail = dummy;
    while (!pq.isEmpty()) {
        ListNode min = pq.poll();
        tail.next = min;
        tail = tail.next;
        if (min.next != null) pq.offer(min.next);
    }
    return dummy.next;
}
```

**Template (divide-and-conquer pairwise merge - O(N log K), no heap):**
```java
public ListNode mergeKLists(ListNode[] lists) {
    if (lists.length == 0) return null;
    int interval = 1;
    int n = lists.length;
    while (interval < n) {
        for (int i = 0; i + interval < n; i += interval * 2) {
            lists[i] = mergeTwoLists(lists[i], lists[i + interval]);
        }
        interval *= 2;
    }
    return lists[0];
}
```

**Variations:**
- **Smallest Range Covering Elements from K Lists**: heap of indices into K lists; track current min and max in window.
- **Kth Smallest Element in K Sorted Lists**: heap-based, advance K-1 times.

**Common bugs:**
- For heap version: forgetting null check before pushing initial heads (NPE on `comparator(null)`).
- For divide-and-conquer: off-by-one in `interval` doubling - mishandles odd K.
- For very long lists: comparator overhead in heap (boxing) can dominate; profile if needed.

---

## 7. Deep Copy with Random Pointer

**When to use it:** Clone a linked list where each node has an extra `random` pointer to ANY node (or null). The challenge: random pointers form a graph, not a tree.

**Trigger phrases:** "copy list with random pointer", "deep copy graph-like list", "clone with arbitrary pointer".

**Anchor problem:** Copy List with Random Pointer.

**Template (HashMap - two passes, O(N) space):**
```java
public Node copyRandomList(Node head) {
    if (head == null) return null;
    Map<Node, Node> map = new HashMap<>();

    // Pass 1: create copies of all nodes
    Node cur = head;
    while (cur != null) {
        map.put(cur, new Node(cur.val));
        cur = cur.next;
    }
    // Pass 2: wire up next + random
    cur = head;
    while (cur != null) {
        map.get(cur).next = map.get(cur.next);
        map.get(cur).random = map.get(cur.random);
        cur = cur.next;
    }
    return map.get(head);
}
```

**Template (interleave clones - O(1) extra space):**
```java
public Node copyRandomList(Node head) {
    if (head == null) return null;

    // Pass 1: interleave clones - A -> A' -> B -> B' -> C -> C'
    Node cur = head;
    while (cur != null) {
        Node clone = new Node(cur.val);
        clone.next = cur.next;
        cur.next = clone;
        cur = clone.next;
    }
    // Pass 2: set random pointers on clones
    cur = head;
    while (cur != null) {
        if (cur.random != null) cur.next.random = cur.random.next;     // clone of cur.random is its next
        cur = cur.next.next;
    }
    // Pass 3: separate the two interleaved lists
    Node dummy = new Node(0), copyTail = dummy;
    cur = head;
    while (cur != null) {
        copyTail.next = cur.next;
        copyTail = copyTail.next;
        cur.next = cur.next.next;                                       // restore original
        cur = cur.next;
    }
    return dummy.next;
}
```

**Common bugs:**
- HashMap: forgetting null check when wiring `random` (random can be null).
- Interleave version: not restoring the original list's `next` - corrupts input.

---

## 8. Reorder / Palindrome (Find Middle + Reverse + Merge/Compare)

**When to use it:** Problems requiring reordering or symmetry checks on a linked list. Standard technique: find middle, reverse the second half, then walk both halves.

**Trigger phrases:** "palindrome linked list", "reorder list (L0->Ln->L1->Ln-1...)", "is symmetric list".

**Anchor problem:** Reorder List.

**Template (Reorder List):**
```java
public void reorderList(ListNode head) {
    if (head == null || head.next == null) return;

    // 1. Find middle (slow/fast)
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    // 2. Reverse second half
    ListNode second = reverseList(slow.next);
    slow.next = null;                                  // split into two lists

    // 3. Merge alternately
    ListNode first = head;
    while (second != null) {
        ListNode tmp1 = first.next, tmp2 = second.next;
        first.next = second;
        second.next = tmp1;
        first = tmp1;
        second = tmp2;
    }
}
```

**Template (Palindrome Linked List):**
```java
public boolean isPalindrome(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    ListNode second = reverseList(slow);

    ListNode first = head;
    while (second != null) {
        if (first.val != second.val) return false;
        first = first.next;
        second = second.next;
    }
    return true;
}
```

**Variations:**
- **Reorder List**: above.
- **Palindrome Linked List**: above.
- **Sort List** (merge sort): same find-middle + recurse + merge pattern.

**Common bugs:**
- For palindrome with even length: slow lands on first-middle - reversing from `slow` gives equal halves. With odd length, slow lands on middle - reversing from `slow` includes the middle in both, which is fine.
- For reorder: forgetting `slow.next = null` - the first half's tail still points into the (now-reversed) second half -> infinite loop.

---

## 9. Intersection of Two Linked Lists

**When to use it:** Two lists merge at some point (Y-shape). Find the node where they intersect.

**Trigger phrases:** "intersection of two linked lists", "find common node", "Y-shaped lists".

**Anchor problem:** Intersection of Two Linked Lists.

**Template (length-difference walk):**
```java
public ListNode getIntersectionNode(ListNode a, ListNode b) {
    int lenA = length(a), lenB = length(b);
    while (lenA > lenB) { a = a.next; lenA--; }
    while (lenB > lenA) { b = b.next; lenB--; }
    while (a != b) { a = a.next; b = b.next; }
    return a;     // null if no intersection
}
private int length(ListNode n) {
    int len = 0;
    while (n != null) { len++; n = n.next; }
    return len;
}
```

**Template (elegant trick - swap heads at end, two passes):**
```java
public ListNode getIntersectionNode(ListNode a, ListNode b) {
    ListNode p1 = a, p2 = b;
    while (p1 != p2) {
        p1 = (p1 == null) ? b : p1.next;
        p2 = (p2 == null) ? a : p2.next;
    }
    return p1;
}
```
**Intuition:** if lengths differ by `d`, after switching heads both pointers will have walked `a + b` nodes by the time they meet (or both hit null).

**Common bugs:**
- For length-difference: forgetting the second loop (walking together) - returns the wrong node.
- For elegant trick: using `p1.next` instead of `b` on switch - infinite loop.

---

## 10. Add Numbers as Linked Lists

**When to use it:** Numbers represented digit-by-digit in a linked list (typically least-significant digit first). Walk both, track carry.

**Trigger phrases:** "add two numbers (linked list)", "multiply linked lists", "linked list digit arithmetic".

**Anchor problem:** Add Two Numbers.

**Template:**
```java
public ListNode addTwoNumbers(ListNode a, ListNode b) {
    ListNode dummy = new ListNode(0), tail = dummy;
    int carry = 0;
    while (a != null || b != null || carry > 0) {
        int sum = carry;
        if (a != null) { sum += a.val; a = a.next; }
        if (b != null) { sum += b.val; b = b.next; }
        carry = sum / 10;
        tail.next = new ListNode(sum % 10);
        tail = tail.next;
    }
    return dummy.next;
}
```

**Variations:**
- **Add Two Numbers II** (most-significant digit first): reverse both, add, reverse result. OR use stacks.
- **Plus One Linked List**: traverse, propagate carry from end backwards (reverse trick or recursion).
- **Multiply Strings** (similar idea on strings, not linked lists): conventional long multiplication.

**Common bugs:**
- Forgetting the carry check `|| carry > 0` in the loop condition - drops final carry on overflow (e.g., 99 + 1).
- Reading `a.val` without checking `a != null` first.

---

## 11. Partition List

**When to use it:** Rearrange a list so all nodes less than X come before all nodes >= X, preserving relative order.

**Trigger phrases:** "partition list around value X", "rearrange nodes around pivot".

**Anchor problem:** Partition List.

**Template (two dummy lists, then stitch):**
```java
public ListNode partition(ListNode head, int x) {
    ListNode lessHead = new ListNode(0), lessTail = lessHead;
    ListNode greaterHead = new ListNode(0), greaterTail = greaterHead;

    while (head != null) {
        if (head.val < x) {
            lessTail.next = head; lessTail = lessTail.next;
        } else {
            greaterTail.next = head; greaterTail = greaterTail.next;
        }
        head = head.next;
    }
    greaterTail.next = null;                       // CRITICAL - terminate, else cycle
    lessTail.next = greaterHead.next;
    return lessHead.next;
}
```

**Variations:**
- **Odd Even Linked List**: same skeleton with odd/even index grouping.
- **Reorder by Some Predicate**: substitute the condition.

**Common bugs:**
- Forgetting `greaterTail.next = null` - last node's original `next` still points into the less-list -> cycle or wrong order.

---

## 12. Doubly Linked List (LRU and friends)

**When to use it:** O(1) insert/remove anywhere in the list, given a node reference. Typically combined with HashMap for an O(1) lookup-by-key.

**Trigger phrases:** "design LRU cache", "design LFU cache", "constant-time insert and remove from middle".

**Anchor problem:** LRU Cache (covered in detail in `hashmap-patterns.md`).

**Template (DLL with sentinels - removes head/tail edge cases):**
```java
class DLLNode {
    int key, val;
    DLLNode prev, next;
    DLLNode(int k, int v) { key = k; val = v; }
}

DLLNode head = new DLLNode(0, 0), tail = new DLLNode(0, 0);
// init:
head.next = tail;
tail.prev = head;

void addToFront(DLLNode n) {
    n.next = head.next;
    n.prev = head;
    head.next.prev = n;
    head.next = n;
}
void remove(DLLNode n) {
    n.prev.next = n.next;
    n.next.prev = n.prev;
}
void moveToFront(DLLNode n) { remove(n); addToFront(n); }
DLLNode removeLast() {                          // for eviction
    DLLNode n = tail.prev;
    if (n == head) return null;                  // empty
    remove(n);
    return n;
}
```

**Variations:**
- **LRU Cache** (pattern 9 in hashmap-patterns).
- **LFU Cache**: DLL per frequency bucket; HashMap key->node; HashMap freq->bucket.
- **Browser History**: DLL with pointer to current.

**Common bugs:**
- Not using sentinels - head/tail removal needs special-case logic.
- For LRU: forgetting to also remove from the HashMap on eviction.

---

## 13. Sentinel / Dummy Node

**When to use it:** Whenever the first node might be removed, replaced, or the head might be null - dummy avoids special-case logic. Always returns `dummy.next`.

**Trigger phrases:** Implicit - use any time a head modification is possible.

**Anchor problems:** Used in Merge Two Sorted Lists, Remove Nth from End, Partition List, Reverse Between, Reorder List, basically half the linked-list catalog.

**Pattern:**
```java
ListNode dummy = new ListNode(0);
dummy.next = head;
ListNode cur = dummy;
// ... operate on dummy.next, never on head directly ...
return dummy.next;
```

**Benefits:**
- No special case for empty list or head deletion.
- Uniform `prev.next = ...` instead of `if (prev == null) head = ... else prev.next = ...`.

**Cost:** one extra node allocation (negligible).

**Common bugs:**
- Forgetting to return `dummy.next` - returns the sentinel itself.
- Returning `head` after dummy-based operations - `head` might have been removed.

---

## Suggested revision order if relearning from scratch

1. **Sentinel / Dummy Node (pattern 13)** - foundational technique used by many others. Internalize first.
2. **Iterative Reverse (pattern 3)** - classic; prev/cur/next triplet is the most reused snippet.
3. **Merge Two Sorted (pattern 5)** - uses dummy; trivial; foundation for pattern 6.
4. **Slow/Fast Pointer (pattern 1)** - find middle, cycle detection. Floyd's classic.
5. **Two Pointer with Gap (pattern 2)** - remove Nth from end (one pass).
6. **Reorder / Palindrome (pattern 8)** - composes patterns 1 + 3 + 5.
7. **Merge K Sorted (pattern 6)** - heap variant first, then divide-and-conquer.
8. **Reverse in K-Groups (pattern 4)** - the hardest pure linked-list problem; many bugs.
9. **Intersection of Two Lists (pattern 9)** - both length-diff and elegant tricks.
10. **Add Numbers (pattern 10)** - easy; carry handling is the only trap.
11. **Partition List (pattern 11)** - simple two-list-stitch.
12. **Deep Copy with Random (pattern 7)** - HashMap version first; interleave is a bonus.
13. **Doubly Linked (pattern 12)** - feeds directly into LRU/LFU design.

**Universal traps across every pattern:**
- **NullPointerException on `.next`** - always check `cur != null` before `cur.next`; double-check for `cur.next.next`.
- **Losing the rest of the list** - when modifying `cur.next`, save the original `cur.next` first (e.g., `ListNode next = cur.next`).
- **Forgetting to terminate** - leftover `next` pointers from previous structure cause cycles. Explicitly set `last.next = null`.
- **Head modification without dummy** - special-cases everywhere. Always use a dummy if the head can change.
- **Recursive solutions on huge lists** - StackOverflowError around 10k-50k nodes. Convert to iterative for production.
- **Comparing values vs nodes** - `node1 == node2` is reference identity (you usually WANT this for linked lists). `node1.val == node2.val` is content. Pick deliberately.
- **Cycle from incorrect rewiring** - common with reverse-between and K-group. Draw it out for K = 3 with a 6-node list before coding.
- **`for (ListNode n : list)`** doesn't exist - linked lists aren't Iterable. Use `while (n != null) ... n = n.next;`.
