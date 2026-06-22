# 07 - Comparator Cheat Sheet

The one note that ends all Comparator confusion. PriorityQueue, Collections.sort, `int[]`, `int[][]`, custom multi-criteria - all in one place.

---

## The 3-line contract you MUST memorize

A `Comparator<T>` returns:

```
compare(a, b) < 0   ->  a comes BEFORE b   (a is "smaller")
compare(a, b) == 0  ->  a and b are equal
compare(a, b) > 0   ->  a comes AFTER b    (a is "bigger")
```

That's it. Negative = first, positive = last, zero = tie.

### Memory hook: "ascending = a-b"
```java
(a, b) -> a - b      // ascending  (small first)
(a, b) -> b - a      // descending (big first)
```
Pretend you're answering "where should `a` go relative to `b`?". If `a < b`, you want a negative result -> `a - b` is negative. Done.

---

## The CRITICAL overflow gotcha (`a - b` trap)

```java
int a = Integer.MAX_VALUE;
int b = -1;
int result = a - b;     // OVERFLOWS to Integer.MIN_VALUE !
```

Sorting an `int[]` with `(a, b) -> a - b` can give wrong order if you have huge positive AND huge negative numbers.

### ALWAYS use `Integer.compare` instead:

```java
//  WRONG (overflow-prone)
Arrays.sort(boxed, (a, b) -> a - b);

//  RIGHT (overflow-safe)
Arrays.sort(boxed, (a, b) -> Integer.compare(a, b));
Arrays.sort(boxed, Integer::compare);                  // method ref - cleaner
Arrays.sort(boxed, Comparator.naturalOrder());          // even cleaner
```

Same applies to `Long.compare`, `Double.compare`, etc.

---

## Sorting 1D arrays

### Primitive `int[]` - NO Comparator allowed
```java
int[] arr = {3, 1, 4, 1, 5, 9, 2, 6};

Arrays.sort(arr);                              // ascending (only option for primitives)

// DESCENDING for primitive int[] - no direct way! Three workarounds:

// Workaround 1: box, sort with reverseOrder, copy back (verbose)
Integer[] boxed = Arrays.stream(arr).boxed().toArray(Integer[]::new);
Arrays.sort(boxed, Comparator.reverseOrder());
for (int i = 0; i < arr.length; i++) arr[i] = boxed[i];

// Workaround 2: sort ascending, then reverse manually
Arrays.sort(arr);
for (int i = 0, j = arr.length - 1; i < j; i++, j--) {
    int t = arr[i]; arr[i] = arr[j]; arr[j] = t;
}

// Workaround 3: use Integer[] from the start (most code-friendly)
Integer[] arr2 = {3, 1, 4, 1, 5, 9};
Arrays.sort(arr2, Comparator.reverseOrder());
```

### Why no Comparator for `int[]`?
`Arrays.sort(int[])` uses Dual-Pivot Quicksort tuned for primitives - no callback overhead. The trade-off: you can't customize order.

### Object array `Integer[]`, `String[]`, etc.
```java
Integer[] arr = {3, 1, 4, 1, 5};

Arrays.sort(arr);                              // natural (ascending)
Arrays.sort(arr, Comparator.naturalOrder());   // same
Arrays.sort(arr, Comparator.reverseOrder());   // descending
Arrays.sort(arr, (a, b) -> b - a);             // descending (overflow risk!)
Arrays.sort(arr, (a, b) -> Integer.compare(b, a));  // descending (safe)

// Partial sort
Arrays.sort(arr, 2, 5);                        // sort indices [2..5)
```

### `String[]`
```java
String[] words = {"banana", "Apple", "cherry"};

Arrays.sort(words);                                  // natural - lexicographic, case-sensitive (uppercase < lowercase)
Arrays.sort(words, String.CASE_INSENSITIVE_ORDER);   // case-insensitive
Arrays.sort(words, Comparator.comparingInt(String::length));    // by length
Arrays.sort(words, (a, b) -> a.length() - b.length());          // by length (safe here - lengths >= 0)
```

---

## Sorting 2D arrays `int[][]` - the interview classic

`int[][]` is an array of `int[]` references. So you CAN use a Comparator (because rows are objects).

```java
int[][] intervals = {{1, 3}, {2, 6}, {8, 10}, {15, 18}};

// Sort by FIRST column (most common - meeting rooms, merge intervals)
Arrays.sort(intervals, (a, b) -> a[0] - b[0]);              // common but overflow-risky
Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));  // safe
Arrays.sort(intervals, Comparator.comparingInt(a -> a[0])); // cleanest

// Sort by SECOND column
Arrays.sort(intervals, Comparator.comparingInt(a -> a[1]));

// Sort by first column DESCENDING
Arrays.sort(intervals, (a, b) -> Integer.compare(b[0], a[0]));
Arrays.sort(intervals, Comparator.<int[]>comparingInt(a -> a[0]).reversed());

// MULTI-CRITERIA: first column asc, ties broken by second column desc
Arrays.sort(intervals,
    Comparator.<int[]>comparingInt(a -> a[0])
              .thenComparing((a, b) -> Integer.compare(b[1], a[1]))
);
```

### Common 2D patterns (interview gold)

```java
// Meeting Rooms II - sort by start time
Arrays.sort(intervals, (a, b) -> a[0] - b[0]);

// Merge Intervals - sort by start
Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]));

// K Closest Points to Origin - sort by distance squared
int[][] points = {{1,3}, {-2,2}, {5,8}};
Arrays.sort(points, (a, b) -> (a[0]*a[0] + a[1]*a[1]) - (b[0]*b[0] + b[1]*b[1]));

// Reduce Array Size to The Half - sort by frequency desc
// (assumes int[][] where each row is [value, frequency])
Arrays.sort(freqs, (a, b) -> b[1] - a[1]);
```

### Why `Comparator.<int[]>comparingInt(...)` with explicit type?
Without the type witness `<int[]>`, the compiler infers `Comparator<Object>` and `.reversed()` won't chain on an `int[]`. Once you call `.thenComparing(...)` or `.reversed()`, you need the type hint.

```java
// FAILS to compile
Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]).reversed());

// COMPILES
Arrays.sort(intervals, Comparator.<int[]>comparingInt(a -> a[0]).reversed());

// OR avoid the witness by extracting:
Comparator<int[]> byStart = Comparator.comparingInt(a -> a[0]);
Arrays.sort(intervals, byStart.reversed());
```

---

## Sorting Collections (List)

### Old way (works, but verbose)
```java
List<Integer> list = new ArrayList<>(List.of(3, 1, 4, 1, 5));
Collections.sort(list);                              // natural ascending
Collections.sort(list, Comparator.reverseOrder());   // descending
```

### Modern way (Java 8+, preferred)
```java
list.sort(null);                                  // natural - null means natural
list.sort(Comparator.naturalOrder());
list.sort(Comparator.reverseOrder());
list.sort((a, b) -> a - b);                       // overflow risk
list.sort(Integer::compare);                       // safe
```

### List of objects
```java
List<Employee> employees = ...;

employees.sort(Comparator.comparing(Employee::getName));
employees.sort(Comparator.comparing(Employee::getName).reversed());
employees.sort(Comparator.comparingInt(Employee::getAge));    // primitive int - avoids boxing

// Multi-criteria: department asc, then salary desc, then name asc
employees.sort(
    Comparator.comparing(Employee::getDepartment)
              .thenComparing(Employee::getSalary, Comparator.reverseOrder())
              .thenComparing(Employee::getName)
);

// Null-safe
employees.sort(Comparator.comparing(
    Employee::getMiddleName,
    Comparator.nullsFirst(Comparator.naturalOrder())   // nulls go first
));
```

### List.of(...) is IMMUTABLE - can't sort
```java
List<Integer> immutable = List.of(3, 1, 2);
immutable.sort(null);   // UnsupportedOperationException!

// Fix: copy to ArrayList first
List<Integer> mutable = new ArrayList<>(List.of(3, 1, 2));
mutable.sort(null);
```

---

## PriorityQueue - the heap, NOT a sorted list

**Critical mental model:** PriorityQueue is a **binary heap**, not a sorted collection. Only the *top* (smallest by default) is accessible in O(1). Iteration order is NOT sorted.

### Min-heap (default - smallest first)
```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.add(5); minHeap.add(1); minHeap.add(3);

minHeap.peek();    // 1
minHeap.poll();    // 1 (removes)
minHeap.peek();    // 3
```

### Max-heap (biggest first)
```java
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.add(5); maxHeap.add(1); maxHeap.add(3);

maxHeap.poll();    // 5
```

### Custom Comparator
```java
// Min-heap of int[] by first element (e.g., for Dijkstra's [node, dist])
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);
// or safer:
PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[1]));

// Max-heap of strings by length, ties broken alphabetically
PriorityQueue<String> pq2 = new PriorityQueue<>(
    Comparator.comparingInt(String::length).reversed()
              .thenComparing(Comparator.naturalOrder())
);
```

###  GOTCHA: iteration is NOT sorted
```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.addAll(List.of(5, 1, 3, 2, 4));

for (int x : pq) System.out.print(x + " ");   // could print 1 2 3 5 4 - heap order, NOT sorted!

// To get sorted output, poll until empty:
while (!pq.isEmpty()) System.out.print(pq.poll() + " ");   // 1 2 3 4 5
```

###  GOTCHA: `contains()` and `remove(Object)` are O(n)
PriorityQueue is optimized for add/peek/poll. Don't use it as a general collection.

###  GOTCHA: changing element priority after insertion
If you mutate an element in a way that changes its comparator output AFTER inserting, the heap is silently corrupted.
```java
// Wrong - heap order broken
pq.add(item);
item.priority = 999;   // pq doesn't know; heap invariant broken

// Right - remove, mutate, re-add
pq.remove(item);
item.priority = 999;
pq.add(item);
```

### Common PriorityQueue interview patterns

```java
// Kth Largest Element - min-heap of size K
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
for (int n : nums) {
    minHeap.offer(n);
    if (minHeap.size() > k) minHeap.poll();
}
return minHeap.peek();   // Kth largest

// Top K Frequent - max-heap by frequency
Map<Integer, Integer> freq = new HashMap<>();
for (int n : nums) freq.merge(n, 1, Integer::sum);

PriorityQueue<Map.Entry<Integer, Integer>> maxHeap =
    new PriorityQueue<>((a, b) -> b.getValue() - a.getValue());
maxHeap.addAll(freq.entrySet());

int[] result = new int[k];
for (int i = 0; i < k; i++) result[i] = maxHeap.poll().getKey();

// Merge K Sorted Lists - min-heap of list heads
PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
for (ListNode head : lists) if (head != null) pq.add(head);
// then poll, advance, re-add ...

// Dijkstra's - min-heap of [distance, node]
PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
```

---

## TreeMap / TreeSet - sorted by Comparator

`TreeMap` and `TreeSet` use a `Comparator` (or natural ordering) for **ordering AND equality**. Unlike PriorityQueue, iteration IS sorted.

```java
TreeMap<String, Integer> tm = new TreeMap<>();           // natural - keys sorted alphabetically
TreeMap<String, Integer> rev = new TreeMap<>(Comparator.reverseOrder());

TreeSet<int[]> ts = new TreeSet<>((a, b) -> a[0] - b[0]);   // sorted by first element
```

###  CRITICAL: in a TreeSet, `equals` is determined by `compareTo() == 0`, NOT by `.equals()`
```java
TreeSet<int[]> ts = new TreeSet<>((a, b) -> a[0] - b[0]);
ts.add(new int[]{1, 10});
ts.add(new int[]{1, 20});      // NOT added! comparator says they're equal (both have first=1)
ts.size();                     // 1
```

This burns people regularly. Make sure your comparator differentiates everything that should be unique:
```java
// Better - tie-break on second element
TreeSet<int[]> ts = new TreeSet<>(
    Comparator.<int[]>comparingInt(a -> a[0]).thenComparingInt(a -> a[1])
);
```

---

## The Comparator factory methods - your best friends

These eliminate 90% of comparator bugs. **Use them by default.**

```java
Comparator.naturalOrder()                    // T must be Comparable
Comparator.reverseOrder()                    // reverse natural
Comparator.comparing(Function)               // by extracted Comparable key
Comparator.comparingInt(ToIntFunction)       // by extracted int (no boxing)
Comparator.comparingLong(ToLongFunction)
Comparator.comparingDouble(ToDoubleFunction)
Comparator.comparing(Function, Comparator)   // by extracted key with custom comparator

// Chaining
.thenComparing(Comparator)
.thenComparing(Function)
.thenComparingInt(ToIntFunction)
.reversed()

// Null handling
Comparator.nullsFirst(Comparator)
Comparator.nullsLast(Comparator)
```

### Side-by-side cheat sheet

| Goal | Lambda | Factory method |
|---|---|---|
| Sort `Integer[]` ascending | `(a, b) -> a - b` | `naturalOrder()` |
| Sort `Integer[]` descending | `(a, b) -> b - a` | `reverseOrder()` |
| `int[][]` by col 0 asc | `(a, b) -> a[0] - b[0]` | `comparingInt(a -> a[0])` |
| `int[][]` by col 0 desc | `(a, b) -> b[0] - a[0]` | `Comparator.<int[]>comparingInt(a -> a[0]).reversed()` |
| Employee by name | `(a, b) -> a.name.compareTo(b.name)` | `comparing(Employee::getName)` |
| Employee by salary desc | `(a, b) -> Double.compare(b.sal, a.sal)` | `comparingDouble(Employee::getSalary).reversed()` |
| Multi-criteria | hand-rolled if/else mess | `comparing(...).thenComparing(...)` |

---

## Comparable vs Comparator - one-line distinction

- **`Comparable<T>`** lives **inside** the class. Defines THE one natural ordering. `compareTo(T other)`.
- **`Comparator<T>`** lives **outside** as a separate object. Defines ONE OF MANY orderings. `compare(T a, T b)`.

```java
public class User implements Comparable<User> {        // natural ordering = by age
    int age; String name;
    @Override
    public int compareTo(User other) {
        return Integer.compare(this.age, other.age);   // safe; no overflow
    }
}

// Now you can sort without specifying anything
List<User> users = ...;
Collections.sort(users);                          // uses Comparable
PriorityQueue<User> pq = new PriorityQueue<>();   // uses Comparable

// But also use any Comparator on the fly
users.sort(Comparator.comparing(User::getName));
PriorityQueue<User> pq2 = new PriorityQueue<>(Comparator.comparing(User::getName));
```

### Subtle gotcha - Comparable contract
`compareTo` should be **consistent with equals**: `a.compareTo(b) == 0` should imply `a.equals(b)`.

Classic violation: `BigDecimal("1.0").compareTo(BigDecimal("1.00")) == 0` but they're not `.equals()` (different scale). Don't put `BigDecimal` in a `TreeSet` if scale matters.

---

## The "decisions" table - what comparator/sort to reach for

| Situation | Reach for |
|---|---|
| Sort `int[]` ascending | `Arrays.sort(arr)` |
| Sort `int[]` descending | Box to `Integer[]`, then `Comparator.reverseOrder()` |
| Sort `Integer[]` / `String[]` ascending | `Arrays.sort(arr)` |
| Sort `Integer[]` / `String[]` descending | `Arrays.sort(arr, Comparator.reverseOrder())` |
| Sort `int[][]` by column | `Arrays.sort(arr, Comparator.comparingInt(a -> a[col]))` |
| Sort `List<T>` | `list.sort(comparator)` |
| Min-heap | `new PriorityQueue<>()` |
| Max-heap | `new PriorityQueue<>(Comparator.reverseOrder())` |
| Min-heap of objects by field | `new PriorityQueue<>(Comparator.comparingInt(T::getKey))` |
| Sorted unique keys | `TreeSet<>(comparator)` - but tie-break carefully |
| Sorted map | `TreeMap<>(comparator)` |
| Multi-criteria sort | `comparing(...).thenComparing(...).thenComparing(...)` |

---

## Top 10 mistakes (and how to fix them)

### 1. `a - b` overflow
```java
// BAD - can overflow
pq = new PriorityQueue<>((a, b) -> a - b);

// GOOD
pq = new PriorityQueue<>(Integer::compare);
pq = new PriorityQueue<>(Comparator.naturalOrder());
```

### 2. Trying to sort `int[]` descending with a Comparator
```java
// COMPILES but uses natural - ignores your wish
int[] arr = {3, 1, 4};
Arrays.sort(arr);            // ascending only, no Comparator overload for int[]

// To go descending: box first
Integer[] boxed = Arrays.stream(arr).boxed().toArray(Integer[]::new);
Arrays.sort(boxed, Comparator.reverseOrder());
```

### 3. Forgetting `Comparator.<int[]>` type witness on chaining
```java
// FAILS
Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]).reversed());

// WORKS
Arrays.sort(intervals, Comparator.<int[]>comparingInt(a -> a[0]).reversed());
```

### 4. Expecting PriorityQueue iteration to be sorted
```java
for (T t : pq) ...;       // HEAP order, not sorted!

// To iterate in priority order:
while (!pq.isEmpty()) process(pq.poll());
```

### 5. Mutating an element after putting it in a PriorityQueue or TreeSet
```java
pq.add(task);
task.priority = 999;     // heap silently corrupted

// Right way
pq.remove(task);
task.priority = 999;
pq.add(task);
```

### 6. Treating TreeSet equality as Object.equals
```java
TreeSet<int[]> ts = new TreeSet<>((a, b) -> a[0] - b[0]);
ts.add(new int[]{1, 5});
ts.add(new int[]{1, 10});   // dropped! comparator says equal
```

### 7. Sorting an immutable list
```java
List.of(3, 1, 2).sort(null);    // UnsupportedOperationException

// Wrap first
new ArrayList<>(List.of(3, 1, 2)).sort(null);
```

### 8. NullPointerException in `comparing()` when extracted key is null
```java
list.sort(Comparator.comparing(User::getMiddleName));    // NPE if middleName is null

// Null-safe
list.sort(Comparator.comparing(
    User::getMiddleName,
    Comparator.nullsLast(Comparator.naturalOrder())
));
```

### 9. Forgetting `Comparable` is needed for `naturalOrder()` / no-arg `PriorityQueue`
```java
class Task { int priority; }     // does NOT implement Comparable

PriorityQueue<Task> pq = new PriorityQueue<>();
pq.add(new Task());
pq.add(new Task());              // ClassCastException at insertion time!

// Fix: provide Comparator OR implement Comparable
PriorityQueue<Task> pq = new PriorityQueue<>(Comparator.comparingInt(t -> t.priority));
```

### 10. Returning `boolean`-ish values from compare
```java
// BAD - returns 0 or 1, treats "equal" and "before" as the same
(a, b) -> a < b ? 1 : 0;

// Always return three states; use Integer.compare
(a, b) -> Integer.compare(a, b);
```

---

## TL;DR - the 4 lines to remember

```java
// Sort int[] ascending - that's all primitives can do
Arrays.sort(intArr);

// Sort anything else: pass a comparator
Arrays.sort(objArr, Comparator.comparingInt(o -> o.key));

// Min-heap with custom key
PriorityQueue<T> pq = new PriorityQueue<>(Comparator.comparingInt(T::getKey));

// Reverse anything
.reversed()
```

If you internalize these four lines + the overflow rule (`Integer.compare` instead of `a - b`), 95% of comparator confusion goes away.
