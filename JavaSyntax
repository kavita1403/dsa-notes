┌─────────────────────────────────────────────────────────────────┐
│                  COMPARATOR CHEAT SHEET                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Want SMALLER first?  → Integer.compare(a, b)   ← a before b  │
│  Want BIGGER  first?  → Integer.compare(b, a)   ← swap!       │
│                                                                 │
│  Integers:   (a, b) -> Integer.compare(a, b)                   │
│  Strings:    (a, b) -> a.compareTo(b)                          │
│  By field:   (a, b) -> Integer.compare(a[0], b[0])             │
│  By map val: (a, b) -> Integer.compare(mp.get(a), mp.get(b))   │
│                                                                 │
│  TO REVERSE ANYTHING → just swap a and b inside compare()      │
│                                                                 │
│  PQ default = MIN HEAP (no comparator needed)                  │
│  PQ max heap = (a, b) -> Integer.compare(b, a)                 │
│                                                                 │
│  NEVER use a - b  →  use Integer.compare(a, b)  (no overflow)  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘


# Java Interview Cheat Sheet

## Table of Contents
1. [Comparator](#1-comparator)
2. [HashMap / TreeMap](#2-hashmap--treemap)
3. [Set (Hash / Tree / Linked)](#3-set-hash--tree--linked)
4. [List / ArrayList](#4-list--arraylist)
5. [Deque / Stack / Queue](#5-deque--stack--queue)
6. [PriorityQueue (Heap)](#6-priorityqueue-heap)
7. [String Methods](#7-string-methods)
8. [Arrays](#8-arrays)
9. [Loops & Control Flow](#9-loops--control-flow)
10. [Lambdas & Functional Interfaces](#10-lambdas--functional-interfaces)
11. [Streams](#11-streams)
12. [Common Gotchas](#12-common-gotchas)
13. [Time Complexity Reference](#13-time-complexity-reference)

---

## 1. Comparator

**Golden rule:** smaller value first → `Integer.compare(a, b)`. Want it reversed? Just swap `a` and `b`.

| Goal | Code |
|---|---|
| Integers | `(a, b) -> Integer.compare(a, b)` |
| Strings | `(a, b) -> a.compareTo(b)` |
| By array field | `(a, b) -> Integer.compare(a[0], b[0])` |
| By map value | `(a, b) -> Integer.compare(mp.get(a), mp.get(b))` |
| Reverse anything | swap `a` and `b` inside `compare()` |

> ⚠️ Never use `a - b` for sorting — it can overflow. Always use `Integer.compare(a, b)`.

**PriorityQueue defaults:** min-heap by default (no comparator needed). Max-heap → `(a, b) -> Integer.compare(b, a)`.

**Sorting a List**
```java
Collections.sort(list);                                     // natural order
Collections.sort(list, Collections.reverseOrder());
list.sort((a, b) -> a.compareTo(b));
Collections.sort(list, Comparator.comparing(Person::getAge));
Collections.sort(list, Comparator.comparing(Person::getAge)
                                  .thenComparing(Person::getName));
Collections.sort(list, Comparator.comparing(Person::getAge).reversed());
```

**Sorting a 2D array**
```java
Arrays.sort(arr, (a, b) -> Integer.compare(a[0], b[0]));    // safe, no overflow
```

---

## 2. HashMap / TreeMap

```java
Map<String, Integer> mp = new HashMap<>();
```

**Basics**
```java
mp.put(key, val);
mp.get(key);                       // null if absent
mp.getOrDefault(key, 0);           // safe get
mp.containsKey(key);
mp.containsValue(val);
mp.remove(key);
mp.size();
mp.isEmpty();
```

**Frequency counting (very common in interviews)**
```java
mp.put(key, mp.getOrDefault(key, 0) + 1);
mp.merge(key, 1, Integer::sum);                             // same thing, one-liner
```

**Insert-if-absent / grouping**
```java
mp.putIfAbsent(key, val);
mp.computeIfAbsent(key, k -> new ArrayList<>()).add(val);   // map of lists
mp.compute(key, (k, v) -> (v == null) ? 1 : v + 1);
```

**Iterating**

| Need | Use |
|---|---|
| Only the keys | `mp.keySet()` |
| Only the values | `mp.values()` |
| Both key and value | `mp.entrySet()` (fastest — avoids a `.get()` lookup per key) |

```java
// keySet() — only keys
for (String key : mp.keySet()) {
    System.out.println(key);
}

// values() — only values
for (Integer val : mp.values()) {
    System.out.println(val);
}

// entrySet() — both key and value together (preferred when you need both)
for (Map.Entry<String, Integer> e : mp.entrySet()) {
    String k = e.getKey();
    Integer v = e.getValue();
}

// forEach — lambda shortcut, same as entrySet() under the hood
mp.forEach((k, v) -> System.out.println(k + " " + v));
```

> If you only have the key and need the value too, avoid `for (String key : mp.keySet()) { mp.get(key); }` — that's an extra lookup per key. Use `entrySet()` instead.

**TreeMap — keys sorted automatically**
```java
TreeMap<Integer, String> tm = new TreeMap<>();
tm.firstKey();  tm.lastKey();
tm.ceilingKey(x);  tm.floorKey(x);    // >= x , <= x
tm.higherKey(x);   tm.lowerKey(x);    // > x , < x
```

**LinkedHashMap** — preserves insertion order: `Map<K,V> lhm = new LinkedHashMap<>();`

---

## 3. Set (Hash / Tree / Linked)

```java
Set<Integer> set = new HashSet<>();
set.add(x);  set.remove(x);  set.contains(x);  set.size();
```

```java
TreeSet<Integer> ts = new TreeSet<>();       // sorted, no duplicates
ts.first();  ts.last();
ts.ceiling(x);  ts.floor(x);  ts.higher(x);  ts.lower(x);
```

```java
Set<Integer> lhs = new LinkedHashSet<>();    // sorted by insertion order
```

---

## 4. List / ArrayList

```java
List<Integer> list = new ArrayList<>();
list.add(x);              list.add(index, x);
list.get(index);          list.set(index, x);
list.remove(index);                     // removes BY INDEX
list.remove(Integer.valueOf(x));        // removes BY VALUE
list.contains(x);
list.size();
list.isEmpty();
Collections.reverse(list);
Collections.max(list);   Collections.min(list);
List<Integer> copy = new ArrayList<>(list);
```

**Array ↔ List conversion**
```java
Integer[] arr = list.toArray(new Integer[0]);
List<Integer> l  = Arrays.asList(arr);                       // fixed-size, careful!
List<Integer> l2 = new ArrayList<>(Arrays.asList(arr));      // resizable
```

---

## 5. Deque / Stack / Queue

> Use `ArrayDeque`, not the legacy `Stack` class — it's slower.

```java
Deque<Integer> stack = new ArrayDeque<>();
stack.push(x);  stack.pop();  stack.peek();  stack.isEmpty();

Deque<Integer> queue = new ArrayDeque<>();
queue.offer(x);  queue.poll();  queue.peek();

Deque<Integer> deque = new ArrayDeque<>();
deque.addFirst(x);  deque.addLast(x);
deque.pollFirst();  deque.pollLast();
deque.peekFirst();  deque.peekLast();
```

---

## 6. PriorityQueue (Heap)

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();               // default min-heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);
PriorityQueue<Integer> maxHeap2 = new PriorityQueue<>(Collections.reverseOrder());

pq.offer(x);  pq.add(x);
pq.poll();      // removes & returns top
pq.peek();      // just returns top
pq.isEmpty();

// PQ of arrays / custom objects
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
```

---

## 7. String Methods

```java
s.length();
s.charAt(i);
s.substring(start, end);          // end is EXCLUSIVE
s.indexOf("x");   s.lastIndexOf("x");
s.contains("x");
s.equals(other);  s.equalsIgnoreCase(other);   // NEVER use == for strings!
s.compareTo(other);
s.toCharArray();
s.split(",");                     // regex-based split
s.trim();  s.strip();
s.toUpperCase();  s.toLowerCase();
s.replace("a", "b");
String.join(",", list);
s.isEmpty();  s.isBlank();
```

**StringBuilder** — mutable, use inside loops
```java
StringBuilder sb = new StringBuilder();
sb.append(x);
sb.reverse();
sb.deleteCharAt(i);
sb.insert(i, x);
sb.setCharAt(i, c);
sb.charAt(i);
sb.toString();
```

---

## 8. Arrays

```java
int[] arr = new int[n];
int[][] grid = new int[rows][cols];

Arrays.sort(arr);                     // primitives, ascending only
Arrays.sort(arr, from, to);           // sort a range
Arrays.fill(arr, val);
Arrays.toString(arr);                 // for printing
Arrays.deepToString(grid);            // 2D printing
Arrays.equals(arr1, arr2);
Arrays.copyOf(arr, newLength);
Arrays.copyOfRange(arr, from, to);
Arrays.asList(boxedArr);              // needs Integer[], not int[]
```

> `Arrays.sort()` can't reverse-sort a primitive `int[]` directly. Box it first (`Integer[]`), then use `Comparator.reverseOrder()`.

---

## 9. Loops & Control Flow

```java
for (int i = 0; i < n; i++) { }
for (int i = n - 1; i >= 0; i--) { }
for (int x : arr) { }                 // enhanced for-each
while (cond) { }
do { } while (cond);

switch (x) {
    case 1 -> System.out.println("one");   // modern arrow syntax
    case 2 -> System.out.println("two");
    default -> System.out.println("other");
}

label: for (...) {
    for (...) {
        break label;       // break out of outer loop
        continue label;
    }
}
```

---

## 10. Lambdas & Functional Interfaces

```java
Runnable r = () -> System.out.println("run");
Comparator<Integer> c = (a, b) -> a - b;
Function<Integer, Integer> square = x -> x * x;
Predicate<Integer> isEven = x -> x % 2 == 0;
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;

// Method reference shortcuts
list.forEach(System.out::println);
list.sort(Comparator.comparing(Person::getName));
```

---

## 11. Streams

```java
list.stream().filter(x -> x > 5).collect(Collectors.toList());
list.stream().map(x -> x * 2).collect(Collectors.toList());
list.stream().sorted().collect(Collectors.toList());
list.stream().mapToInt(Integer::intValue).sum();   // note: mapToInt needed for sum()
list.stream().max(Integer::compare);
list.stream().count();
list.stream().anyMatch(x -> x > 5);
list.stream().collect(Collectors.groupingBy(x -> x % 2));    // -> Map<Integer, List<Integer>>
list.stream().collect(Collectors.toMap(x -> x, x -> x * x));
```

---

## 12. Common Gotchas

- Use `.equals()` for object comparison, **never `==`** (except primitives).
- Integer caching: `==` only works reliably for values **-128 to 127** — a classic interview trap.
- `Arrays.asList()` returns a **fixed-size** list — `.add()`/`.remove()` throws an error.
- `String` is immutable — concatenating in a loop creates a new object each time → use `StringBuilder`.
- `HashMap` iteration order is **not guaranteed** — use `LinkedHashMap`/`TreeMap` if order matters.
- Autoboxing NPE trap: `Integer x = map.get(key);` — if the key is absent, `x` is `null` → NPE on `x == 5`.
- `ArrayDeque` is faster than `Stack`/`LinkedList` for stack/queue use — prefer it.
- Modifying a `List` while iterating with for-each → `ConcurrentModificationException` (use `Iterator.remove()`, or iterate over a copy).

---

## 13. Time Complexity Reference

| Structure | Operation | Complexity |
|---|---|---|
| HashMap / HashSet | get / put / contains | O(1) avg |
| TreeMap / TreeSet | get / put / contains | O(log n) |
| ArrayList | get by index | O(1) |
| ArrayList | add/remove at end | O(1) amortized |
| LinkedList | add/remove at ends | O(1) |
| LinkedList | get by index | O(n) |
| PriorityQueue | offer / poll | O(log n) |
| PriorityQueue | peek | O(1) |
| Arrays.sort (primitives) | sort | O(n log n) — dual-pivot quicksort |
| Collections.sort (objects) | sort | O(n log n) — Timsort (stable) |
