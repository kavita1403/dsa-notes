# Linked Lists - Cheat Sheet

**Decision fork:** Detect cycle / cycle start -> Floyd's slow/fast. Find middle / Kth from end -> two pointers. Reverse all/part -> iterative prev/cur/next. Reverse in K-groups -> reverse + reconnect. Merge 2 sorted -> dummy + walk both. Merge K sorted -> min-heap of heads OR pairwise. Remove Nth from end (one pass) -> two-pointer with gap N+1. Deep copy w/ random -> HashMap OR interleave clones. Palindrome / reorder -> find middle + reverse second half + walk both. Intersection of 2 lists -> length-diff walk OR swap heads at end (a+b = b+a). Add as digits -> walk both with carry. Anything where head can change -> use a dummy.

## Sentinel / Dummy Node (THE most useful trick)
```java
ListNode dummy = new ListNode(0);
dummy.next = head;
// ... work via dummy ...
return dummy.next;
```
Removes special-casing for head deletion / empty list / merging with optional first.

## Slow/Fast Pointer (Floyd's - detect cycle)
```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow == fast) return true;
}
return false;
```
Loop condition needs BOTH `fast != null && fast.next != null` to avoid NPE on `fast.next.next`.

## Floyd's - Find Cycle Start (Linked List Cycle II)
```java
// ... slow/fast as above; break when they meet ...
slow = head;
while (slow != fast) { slow = slow.next; fast = fast.next; }
return slow;     // cycle start
```
After meeting, reset slow to head; walk both at 1; they meet at cycle entry.

## Find Middle of List
```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
}
return slow;     // even count: SECOND middle. For first-middle, start fast = head.next.
```

## Two Pointer Gap (Remove Nth from End - one pass)
```java
ListNode dummy = new ListNode(0, head), first = dummy, second = dummy;
for (int i = 0; i < n + 1; i++) first = first.next;     // gap of n + 1
while (first != null) { first = first.next; second = second.next; }
second.next = second.next.next;
return dummy.next;
```
Gap is n+1, NOT n - lands `second` at node BEFORE the one to remove.

## Iterative Reverse (the prev/cur/next dance)
```java
ListNode prev = null, cur = head;
while (cur != null) {
    ListNode next = cur.next;       // 1. save
    cur.next = prev;                 // 2. reverse
    prev = cur; cur = next;          // 3. advance
}
return prev;
```
Save `next` BEFORE reversing or you lose the rest of the list.

## Reverse Between m and n
```java
ListNode dummy = new ListNode(0, head), beforeM = dummy;
for (int i = 1; i < m; i++) beforeM = beforeM.next;
ListNode prev = null, cur = beforeM.next;
for (int i = 0; i <= n - m; i++) {                       // n - m + 1 nodes
    ListNode next = cur.next; cur.next = prev; prev = cur; cur = next;
}
beforeM.next.next = cur;       // old first becomes new tail; point to remainder
beforeM.next = prev;            // beforeM points to new first
return dummy.next;
```

## Reverse in K-Groups
```java
ListNode dummy = new ListNode(0, head), groupPrev = dummy;
while (true) {
    ListNode kth = getKth(groupPrev, k);
    if (kth == null) break;                              // fewer than k - stop
    ListNode groupNext = kth.next;
    ListNode prev = groupNext, cur = groupPrev.next;
    while (cur != groupNext) {
        ListNode next = cur.next; cur.next = prev; prev = cur; cur = next;
    }
    ListNode tmp = groupPrev.next;
    groupPrev.next = kth;
    groupPrev = tmp;
}
```

## Merge Two Sorted Lists
```java
ListNode dummy = new ListNode(0), tail = dummy;
while (a != null && b != null) {
    if (a.val <= b.val) { tail.next = a; a = a.next; }
    else                { tail.next = b; b = b.next; }
    tail = tail.next;
}
tail.next = (a != null) ? a : b;       // append remaining
return dummy.next;
```
Use `<=` for stable order with duplicates.

## Merge K Sorted Lists - Min-Heap (O(N log K))
```java
PriorityQueue<ListNode> pq = new PriorityQueue<>(Comparator.comparingInt(n -> n.val));
for (ListNode head : lists) if (head != null) pq.offer(head);

ListNode dummy = new ListNode(0), tail = dummy;
while (!pq.isEmpty()) {
    ListNode min = pq.poll();
    tail.next = min; tail = tail.next;
    if (min.next != null) pq.offer(min.next);
}
return dummy.next;
```

## Merge K Sorted - Divide & Conquer (no heap, O(N log K))
```java
int interval = 1;
while (interval < lists.length) {
    for (int i = 0; i + interval < lists.length; i += interval * 2)
        lists[i] = mergeTwoLists(lists[i], lists[i + interval]);
    interval *= 2;
}
return lists.length == 0 ? null : lists[0];
```

## Palindrome Linked List
```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) { slow = slow.next; fast = fast.next.next; }
ListNode second = reverseList(slow);
ListNode first = head;
while (second != null) {
    if (first.val != second.val) return false;
    first = first.next; second = second.next;
}
return true;
```

## Reorder List (L0->Ln->L1->Ln-1->...)
```java
// 1. find middle (slow/fast)
// 2. split: slow.next = null; reverse second half
// 3. merge alternately
ListNode first = head, second = reversedSecond;
while (second != null) {
    ListNode tmp1 = first.next, tmp2 = second.next;
    first.next = second; second.next = tmp1;
    first = tmp1; second = tmp2;
}
```
Forgetting `slow.next = null` -> infinite loop (first half's tail still points into reversed second).

## Intersection of Two Lists (elegant trick)
```java
ListNode p1 = a, p2 = b;
while (p1 != p2) {
    p1 = (p1 == null) ? b : p1.next;
    p2 = (p2 == null) ? a : p2.next;
}
return p1;
```
On switch: use `b` (or `a`), NOT `b.next` - else infinite loop. Both walk `a + b` total before meeting / both null.

## Copy List with Random Pointer (HashMap)
```java
Map<Node, Node> map = new HashMap<>();
for (Node cur = head; cur != null; cur = cur.next) map.put(cur, new Node(cur.val));
for (Node cur = head; cur != null; cur = cur.next) {
    map.get(cur).next   = map.get(cur.next);     // null-safe (map.get(null) = null)
    map.get(cur).random = map.get(cur.random);
}
return map.get(head);
```

## Add Two Numbers (least-significant digit first)
```java
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
```
Forgetting `|| carry > 0` drops final carry (99 + 1 = 100 -> missing digit).

## Partition List (< x first, >= x after)
```java
ListNode lessH = new ListNode(0), lessT = lessH;
ListNode greaterH = new ListNode(0), greaterT = greaterH;
while (head != null) {
    if (head.val < x) { lessT.next = head; lessT = lessT.next; }
    else              { greaterT.next = head; greaterT = greaterT.next; }
    head = head.next;
}
greaterT.next = null;                            // CRITICAL - else cycle
lessT.next = greaterH.next;
return lessH.next;
```

## Doubly Linked List (LRU helpers)
```java
class DLLNode { int key, val; DLLNode prev, next; }
DLLNode head = new DLLNode(), tail = new DLLNode();
head.next = tail; tail.prev = head;              // sentinels

void addToFront(DLLNode n) {
    n.next = head.next; n.prev = head;
    head.next.prev = n; head.next = n;
}
void remove(DLLNode n) {
    n.prev.next = n.next; n.next.prev = n.prev;
}
void moveToFront(DLLNode n) { remove(n); addToFront(n); }
```

## Universal traps
- **NullPointerException on `.next`**: always check `cur != null` before `cur.next`; double-check for `cur.next.next` in Floyd's.
- **Losing the rest of the list** when rewiring - save `next` BEFORE you touch `cur.next`.
- **Forgetting to terminate** with `last.next = null` - leftover pointers create cycles or wrong ordering.
- **Modifying head without dummy** - special-cases everywhere. Always use a dummy if head can change.
- **Recursive solutions on huge lists** - StackOverflowError around 10k-50k nodes. Convert to iterative.
- **Comparing `node1 == node2` vs `node1.val == node2.val`** - reference vs content. Pick deliberately (intersection uses reference).
- **`for (ListNode n : list)`** doesn't exist - linked lists aren't Iterable. `while (n != null) { ... n = n.next; }`.
