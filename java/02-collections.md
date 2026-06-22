# 02 — Collections Framework

The most-asked Java interview topic. HashMap internals = required knowledge for any senior role.

---

## Q1. Draw the Collections Framework hierarchy.

```
                       Iterable<T>
                           |
                      Collection<T>
            +---------------+---------------+
            |               |               |
         List<T>         Set<T>         Queue<T>
            |               |               |
   +--------+-----+    +----+----+     +----+----+
   |        |     |    |    |    |     |         |
ArrayList LinkedList  HashSet  TreeSet ArrayDeque PriorityQueue
Vector              LinkedHashSet      (also Deque<T>)
(legacy)            (extends HashSet)

                       Map<K,V>   (NOT a Collection!)
                           |
              +------------+------------+
              |            |            |
          HashMap     TreeMap      LinkedHashMap
              |       (Sorted/        (insertion or
       LinkedHashMap   Navigable)      access order)
        (extends)
                       Hashtable (legacy, synchronized)

          java.util.concurrent:
              ConcurrentHashMap, CopyOnWriteArrayList,
              CopyOnWriteArraySet, ConcurrentSkipListMap,
              BlockingQueue family, ConcurrentLinkedQueue
```

**Key insight**: `Map` is NOT a `Collection`. It's a separate hierarchy because key-value semantics differ fundamentally.

---

## Q2. Explain HashMap internals.

This is the single most-asked deep question. Get this right and you've passed half the interview.

### Structure (Java 8+)

```
+--------------+       +------+---+---+---+
| table[]      |  -->  | hash | k | v | next |  --> linked list (or red-black tree if >= 8)
+--------------+       +------+---+---+---+
```

- **Backing array**: `Node<K,V>[] table`
- **Initial capacity**: 16, always a power of 2 (so `hash & (n-1)` works as modulo)
- **Load factor**: 0.75 default — resize when `size > capacity * loadFactor`
- **Bucket layout**:
  - Empty (null)
  - Single Node
  - Linked list of Nodes (collisions)
  - **Red-black tree** (Java 8+) when bucket has >= 8 entries AND table size >= 64

### put(K, V) algorithm

```
1. hash = hash(key)             // (h = key.hashCode()) ^ (h >>> 16)  — spreads bits
2. i = hash & (table.length - 1)
3. if (table[i] == null) table[i] = new Node(...)
4. else walk chain/tree:
     - if key.equals(existing.key) → replace value
     - else append/insert new Node
5. ++size
6. if (size > threshold) resize()  // double capacity, rehash all
```

### Why XOR with `h >>> 16`?

Real-world hashCodes often have entropy concentrated in the upper bits. Since indexing uses only the low `log2(capacity)` bits, those upper bits would be ignored. XOR-shifting mixes the upper bits into the lower bits. Significantly reduces collisions on small tables.

### Why treeify at 8?

Statistical: with a good hash function and load factor 0.75, the probability of a bucket having 8+ entries by chance follows a Poisson distribution and is ~0.00000006. So if you're seeing it, your hashCode is bad — and a tree (O(log n)) saves you from the O(n) worst case.

### resize()

- Doubles capacity, creates new table
- Each node either stays at index `i` or moves to `i + oldCap` — determined by one bit of the hash
- O(n) operation, blocks all reads/writes (in non-concurrent HashMap)
- **This is why initial sizing matters**: `new HashMap<>(expectedSize / 0.75 + 1)` avoids resizes

###  Pitfall — mutable keys
```java
Map<List<Integer>, String> bad = new HashMap<>();
List<Integer> key = new ArrayList<>(List.of(1, 2, 3));
bad.put(key, "value");
key.add(4);                        // mutates the key after put
bad.get(key);                      // null — different hashCode, can't find it
```
Never use mutable objects as map keys. Use immutable types (records, `List.copyOf`, strings).

###  Pitfall — infinite loop in concurrent HashMap (pre-Java 8)
Pre-Java 8, concurrent puts during resize could create a circular linked list → 100% CPU forever. Java 8 changed the algorithm to not reverse the list during transfer. But **HashMap is still NOT thread-safe** — use ConcurrentHashMap.

---

## Q3. HashMap vs Hashtable vs ConcurrentHashMap vs LinkedHashMap vs TreeMap.

| | HashMap | Hashtable | ConcurrentHashMap | LinkedHashMap | TreeMap |
|---|---|---|---|---|---|
| Null key | 1 allowed | No | No | 1 allowed | No (unless custom Comparator allows) |
| Null values | Yes | No | No | Yes | Depends on Comparator |
| Thread-safe | No | Yes (synchronized everything) | Yes (lock striping / CAS) | No | No |
| Ordering | None | None | None | Insertion (or access) | Sorted (natural or Comparator) |
| Iteration | Fail-fast | Fail-fast | Fail-safe (weakly consistent) | Fail-fast | Fail-fast |
| get/put | O(1) avg | O(1) avg | O(1) avg | O(1) avg | O(log n) |

### ConcurrentHashMap internals (Java 8+)
- Pre-Java 8: divided into "segments" each with its own lock
- Java 8+: **lock-free reads** (volatile reads), **per-bucket CAS** for writes, only locks the bucket on collision
- `compute()`, `computeIfAbsent()`, `merge()` are atomic — preferred over get+check+put
- `size()` is approximate during concurrent updates (uses LongAdder-style counter)

```java
// BAD — race condition
if (!map.containsKey(k)) map.put(k, v);

// GOOD — atomic
map.putIfAbsent(k, v);

// BEST for compute-once-on-miss
map.computeIfAbsent(k, key -> expensiveCompute(key));
```

###  LinkedHashMap as an LRU cache
```java
Map<K, V> lru = new LinkedHashMap<>(16, 0.75f, true) {  // accessOrder = true
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > 1000;
    }
};
```

###  TreeMap — when to use
- Sorted iteration
- Range queries: `headMap`, `tailMap`, `subMap`, `floorKey`, `ceilingKey`
- O(log n) operations, but typically much slower than HashMap due to tree overhead

---

## Q4. ArrayList vs LinkedList — when to use which?

| Operation | ArrayList | LinkedList |
|---|---|---|
| `get(index)` | **O(1)** | O(n) |
| `add(e)` at end | O(1) amortized | O(1) |
| `add(0, e)` at start | O(n) (shift everything) | O(1) |
| `remove(index)` | O(n) | O(n) (find) + O(1) (unlink) |
| `iterator.remove()` | O(n) (shift) | **O(1)** |
| Memory | Array of refs | 2x refs per node + Node object |
| Cache locality | **Excellent** | Terrible |

### Reality check

**99% of the time, use ArrayList.** Modern CPUs are so much faster at sequential memory access than pointer chasing that even removing from the middle of an ArrayList often beats LinkedList in real benchmarks. LinkedList is rarely the right answer.

If you need fast queue/deque operations: use **`ArrayDeque`** (faster than LinkedList for that too).

###  ArrayList growth
```java
int oldCapacity = elementData.length;
int newCapacity = oldCapacity + (oldCapacity >> 1);  // 1.5x growth
```

If you know the final size, pre-size it: `new ArrayList<>(expectedSize)` — avoids multiple grow+copy.

---

## Q5. HashSet vs LinkedHashSet vs TreeSet.

| | HashSet | LinkedHashSet | TreeSet |
|---|---|---|---|
| Backed by | HashMap | LinkedHashMap | TreeMap |
| Order | None | Insertion | Sorted |
| add/contains | O(1) | O(1) | O(log n) |
| Null elements | 1 allowed | 1 allowed | No |

```java
// HashSet is literally HashMap with a sentinel value
public HashSet() {
    map = new HashMap<>();
}
public boolean add(E e) {
    return map.put(e, PRESENT) == null;
}
private static final Object PRESENT = new Object();
```

So everything you know about HashMap (load factor, treeify-at-8, immutability of elements) applies to HashSet.

---

## Q6. What is `fail-fast` vs `fail-safe` iteration?

**Fail-fast** (ArrayList, HashMap, HashSet): throws `ConcurrentModificationException` if the collection is structurally modified during iteration (other than via the iterator itself).

```java
List<String> list = new ArrayList<>(List.of("a", "b", "c"));
for (String s : list) {
    if (s.equals("b")) list.remove(s);  // CME thrown on next iteration
}

// Correct way — use the iterator's remove
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (it.next().equals("b")) it.remove();  // OK
}

// Modern way — removeIf (Java 8+)
list.removeIf(s -> s.equals("b"));
```

**Fail-safe** (ConcurrentHashMap, CopyOnWriteArrayList): iterator operates on a snapshot or weakly consistent view. Never throws CME but may not see concurrent modifications.

Mechanism: fail-fast uses a `modCount` field. Iterator captures it on creation, checks before each `next()`. Mismatch → CME.

###  Real-world: how to safely iterate-and-modify
1. `iterator.remove()` — works for ArrayList/HashMap
2. `removeIf(Predicate)` — Java 8+, cleanest
3. Collect-then-remove — build a list of items to remove, then `removeAll`
4. Use a concurrent collection if multi-threaded

---

## Q7. equals() / hashCode() — why they matter for Collections.

**Without consistent equals/hashCode, hash-based collections silently break.**

```java
class Bad {
    int id;
    Bad(int id) { this.id = id; }
    @Override public boolean equals(Object o) { return o instanceof Bad b && b.id == id; }
    // no hashCode override — uses Object's identity-based hash
}

Set<Bad> set = new HashSet<>();
set.add(new Bad(1));
System.out.println(set.contains(new Bad(1)));  // false!
```

Because the two `Bad` instances have different identity hashes, they end up in different buckets. `contains` only checks the bucket determined by `hashCode`. Equality is never even tested.

### The four rules (recap from core-java)
1. `a.equals(b) ⇒ a.hashCode() == b.hashCode()` (the killer rule)
2. Reflexive, symmetric, transitive, consistent
3. Mutating fields used in equals/hashCode after putting into a set = silent corruption
4. Use `Objects.hash(...)` or `record` to never get this wrong

---

## Q8. Comparable vs Comparator (Collections context).

`TreeMap` / `TreeSet` use `compareTo` (or a provided Comparator), **NOT equals/hashCode**.

```java
TreeSet<BigDecimal> set = new TreeSet<>();
set.add(new BigDecimal("1.0"));
System.out.println(set.contains(new BigDecimal("1.00")));  // TRUE!
// because 1.0.compareTo(1.00) == 0 even though !equals()
```

This is a famous gotcha — `TreeSet` and `HashSet` can give different answers for `contains()`.

###  Modern Comparator API
```java
people.sort(Comparator.comparing(Person::lastName)
                       .thenComparing(Person::firstName)
                       .thenComparingInt(Person::age)
                       .reversed());

// Null-safe
Comparator<Person> safe = Comparator.comparing(
    Person::middleName,
    Comparator.nullsFirst(Comparator.naturalOrder())
);
```

---

## Q9. Map iteration — entrySet vs keySet performance.

```java
// SLOW — does a get() for each key
for (K key : map.keySet()) {
    V value = map.get(key);
    ...
}

// FAST — single traversal, no lookup
for (Map.Entry<K, V> e : map.entrySet()) {
    K key = e.getKey();
    V value = e.getValue();
    ...
}

// CLEANEST (Java 8+)
map.forEach((key, value) -> { ... });
```

`entrySet()` is 2x+ faster on large maps. `forEach` is sometimes even faster (no Entry object allocation per iteration in some impls).

---

## Q10. Concurrent collections — when to use which.

| Need | Use |
|---|---|
| Thread-safe map (high read concurrency) | `ConcurrentHashMap` |
| Thread-safe sorted map | `ConcurrentSkipListMap` |
| Thread-safe list (read-heavy, write-rare) | `CopyOnWriteArrayList` |
| Thread-safe set (read-heavy, write-rare) | `CopyOnWriteArraySet` |
| Producer-consumer queue (bounded) | `ArrayBlockingQueue` or `LinkedBlockingQueue` |
| Producer-consumer queue (unbounded, fast) | `ConcurrentLinkedQueue` (non-blocking, CAS-based) |
| Priority queue (thread-safe) | `PriorityBlockingQueue` |
| Hand-off between threads (no buffer) | `SynchronousQueue` |
| Delayed execution queue | `DelayQueue` |
| Single-shot transfer | `Exchanger` |

###  CopyOnWriteArrayList — when NOT to use
Every `add`/`set`/`remove` copies the entire backing array. O(n) writes, but O(1) wait-free reads. Only good when you have very few writes (e.g., a listener registry that's modified at startup).

###  Collections.synchronizedX vs concurrent
```java
Map<K, V> m = Collections.synchronizedMap(new HashMap<>());
// every method is synchronized on the map — serializes all access
// iteration requires manual external sync:
synchronized (m) {
    for (Map.Entry<K, V> e : m.entrySet()) { ... }
}
```
**Almost always use `ConcurrentHashMap` instead.** Better throughput, atomic compound operations.

---

## Q11. Queue, Deque, BlockingQueue families.

```
                  Queue<E>
                  /      \
          BlockingQueue   Deque<E>      (double-ended queue)
              |               |
   +----------+----+    +-----+------+
   |          |    |    |            |
ArrayBlocking |   ...  ArrayDeque  LinkedList
LinkedBlocking         (extends Deque,
PriorityBlocking        ALSO implements Queue)
SynchronousQueue
```

### Queue contract — four method pairs

| | Throws exception | Returns special value |
|---|---|---|
| Insert | `add(e)` | `offer(e)` |
| Remove | `remove()` | `poll()` |
| Examine | `element()` | `peek()` |

### BlockingQueue adds blocking variants

| | Blocks indefinitely | Blocks with timeout |
|---|---|---|
| Insert | `put(e)` | `offer(e, time, unit)` |
| Remove | `take()` | `poll(time, unit)` |

### Producer-consumer pattern
```java
BlockingQueue<Task> queue = new LinkedBlockingQueue<>(1000);

// Producer
new Thread(() -> {
    while (running) queue.put(produceTask());  // blocks if full
}).start();

// Consumer
new Thread(() -> {
    while (running) {
        Task t = queue.take();                  // blocks if empty
        process(t);
    }
}).start();
```

###  ArrayDeque is better than Stack and LinkedList
- Faster than `LinkedList` for queue/stack operations (cache locality)
- Better than `Stack` (which is synchronized and extends Vector — legacy)
- Use `addFirst`/`removeFirst` for stack semantics, `addLast`/`removeFirst` for queue semantics

---

## Q12. PriorityQueue — what and how?

A min-heap implementation. By default orders by `Comparable.compareTo`; or pass a Comparator.

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.addAll(List.of(5, 1, 3, 2, 4));
while (!minHeap.isEmpty()) System.out.print(minHeap.poll() + " ");
// prints: 1 2 3 4 5

// Max-heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
```

###  Caveats
- **NOT sorted on iteration** — only `poll()` returns in order. `iterator()` returns arbitrary order.
- O(log n) `add`/`poll`, O(n) `contains`/`remove(Object)`.
- NOT thread-safe — use `PriorityBlockingQueue`.

---

## Q13. NavigableMap / SortedMap — TreeMap operations.

`TreeMap` implements `NavigableMap` which adds powerful range/proximity operations:

```java
NavigableMap<Integer, String> m = new TreeMap<>();
m.put(10, "a"); m.put(20, "b"); m.put(30, "c");

m.firstKey();              // 10
m.lastKey();               // 30
m.ceilingKey(15);          // 20  (smallest >= 15)
m.floorKey(25);            // 20  (largest <= 25)
m.higherKey(20);           // 30  (strictly greater)
m.lowerKey(20);            // 10  (strictly less)
m.subMap(15, 25);          // {20=b}
m.headMap(20);             // {10=a}        — strictly less
m.headMap(20, true);       // {10=a, 20=b}  — inclusive
m.tailMap(20);             // {20=b, 30=c}
m.descendingMap();         // reverse view
```

Use TreeMap when you need: range queries, nearest-neighbor lookups, or sorted iteration.

---

## Q14. Immutable collections — Java 9+ factory methods.

```java
List<String> l = List.of("a", "b", "c");
Set<Integer> s = Set.of(1, 2, 3);
Map<String, Integer> m = Map.of("a", 1, "b", 2);
Map<String, Integer> big = Map.ofEntries(
    Map.entry("a", 1),
    Map.entry("b", 2)
);

// From an existing collection
List<String> copy = List.copyOf(existingList);    // safe even if you don't trust the source
```

### Properties
- Truly immutable — `add`/`remove` throws `UnsupportedOperationException`
- Reject `null` elements — throws NPE on construction
- More memory-efficient than `Collections.unmodifiableList(new ArrayList<>())`
- Hashcode is computed once and cached

###  `Collections.unmodifiableX` is a VIEW, not a copy
```java
List<String> mutable = new ArrayList<>(List.of("a"));
List<String> view = Collections.unmodifiableList(mutable);
mutable.add("b");
view.size();  // 2 — view reflects the underlying list!
```
Use `List.copyOf(mutable)` if you want a true immutable snapshot.

---

## Q15. Collectors — common patterns for terminal stream ops.

```java
// To collection
.collect(Collectors.toList())            // -> ArrayList, modifiable
.collect(Collectors.toUnmodifiableList()) // -> truly immutable (Java 10+)
.toList()                                 // Java 16+, immutable, cleanest

// To map
.collect(Collectors.toMap(User::id, Function.identity()))
.collect(Collectors.toMap(User::id, Function.identity(),
                          (existing, duplicate) -> existing))  // merge function for duplicates
.collect(Collectors.toMap(User::id, Function.identity(),
                          (a, b) -> a, LinkedHashMap::new))    // preserve order

// Grouping
Map<Department, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::department));

Map<Department, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::department, Collectors.counting()));

Map<Department, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::department,
             Collectors.averagingDouble(Employee::salary)));

// Partitioning (binary grouping by predicate)
Map<Boolean, List<User>> adults = users.stream()
    .collect(Collectors.partitioningBy(u -> u.age() >= 18));

// Joining
String csv = list.stream().collect(Collectors.joining(",", "[", "]"));
```

(Streams covered in depth in `05-streams-functional.md`.)

---

## Q16. Why are array generics restricted? (`new T[10]` doesn't compile)

Because arrays are **covariant** but generics are **invariant** — combining them would break type safety.

```java
// Arrays are covariant
Object[] objs = new String[10];
objs[0] = 42;        // ArrayStoreException at RUNTIME — not caught at compile time

// Generics are invariant
List<Object> objList = new ArrayList<String>();   // compile error
```

If `new T[10]` were allowed, type erasure would create `Object[10]`. Putting non-T into it via an Object reference would not be caught.

###  Workaround if you must
```java
@SuppressWarnings("unchecked")
T[] arr = (T[]) new Object[10];                              // works but unsafe
T[] arr = (T[]) Array.newInstance(componentType, 10);         // safe — needs Class<T>
```

Prefer `List<T>` over `T[]` in generic code.

---

## Q17. Spliterator and parallel iteration.

A `Spliterator` is like an `Iterator` but can split itself for parallel iteration. Returned by `Collection.spliterator()` and used by `Stream` / `parallelStream`.

```java
Spliterator<Integer> s = List.of(1,2,3,4,5,6,7,8).spliterator();
Spliterator<Integer> s2 = s.trySplit();   // splits roughly in half
s.forEachRemaining(System.out::println);  // second half
s2.forEachRemaining(System.out::println); // first half
```

You normally never write Spliterators yourself — use `Stream` or `parallelStream()`. But knowing they exist explains why parallel streams work efficiently for ArrayList (good split) but poorly for LinkedList (bad split — has to walk the chain).

---

## Q18. Stream.toList() vs Collectors.toList()

Java 16+ added `Stream.toList()` as the cleanest terminal op:

```java
// Old way
List<String> a = stream.collect(Collectors.toList());
// returns an ArrayList — modifiable, but you shouldn't mutate it

// Java 16+
List<String> b = stream.toList();
// returns an immutable List, allows nulls (unlike List.of), more efficient
```

Use `stream.toList()` by default; only use `Collectors.toList()` when you specifically need a mutable list.

---

## Q19. What's the time complexity cheat sheet?

| Operation | ArrayList | LinkedList | HashMap | TreeMap | HashSet | TreeSet | ArrayDeque |
|---|---|---|---|---|---|---|---|
| add (end) | O(1)* | O(1) | O(1)* | O(log n) | O(1)* | O(log n) | O(1)* |
| add (begin) | O(n) | O(1) | - | - | - | - | O(1) |
| add (middle) | O(n) | O(n) | - | - | - | - | - |
| get (index) | O(1) | O(n) | - | - | - | - | - |
| get (key) | - | - | O(1)* | O(log n) | - | - | - |
| contains | O(n) | O(n) | O(1)* | O(log n) | O(1)* | O(log n) | O(n) |
| remove (index) | O(n) | O(n) | - | - | - | - | - |
| remove (object) | O(n) | O(n) | O(1)* | O(log n) | O(1)* | O(log n) | O(n) |
| iterator.remove | O(n) | O(1) | O(1) | O(log n) | O(1) | O(log n) | - |

\* amortized; worst case may differ (e.g., HashMap collision chain or resize)

---

## Q20. What problems does the legacy `Vector` / `Hashtable` have?

Both date from Java 1.0 and have:
- Every method synchronized → terrible concurrent throughput (locks the whole structure)
- Cannot be made non-synchronized for single-thread use
- Compound operations still race-prone:
  ```java
  if (!vector.contains(x)) vector.add(x);  // still TOCTOU race!
  ```
- `Hashtable` rejects null keys and values
- `Stack extends Vector` — drags all the legacy baggage

**Modern replacements**:
- `Vector` → `ArrayList` (or `Collections.synchronizedList(new ArrayList<>())`)
- `Hashtable` → `ConcurrentHashMap`
- `Stack` → `ArrayDeque`
- `Enumeration` → `Iterator`
- `Properties` → still used (config files) — but inherits from `Hashtable`, careful
