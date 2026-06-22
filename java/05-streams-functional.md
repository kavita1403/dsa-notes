# 05 — Streams & Functional Programming

Lambdas, Stream API internals, collectors, parallel streams, Optional. Java 8 features that should be second nature by now.

---

## Q1. What is a lambda expression and how does it compare to anonymous classes?

A lambda is a concise way to implement a **functional interface** (interface with one abstract method).

```java
// Anonymous class (pre-Java 8)
Runnable r1 = new Runnable() {
    @Override
    public void run() { System.out.println("hi"); }
};

// Lambda (Java 8+)
Runnable r2 = () -> System.out.println("hi");
```

### Under the hood
- Anonymous class: generates a new `.class` file at compile time, instantiates per usage
- Lambda: uses `invokedynamic` bytecode, JVM creates the implementation lazily, can be reused, no extra class file

```bash
javap -p MyClass    # see the generated lambda$0 synthetic methods
```

### Differences
| | Anonymous class | Lambda |
|---|---|---|
| `this` | refers to the anonymous instance | refers to the **enclosing** instance |
| Can capture mutable local? | No (must be final/effectively final) | No (same) |
| Can have multiple methods? | Yes | No (single abstract method only) |
| Has fields? | Yes | No |
| `new` per instance? | Yes (always) | JIT may cache; not guaranteed identity |
| Suited for | Stateful, complex | One-line behavior |

###  `this` trap
```java
class Outer {
    Runnable a = new Runnable() {
        @Override public void run() { System.out.println(this); }  // anonymous instance
    };
    Runnable b = () -> System.out.println(this);                   // Outer instance
}
```

---

## Q2. What is a functional interface? List the key ones.

A functional interface has **exactly one abstract method** (SAM = Single Abstract Method). May have default/static methods. Annotate with `@FunctionalInterface` for compiler enforcement.

```java
@FunctionalInterface
interface Validator<T> {
    boolean isValid(T input);              // the single abstract method
    default boolean isInvalid(T x) { return !isValid(x); }  // OK, has body
}
```

### Standard functional interfaces (`java.util.function`)

| Interface | Method | Use case |
|---|---|---|
| `Function<T, R>` | `R apply(T)` | Transform |
| `BiFunction<T, U, R>` | `R apply(T, U)` | Two-arg transform |
| `Predicate<T>` | `boolean test(T)` | Filter |
| `Consumer<T>` | `void accept(T)` | Side effect |
| `Supplier<T>` | `T get()` | Lazy value |
| `UnaryOperator<T>` | `T apply(T)` | Transform same type |
| `BinaryOperator<T>` | `T apply(T, T)` | Reduce, combine |
| `Runnable` | `void run()` | No-arg, no-return |
| `Callable<V>` | `V call()` throws | Returns value, can throw checked |

### Primitive specializations (avoid boxing!)
`IntFunction<R>`, `ToIntFunction<T>`, `IntPredicate`, `IntConsumer`, `IntSupplier`, `IntUnaryOperator`, etc. Same for `Long` and `Double`. **Use these in hot paths.**

---

## Q3. Method references — the four kinds.

```java
// 1. Static method reference
Function<String, Integer> parser = Integer::parseInt;          // Integer.parseInt(s)

// 2. Instance method reference (bound — on a specific object)
String greet = "hello";
Supplier<Integer> len = greet::length;                          // greet.length()

// 3. Instance method reference (unbound — on the parameter)
Function<String, Integer> len2 = Strih;                // s.length()

// 4. Constructor reference
Supplier<ArrayList<String>> factory = ArrayList::new;
Function<Integer, ArrayList<String>> sized = ArrayList::new;
```

###  Pro tip: prefer method refs over lambdas when possible — more readable, sometimes faster (no synthetic method).
```java
list.forEach(System.out::println);          // BETTER than list.forEach(x -> System.out.println(x));
```

---

## Q4. Stream API — what is a Stream and how does it work?

A `Stream` is a **lazy, functional, single-pass** sequence of elements supporting pipeline operations.

```java
list.stream()                  // source
    .filter(x -> x > 10)       // intermediate (lazy)
    .map(x -> x * 2)           // intermediate (lazy)
    .sorted()                  // intermediatestateful)
    .limit(5)                  // intermediate (short-circuit)
    .collect(toList());        // terminal (eager — triggers evaluation)
```

### Three kinds of operations

| Kind | When run | Returns | Examples |
|---|---|---|---|
| Source | At collection | Stream | `stream()`, `Stream.of`, `IntStream.range`, `Files.lines` |
| Intermediate | Lazy (not until terminal) | Stream | `filter`, `map`, `peek`, `sorted`, `distinct`, `limit` |
| Terminal | Triggers pipeline | Result | `collect`, `forEach`, `count`, `reduce`, `findFirst`, `toArray` |

### Stateless vs stateful intermediate ops
- **Stateless** (`map`, `filter`, `peek`): process one element at a time
- **Stateful** (`sorted`, `distinct`, `limit`, `skip`): need to buffer or know about other elements → bottleneck for parallel streams

### Short-circuiting
Some operations can stop early:
- `findFirst`, `findAny`, `anyMatch`, `allMatch`, `noneMatch`
- `limit(n)`

```java
Stream.iterate(1, i -> i + 1)        // infinite stream
      .filter(i -> i % 7 == 0)
      .findFirst()                    // stops at first match
      .get();                          // 7
```

---

## Q5. Streams are single-use — what does that mean?

```java
Stream<Integer> s = list.stream();
s.count();                          // OK
s.count();                          // IllegalStateException — stream has already been operated upon or closed
```

A Stream cannot be reused. You must re-create from the source.

```java
Supplier<Stream<Integer>> s = list::stream;
s.get().count();    // OK
s.get().count();    // OK — new stream each call
```

---

## Q6. map vs flatMap.

```java
List<List<Integer>> nested = List.of(List.of(1, 2), List.of(3, 4));

// map — wraps each result
Stream<List<Integer>> mapped = nested.stream().map(l -> l);
// type: Stream<List<Integer>>

// flatMap — unwraps one level
Stream<Integer> flat = nested.stream().flatMap(List::stream);
// type: Stream<Integer>  -> 1, 2, 3, 4
```

`flatMap` takes a function `T -> Stream<R>` and concatenates the resulting streams.

### Common usage
```java
List<Order> allOrders = users.stream()
    .flatMap(user -> user.getOrders().stream())
    .toList();

// Parse lines into words
List<String> words = lines.stream()
    .flatMap(line -> Arrays.stream(line.split("\\s+")))
    .toList();

// Optional flatMap — chains Optional<T> -> Optional<R>
Optional<String> email = user.flatMap(User::getEmail);
```

###  Optional has its own map/flatMap — same idea
```java
optional.map(String::trim).filter(s -> !s.isEmpty()).orElse("default");
```

---

## Q7. Collectors — the most useful ones.

```java
// Basic collections
.collect(Collectors.toList())            // mutable ArrayList
.collect(Collectors.toUnmodifiableList()) // immutable; Java 10+
.toList()                                 // Java 16+ — preferred, immutable

.collect(Collectors.toSet())
.collect(Collectors.toCollection(TreeSet::new))    // specific implementation

// Map
.collect(Collectors.toMap(User::getId, Function.identity()))

// With duplicate key handling
.collect(Collectors.toMap(
    User::getEmail,
    Function.identity(),
    (existing, dup) -> existing,           // keep first
    LinkedHashMap::new                      // preserve order
));

// Grouping
Map<Department, List<Employee>> byDept = emps.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// Grouping + downstream
Map<Department, Long> countByDept = emps.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment, Collectors.counting()));

Map<Department, Double> avgSalary = emps.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));

Map<Department, List<String>> namesByDept = emps.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.mapping(Employee::getName, Collectors.toList())
    ));

// Partitioning (binary group by predicate)
Map<Boolean, List<User>> adults = users.stream()
    .collect(Collectors.partitioningBy(u -> u.age() >= 18));

// Joining strings
String csv = names.stream().collect(Collectors.joining(", ", "[", "]"));

// Stats
DoubleSummaryStatistics stats = nums.stream()
    .collect(Collectors.summarizingDouble(Number::doubleValue));
// stats.getAverage(), getMin(), getMax(), getCount(), getSum()

// Reducing
Optional<Employee> highestPaid = emps.stream()
    .collect(Collectors.reducing(BinaryOperator.maxBy(comparingDouble(Employee::salary))));
```

---

## Q8. reduce — what are the three forms?

```java
// 1. Two-arg: identity + binary op
int sum = nums.stream().reduce(0, Integer::sum);

// 2. One-arg: returns Optional (empty stream -> empty)
Optional<Integer> max = nums.stream().reduce(Integer::max);

// 3. Three-arg: identity + accumulator + combiner (for parallel)
int totalLength = strings.stream()
    .reduce(0,
            (acc, s) -> acc + s.length(),      // accumulator (per element)
            Integer::sum);                      // combiner (merge partials in parallel)
```

### Why three-arg for parallel?
In parallel, the stream is split. Each split is reduced to a partial result. The combiner merges partial results. The accumulator may be of a different type than the elements.

### Reduce contract
- **Associative**: `(a op b) op c == a op (b op c)`
- **Identity**: `identity op a == a`
- **Stateless** if parallel

Violating → unpredictable results in parallel mode.

---

## Q9. Parallel streams — when to use, when to avoid.

```java
list.parallelStream().filter(...).reduce(...);
// or equivalently
list.stream().parallel().filter(...).reduce(...);
```

Uses `ForkJoinPool.commonPool()` (`numProcessors - 1` worker threads).

### Use when
- Large source (millions of elements)
- Per-element computation is expensive
- Operations are stateless, associative
- Source splits well (ArrayList, array, range)
- Not order-dependent

### Avoid when
- Small data — overhead dominates
- I/O or blocking in operations — starves the common pool
- LinkedList, `Stream.iterate`, `Files.lines` — poor splitting
- Order matters (parallel + `forEach` = arbitrary order — use `forEachOrdered`)
- Inside a server request handler — competes with other parallel work

### The commonPool danger
```java
// SCARY — blocks every other parallel op in your JVM
list.parallelStream().map(x -> httpClient.get(...)).toList();
```
For I/O, use `CompletableFuture` with your own executor.

###  Pick your own pool with submit()
```java
ForkJoinPool myPool = new ForkJoinPool(8);
List<Integer> result = myPool.submit(() ->
    list.parallelStream().filter(...).collect(toList())
).get();
```

---

## Q10. Optional — proper usage.

`Optional<T>` is a container for "value or absent". Designed to be a **return type** for methods that might not have a value.

```java
public Optional<User> findByEmail(String email) {
    User u = db.query(...);
    return Optional.ofNullable(u);
}

// Consume
findByEmail(e).ifPresent(user -> log("found " + user));

// Transform
String displayName = findByEmail(e)
    .map(User::getName)
    .map(String::toUpperCase)
    .orElse("ANONYMOUS");

// Chain
findByEmail(e)
    .flatMap(User::getOptionalSpouse)
    .map(User::getName)
    .ifPresent(System.out::println);

// Throw if missing
User u = findByEmail(e).orElseThrow(() -> new NotFoundException(e));
```

###  Anti-patterns

```java
// 1. Optional as method parameter — just use null or overload
void update(Optional<String> name) { ... }      // BAD

// 2. Optional as field — Optional isn't Serializable, adds heap pressure
class User { Optional<String> email; }           // BAD

// 3. Optional<List<T>> — return empty list instead
Optional<List<X>> getThings();                   // BAD
List<X> getThings();                              // GOOD — empty if none

// 4. .get() without check — defeats the purpose
opt.get()                                         // throws NoSuchElementException
opt.orElseThrow()                                 // better; same behavior, clearer intent

// 5. Optional.of(value) when value might be null
Optional.of(maybeNull);                           // NPE if null!
Optional.ofNullable(maybeNull);                   // safe
```

###  Use the right primitive Optional
`OptionalInt`, `OptionalLong`, `OptionalDouble` — avoid boxing in hot paths.

---

## Q11. peek() — what is it for?

`peek` is an intermediate op that consumes each element without modifying it. **Intended for debugging only.**

```java
list.stream()
    .filter(x -> x > 10)
    .peek(x -> System.out.println("after filter: " + x))   // debug
    .map(x -> x * 2)
    .peek(x -> System.out.println("after map: " + x))      // debug
    .toList();
```

###  Don't use peek for side effects
- Not guaranteed to run if downstream is short-circuited (e.g., findFirst)
- Misleading — readers expect intermediate ops to be pure
- Use `forEach` for intentional side effects (terminal op)

---

## Q12. Stream source types.

```java
// From collections
list.stream()
list.parallelStream()

// From values
Stream.of(1, 2, 3)
Stream.empty()
Stream.ofNullable(maybeNull)   // empty if null; one elem otherwise

// Infinite
Stream.generate(() -> Math.random())     // unbounded; need limit
Stream.iterate(0, i -> i + 1)             // 0, 1, 2, ...
Stream.iterate(0, i -> i < 100, i -> i+1) // Java 9+ bounded form

// From primitive ranges
IntStream.range(0, 100)         // 0..99
IntStream.rangeClosed(1, 100)   // 1..100

// From arrays
Arrays.stream(intArray)
Stream.of(strArray)

// From files (auto-closes with try-with-resources)
try (Stream<String> lines = Files.lines(path)) {
    lines.filter(...).forEach(...);
}

// From regex
Pattern.compile(",").splitAsStream("a,b,c");

// From Iterator/Iterable
StreamSupport.stream(iterable.spliterator(), false);
```

---

## Q13. Stream collectors — writing a custom one.

A `Collector<T, A, R>` has 5 parts:
- **Supplier**: create new accumulator container
- **Accumulator**: add element to container
- **Combiner**: merge two containers (for parallel)
- **Finisher**: transform container into result
- **Characteristics**: CONCURRENT, UNORDERED, IDENTITY_FINISH

```java
Collector<String, StringBuilder, String> joiner = Collector.of(
    StringBuilder::new,              // supplier
    (sb, s) -> sb.append(s),         // accumulator
    (a, b) -> a.append(b),           // combiner
    StringBuilder::toString          // finisher
);
String joined = stream.collect(joiner);
```

### Real-world example — to ImmutableList (Guava)
```java
public static <T> Collector<T, ?, ImmutableList<T>> toImmutableList() {
    return Collector.of(
        ImmutableList::<T>builder,
        ImmutableList.Builder::add,
        (a, b) -> a.addAll(b.build()),
        ImmutableList.Builder::build
    );
}
```

---

## Q14. Common stream pitfalls.

### 1. Boxing
```java
// BAD — boxes every int
int sum = list.stream().mapToInt(Integer::intValue).sum();   // wait, this is fine

// BAD — boxes
list.stream().reduce(0, Integer::sum);

// GOOD
list.stream().mapToInt(Integer::intValue).sum();
```

### 2. Side effects in lambdas
```java
List<Integer> results = new ArrayList<>();
stream.parallel().forEach(results::add);      // ConcurrentModificationException-ish
// Use collect instead
List<Integer> results = stream.parallel().toList();
```

### 3. Mutating the source during iteration
```java
list.stream().forEach(x -> list.remove(x));    // ConcurrentModificationException
```

### 4. Calling collection from inside parallel
```java
ConcurrentMap<X, Y> map = stream.parallel().collect(
    Collectors.toConcurrentMap(...));  // use concurrent collector for parallel
```

### 5. Stream.sorted on an infinite stream
```java
Stream.iterate(0, i -> i + 1).sorted().limit(5);   // hangs forever
```

### 6. Order-dependent operations on parallel streams
```java
List<Integer> result = list.parallelStream()
    .map(transform)
    .collect(toList());     // OK — collector maintains order

list.parallelStream().forEach(System.out::println);          // unordered
list.parallelStream().forEachOrdered(System.out::println);   // ordered, slow
```

### 7. `flatMap` with `parallel` ignores order in inner stream
Just one of many subtle parallel surprises.

---

## Q15. Lazy evaluation — what does it mean in practice?

Intermediate ops don't execute until a terminal op asks for data. The stream is a recipe, not work-in-progress.

```java
// Nothing runs yet
Stream<Integer> s = list.stream()
    .filter(x -> { System.out.println("filter " + x); return x > 5; })
    .map(x -> { System.out.println("map " + x); return x * 2; });

s.forEach(System.out::println);    // NOW it runs
```

Order of printed lines: `filter 6, map 6, 12, filter 7, map 7, 14, ...` — pipeline runs element-by-element, not phase-by-phase.

###  Short-circuit benefit
```java
infiniteStream.filter(predicate).findFirst();
// stops at the first match — never processes the whole infinite stream
```

---

## Q16. forEach vs forEachOrdered.

```java
List.of(1,2,3,4).parallelStream().forEach(System.out::println);
// prints in ARBITRARY order

List.of(1,2,3,4).parallelStream().forEachOrdered(System.out::println);
// prints 1 2 3 4 — encounter order maintained
```

`forEachOrdered` adds synchronization overhead in parallel mode to preserve ordering. Slower but predictable.

---

## Q17. Streams vs Collections — what's the difference conceptually?

| | Collection | Stream |
|---|---|---|
| Storage | Stores elements | Doesn't store — pipeline over a source |
| Iteration | External (you control) | Internal (stream controls) |
| Lifecycle | Mutable, multiple use | Single-use, lazy |
| Operations | Modify the collection | Pure functions, return new |
| Time of computation | Eager (operations execute immediately) | Lazy (only on terminal op) |

Stream = transient query; Collection = persistent data.

---

## Q18. Custom functional interface vs `Function`/`Predicate`/etc.

When to define your own:

```java
// Reusable, named, documented
@FunctionalInterface
public interface OrderValidator {
    ValidationResult validate(Order order);
}
```

vs

```java
Function<Order, ValidationResult> validator = ...;   // works but anonymous
```

Custom is better when:
- Domain-meaningful name aids readability
- Method takes multiple args (you'd need BiFunction, or worse)
- You want to add default helper methods
- You want documentation in the contract

---

## Q19. Default methods on interfaces — why and when.

Java 8 added `default` methods so that:
1. Existing interfaces could add methods without breaking implementations (e.g., `Iterable.forEach`, `Collection.stream`)
2. Multiple inheritance of behavior (without state)

```java
public interface Animal {
    String getName();
    default String greet() { return "Hi, I'm " + getName(); }
}
```

###  Diamond problem
If a class implements two interfaces with the same default method, the compiler forces you to disambiguate:
```java
public interface A { default String hi() { return "A"; } }
public interface B { default String hi() { return "B"; } }
public class C implements A, B {
    @Override public String hi() { return A.super.hi(); }       // disambiguate
}
```

###  Class beats interface
If a class inherits a method (concrete OR abstract) AND a default from an interface, the class method wins. The "class is more specific" rule.

###  Don't abuse defaults
- Default = "I needed to evolve this interface without breaking implementers". Not "this is how you should use my class".
- Don't put core domain logic in defaults — it's harder to test, mock, override clearly.

---

## Q20. Stream.iterate vs Stream.generate.

```java
// iterate: seed + UnaryOperator
Stream.iterate(1, i -> i * 2)            // 1, 2, 4, 8, 16, ...

// iterate (Java 9+): with predicate — like a for loop
Stream.iterate(1, i -> i < 100, i -> i * 2)   // 1, 2, 4, 8, 16, 32, 64

// generate: Supplier
Stream.generate(Math::random)            // infinite random doubles
Stream.generate(() -> "constant")        // infinite "constant"
```

###  iterate is sequential
`Stream.iterate(...)` parallelizes poorly — each value depends on the previous. Don't `.parallel()` it.

###  Prefer IntStream.range for index-based loops
```java
Stream.iterate(0, i -> i + 1).limit(n)    // slow, boxes
IntStream.range(0, n)                      // fast, primitive
```
