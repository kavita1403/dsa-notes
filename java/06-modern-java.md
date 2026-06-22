# 06 — Modern Java (Java 9-21+)

Records, sealed classes, pattern matching, text blocks, virtual threads, switch expressions. Each feature exists for a reason — knowing the *why* matters as much as the *how*.

---

## Q1. Java release cadence — how does it work now?

Since Java 9 (2017):
- **New release every 6 months** (March + September)
- **LTS (Long-Term Support) every 2-3 years**: 8, 11, 17, 21, 25 (next)
- Non-LTS releases get 6 months of support
- Most enterprises use LTS — current stable is Java 21 (Sep 2023)

### Feature lifecycle
1. **Incubator** — new API, may change/disappear (`jdk.incubator.*`)
2. **Preview** — language/JVM feature, enable with `--enable-preview`
3. **Standard** — promoted from preview, fully supported

###  Track major LTS migrations
- Java 8 → 11: modules (rarely affects apps), removed APIs (`javax.xml.bind`, `javax.activation`)
- Java 11 → 17: sealed classes, records, pattern matching for instanceof
- Java 17 → 21: virtual threads, pattern matching for switch, sequenced collections

---

## Q2. `var` — local variable type inference (Java 10).

```java
var list = new ArrayList<String>();    // ArrayList<String>
var map = new HashMap<String, Integer>();
var n = 10;                             // int
var d = 3.14;                           // double
var s = "hello";                        // String
```

### Rules
- Local variables only (NOT fields, params, return types)
- Must have an initializer
- Can't be `null` (no type to infer)
- Can't be used for lambda parameters (Java 11 added `var x ->` for that, useful for annotations)

### When `var` aids readability
```java
// Type repetition is obvious noise
var users = new ArrayList<User>();
var iter = users.iterator();
var entry = map.entrySet().iterator().next();

// Generics chains
var groups = list.stream().collect(groupingBy(User::dept));
// vs: Map<Department, List<User>> groups = ...
```

### When `var` hurts readability
```java
var r = process(data);              // what's r? You have to chase the method
var c = svc.create();                // what type? Useless naming
```

###  Java guideline (from Stuart Marks): use `var` if the type is obvious from the initializer OR doesn't matter for the next few lines.

---

## Q3. Switch expressions (Java 14).

The classic `switch` is a **statement** with falls-through, no return value:

```java
// OLD — verbose, fall-through bug-prone
String day;
switch (dayNum) {
    case 1:
    case 7:
        day = "weekend";
        break;
    case 2:
    case 3:
    case 4:
    case 5:
    case 6:
        day = "weekday";
        break;
    default:
        throw new IllegalArgumentException();
}
```

New as an **expression**, returns a value, no fall-through:

```java
// NEW — concise, no fall-through, exhaustive
String day = switch (dayNum) {
    case 1, 7 -> "weekend";                       // arrow form
    case 2, 3, 4, 5, 6 -> "weekday";
    default -> throw new IllegalArgumentException();
};

// Multi-statement block — use yield
int classify = switch (x) {
    case 0 -> 0;
    default -> {
        int temp = compute(x);
        yield temp * 2;
    }
};
```

### Key differences
- `->` arrow form has **no fall-through**
- Each case can be `case A, B, C ->`
- Returns a value (when used as expression)
- Must be **exhaustive** if used as expression (compile error otherwise)

---

## Q4. Pattern matching for `instanceof` (Java 16).

Old: type-check then cast.

```java
// OLD
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// NEW
if (obj instanceof String s) {
    System.out.println(s.length());      // s is in scope and typed
}
```

### Scope rules
The pattern variable is in scope where the test is true:

```java
if (!(obj instanceof String s)) {
    return -1;
}
// s in scope here (because we returned in the false branch)
return s.length();
```

```java
if (obj instanceof String s && !s.isEmpty()) {     // s in scope due to short-circuit
    return s.length();
}
```

---

## Q5. Pattern matching for `switch` (Java 21).

Combines switch expressions with type patterns:

```java
sealed interface Shape permits Circle, Square, Triangle {}
record Circle(double r) implements Shape {}
record Square(double side) implements Shape {}
record Triangle(double a, double b, double c) implements Shape {}

double area = switch (shape) {
    case Circle c -> Math.PI * c.r() * c.r();
    case Square s -> s.side() * s.side();
    case Triangle t -> heron(t.a(), t.b(), t.c());
    // no default needed - sealed type is exhaustive
};
```

### Pattern guards (when clauses)
```java
String describe = switch (obj) {
    case Integer i when i > 0    -> "positive int";
    case Integer i when i < 0    -> "negative int";
    case Integer i               -> "zero";
    case String s when s.isEmpty() -> "empty string";
    case String s                -> "string: " + s;
    case null                    -> "null";
    default                      -> "other";
};
```

### Record patterns (Java 21)
Destructure records inline:

```java
sealed interface Json {}
record JsonString(String value) implements Json {}
record JsonNumber(double value) implements Json {}
record JsonArray(List<Json> items) implements Json {}

String describe = switch (json) {
    case JsonString(String s) -> "str: " + s;
    case JsonNumber(double n) when n == 0 -> "zero";
    case JsonNumber(double n) -> "num: " + n;
    case JsonArray(List<Json> items) -> "arr[" + items.size() + "]";
};
```

---

## Q6. Records (Java 16).

Records are concise immutable data carriers. The compiler generates:
- Private final fields
- All-arg public constructor (canonical)
- Public accessors (no `get` prefix — just `name()`, `age()`)
- `equals`, `hashCode`, `toString`

```java
public record Point(int x, int y) {}

Point p = new Point(3, 4);
p.x();       // 3
p.equals(new Point(3, 4));   // true
p.toString();                 // "Point[x=3, y=4]"
```

### Compact constructor
```java
public record Range(int start, int end) {
    public Range {                                    // compact — no args
        if (start > end) throw new IllegalArgumentException();
    }
}
```

### Custom methods
```java
public record Money(BigDecimal amount, Currency currency) {
    public Money add(Money other) {
        if (!currency.equals(other.currency))
            throw new IllegalStateException("mismatched currency");
        return new Money(amount.add(other.amount), currency);
    }

    public static Money zero(Currency c) {
        return new Money(BigDecimal.ZERO, c);
    }
}
```

### Restrictions
- Implicitly `final` (cannot extend)
- Cannot extend other classes (but CAN implement interfaces)
- Components are final — no setters
- No instance fields beyond the components

### When to use records
- DTOs
- Domain value objects (Money, Coordinate, EmailAddress)
- Tuple returns from methods (`record Pair(L left, R right)`)
- Algebraic data types together with sealed interfaces

###  Defensive copying with records
```java
public record User(String name, List<String> hobbies) {
    public User {
        hobbies = List.copyOf(hobbies);    // defensive copy in compact constructor
    }
}
```

---

## Q7. Sealed classes/interfaces (Java 17).

`sealed` restricts which classes can extend/implement.

```java
public sealed interface Shape permits Circle, Square, Triangle {}

public record Circle(double r) implements Shape {}
public record Square(double side) implements Shape {}
public final class Triangle implements Shape { ... }
```

Permitted subclasses must be:
- `final` (no further extension)
- `sealed` (further sealed)
- `non-sealed` (explicitly open for any extension)

### Why
- **Exhaustive pattern matching** — compiler knows all cases for switch
- **Algebraic data types** in OOP style
- Domain modeling — express "Shape is one of: Circle | Square | Triangle"

### Practical example
```java
public sealed interface PaymentResult {
    record Success(String txnId, BigDecimal amount) implements PaymentResult {}
    record Failure(String code, String message) implements PaymentResult {}
    record Pending(String referenceId) implements PaymentResult {}
}

public String handle(PaymentResult result) {
    return switch (result) {                      // exhaustive
        case Success s -> "OK: " + s.txnId();
        case Failure f -> "FAIL: " + f.message();
        case Pending p -> "WAITING: " + p.referenceId();
        // no default - compiler enforces all cases
    };
}
```

###  When to use sealed
- Closed domain with a known set of variants
- Replacement for enum when variants need to carry different data
- Result/Either/Option-style return types

---

## Q8. Text blocks (Java 15).

Multi-line string literals without escape hell:

```java
String html = """
        <html>
            <body>
                <p>Hello, %s!</p>
            </body>
        </html>
        """.formatted(name);
```

### Indentation rules
- Closing `"""` position determines indentation stripped from each line
- Leading common whitespace removed
- Trailing whitespace stripped (rarely what you want; use `\s` to keep)

### Escape sequences specific to text blocks
- `\` at end of line — suppress newline
- `\s` — literal space (preserved against trailing-whitespace stripping)

```java
String singleLine = """
        the quick \
        brown fox""";
// == "the quick brown fox"
```

### Common use cases
- SQL queries
- HTML/XML/JSON templates
- Code generation
- Multi-line log messages

###  String formatting
```java
// formatted is text-block-friendly
String json = """
    {"name": "%s", "age": %d}
    """.formatted(name, age);

// or use the new String templates (preview in Java 21+)
// String json = STR."""
//     {"name": "\{name}", "age": \{age}}
//     """;
```

---

## Q9. Virtual threads (Java 21).

A virtual thread is JVM-managed, not OS-managed. Cheap to create (millions per JVM).

```java
// Create one
Thread vt = Thread.ofVirtual().start(() -> handleRequest());

// In an executor
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 100_000; i++) {
        executor.submit(() -> processOne());
    }
}     // try-with-resources awaits all
```

### How it works
- N **carrier threads** (typically = CPU count) run virtual threads
- When a virtual thread blocks on I/O, it's **unmounted** from its carrier
- Carrier picks up another virtual thread
- Blocked virtual thread re-mounts when I/O completes

### Why
- Thread-per-request becomes viable again — no need for reactive (RxJava, Project Reactor)
- Same blocking API (`InputStream`, `socket.read`), same `Thread` API
- Massive scalability: 1M concurrent connections without callbacks

### Pinning — when virtual threads CAN'T unmount
- Inside `synchronized` block (relaxed in Java 24)
- Inside JNI / native code
- Some `Object.wait` patterns

If pinned during I/O, you waste a carrier. Fix with `ReentrantLock` instead of `synchronized` for code that does blocking I/O.

###  Don't pool virtual threads
Pooling is for expensive things. Use `Executors.newVirtualThreadPerTaskExecutor()` — create per task, on purpose.

###  Don't put heavy data in ThreadLocal with virtual threads
Each virtual thread has its own ThreadLocal map. Millions of them = OOM. Use `ScopedValue` (preview) instead.

---

## Q10. Structured concurrency (Java 21 preview).

Treats parallel tasks as a single unit of work — cancellation and error propagation become explicit.

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Subtask<User> user = scope.fork(() -> fetchUser(userId));
    Subtask<List<Order>> orders = scope.fork(() -> fetchOrders(userId));
    Subtask<List<Notif>> notifs = scope.fork(() -> fetchNotifs(userId));

    scope.join();                  // wait for all to finish or fail
    scope.throwIfFailed();         // re-throw exceptions from any subtask

    return new Response(user.get(), orders.get(), notifs.get());
}
```

If any subtask fails, others are cancelled. If the enclosing scope is interrupted, all subtasks are interrupted.

### Two policies
- `ShutdownOnFailure` — first failure cancels siblings (like Promise.all rejection)
- `ShutdownOnSuccess<T>` — first success cancels others (like Promise.any)

Solves the "leaky tasks" problem you get with raw CompletableFuture chains.

---

## Q11. Modules — Java Platform Module System (JPMS, Java 9).

A module is a named, versioned, self-describing unit. Each module has `module-info.java`:

```java
module com.example.app {
    requires java.sql;                       // depends on java.sql
    requires transitive com.example.api;     // anyone using me also needs api
    exports com.example.app.public_api;      // these packages visible to others
    exports com.example.app.internal to com.example.test;  // qualified export
    opens com.example.app.entity to hibernate.core;  // reflection access
}
```

### Why it exists
- Strong encapsulation: even `public` classes are hidden unless `exports`ed
- Reliable configuration: missing/conflicting modules detected at startup
- Better startup performance + smaller distribution (jlink lets you build a custom JRE)

### Reality
- Most apps still use classpath, not modulepath
- JDK itself is modularized (you get the benefits — smaller, faster startup)
- Library ecosystem partial — many libs are "automatic modules" (legacy jars treated as modules)
- Compelling for: large monoliths needing internal isolation, distributable apps with jlink

---

## Q12. New HTTP Client (Java 11).

Replaces `HttpURLConnection` (terrible API). Standard since Java 11.

```java
HttpClient client = HttpClient.newBuilder()
    .version(Version.HTTP_2)
    .connectTimeout(Duration.ofSeconds(10))
    .followRedirects(Redirect.NORMAL)
    .build();

HttpRequest req = HttpRequest.newBuilder(URI.create("https://api.example.com/users"))
    .header("Authorization", "Bearer " + token)
    .timeout(Duration.ofSeconds(30))
    .POST(BodyPublishers.ofString("""
        {"name": "Alice"}
        """))
    .build();

// Sync
HttpResponse<String> resp = client.send(req, BodyHandlers.ofString());
int status = resp.statusCode();
String body = resp.body();

// Async
CompletableFuture<HttpResponse<String>> future =
    client.sendAsync(req, BodyHandlers.ofString());
future.thenAccept(r -> System.out.println(r.body()));
```

### Features
- HTTP/2 support
- WebSocket support (built-in)
- Async via `CompletableFuture`
- Built-in JSON? No — that's still your concern (Jackson, etc.)

---

## Q13. var in lambda params (Java 11).

Useful when you need annotations on lambda parameters:

```java
list.stream().map((@NonNull var x) -> x.toString());
```

Otherwise, `var` adds nothing — lambda parameter types are already inferred.

---

## Q14. Compact Strings (Java 9).

`String` internally stores `byte[]` instead of `char[]`. If all chars fit in Latin-1 (most strings), uses 1 byte per char (half the memory). Otherwise, uses UTF-16 (2 bytes).

### Impact
- ~5-10% heap reduction in typical apps
- Slightly slower for non-Latin-1 strings (encoding flag check)
- Transparent — no API change

### Combine with `-XX:+UseStringDeduplication` (G1)
Detects duplicate string contents across the heap and shares the backing array. Even more memory savings for string-heavy apps.

---

## Q15. JShell — the REPL (Java 9).

Interactive shell for trying snippets:

```
$ jshell
jshell> int x = 5
jshell> int y = 10
jshell> x + y
$3 ==> 15

jshell> List<Integer> nums = List.of(1, 2, 3)
jshell> nums.stream().mapToInt(i -> i).sum()
$5 ==> 6
```

Great for learning, testing API behavior, quick calculations.

---

## Q16. Files / NIO improvements you should know.

```java
// Read entire file (Java 11+)
String content = Files.readString(path);
String content = Files.readString(path, StandardCharsets.UTF_8);

// Write entire file
Files.writeString(path, "content");

// Stream file lines (auto-closes with try-with-resources)
try (Stream<String> lines = Files.lines(path)) {
    lines.filter(...).forEach(...);
}

// Walk a directory
try (Stream<Path> paths = Files.walk(root)) {
    paths.filter(Files::isRegularFile).forEach(...);
}

// Find with predicate
try (Stream<Path> finds = Files.find(root, 10,
        (p, attrs) -> attrs.isRegularFile() && p.toString().endsWith(".java"))) {
    finds.forEach(System.out::println);
}
```

###  Always use try-with-resources for file streams
Otherwise file handles leak — eventual OS limit exhaustion.

---

## Q17. Java 16+ — `Stream.toList()` and `Stream.mapMulti()`.

```java
// toList — immutable, allows nulls, cleaner than collect(toList())
List<String> result = stream.toList();

// mapMulti — single op for "map + flatMap" patterns
List<String> result = stream
    .<String>mapMulti((item, downstream) -> {
        if (condition(item)) {
            downstream.accept(transform1(item));
            downstream.accept(transform2(item));
        }
    })
    .toList();
```

`mapMulti` is for cases where flatMap would require creating a Stream for each element (overhead). Push 0-N elements directly to the downstream consumer.

---

## Q18. Sequenced Collections (Java 21).

Adds `SequencedCollection`, `SequencedSet`, `SequencedMap` interfaces with common methods for ordered collections:

```java
list.getFirst();    list.getLast();
list.addFirst(x);   list.addLast(x);
list.removeFirst(); list.removeLast();
list.reversed();    // reverse view (not a copy)

map.firstEntry();   map.lastEntry();
map.reversed();
```

Finally — uniform first/last methods across ArrayList, LinkedList, LinkedHashMap, TreeMap, etc. Previously you had to know which collection used `getFirst`, which used `peek`, which used `iterator().next()`.

---

## Q19. APIs removed/deprecated in recent Java.

### Removed
- `sun.misc.Unsafe` (some methods) — use `VarHandle`
- `javax.xml.bind` / JAXB (Java 11) — add as separate dep
- `javax.activation` (Java 11)
- `Nashorn` JavaScript engine (Java 15) — use GraalJS
- `CMS` GC (Java 14) — use G1 or ZGC
- `Thread.stop`, `Thread.suspend`, `Thread.resume` (Java 21)
- `finalize()` (deprecated Java 9; removal in progress)
- `Applet` API (Java 11)
- `SecurityManager` (deprecated Java 17, target removal)

### Deprecated for removal
- `URL` constructors (Java 20+) — use `URI` instead
- `Subject.doAs` (Java 18)

---

## Q20. What's coming — features to watch.

### Preview/incubator features in Java 21-25
- **Value classes / Project Valhalla** — primitive-like classes, no identity, flat memory layout
- **String templates** — `STR."Hello \{name}"` — type-safe interpolation
- **Universal generics** — generics over primitives (no more `List<Integer>` boxing)
- **Stream gatherers** — custom intermediate operations (like collectors but for in-the-middle)
- **Class-file API** — programmatic bytecode generation/parsing
- **Foreign Function & Memory API** — modern alternative to JNI for native interop

### Production-ready trends
- **Virtual threads** — replace reactive code in most cases
- **Pattern matching** — converging with Scala-style ADTs
- **Generational ZGC** — default low-latency choice
- **Records + sealed + pattern matching** — domain modeling renaissance

###  Watch JEP (JDK Enhancement Proposal) list
https://openjdk.org/jeps/0 — every Java change starts there. Look for "Status: Targeted to JDK XX" for what's coming next.

---

## Q21. Migration tips: Java 8 → Java 21.

### Easy wins (often source-compatible)
- Use `var` for local variables
- Use records for DTOs
- Use text blocks for SQL/JSON/HTML literals
- Use switch expressions in place of switch statements
- Use `Files.readString` instead of manual streams

### Mid-effort
- Replace `Hashtable` → `ConcurrentHashMap`
- Replace `Vector` → `ArrayList` or sync wrapper
- Replace `Date` / `Calendar` → `java.time` (LocalDate, ZonedDateTime, Instant)
- Replace `SimpleDateFormat` → `DateTimeFormatter` (thread-safe!)
- Replace `Optional.get()` calls → `orElseThrow()` for clarity
- Replace defensive `new ArrayList<>(other)` returns with `List.copyOf(other)`

### Bigger lifts
- Reactive code (RxJava/Reactor) → virtual threads + structured concurrency
- Old XML/JAXB → bring in separate dependency or migrate to JSON
- `SecurityManager`-dependent code → other security mechanisms
- Hand-rolled thread pools → virtual threads where applicable
- Enums-with-methods-and-state → sealed records

### Compatibility tools
- `jdeps -jdkinternals` — find internal API usage
- `jdeprscan` — scan for deprecated APIs
- `--release 8` compile flag — target older runtime from a newer JDK

---

## Q22. Performance changes worth knowing.

### Faster startup (Java 9+)
- AppCDS (Application Class Data Sharing) — pre-load class metadata
- jlink — custom small runtime images
- Compact strings — less memory pressure

### Faster throughput
- New G1 (default since 9) better tuned
- ZGC sub-ms pauses
- JIT improvements every release
- Compressed oops by default

### Faster I/O
- Built-in HTTP client
- Asynchronous file I/O (`AsynchronousFileChannel`)
- Virtual threads — millions of concurrent I/O operations

###  Benchmark before optimizing
Use JMH. Micro-benchmarks without JMH lie because of JIT warmup, dead code elimination, etc.

```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@State(Scope.Benchmark)
public class MyBench {
    @Benchmark
    public long parseDate() { return LocalDate.parse("2024-01-01").toEpochDay(); }
}
```

---

## Q23. Quick reference — feature → release.

| Feature | Java | LTS |
|---|---|---|
| Lambdas, Streams, Optional, java.time | 8 | LTS |
| Modules (JPMS) | 9 | |
| JShell, Process API, Stream.iterate w/ predicate | 9 | |
| `var` for locals | 10 | |
| HTTP Client (standard), `var` in lambda params, Files.readString | 11 | LTS |
| Switch expressions (standard) | 14 | |
| Text blocks (standard), `instanceof` pattern (preview) | 15 | |
| Records (standard), pattern matching `instanceof` (standard) | 16 | |
| Sealed classes (standard) | 17 | LTS |
| `Stream.toList()` | 16 | |
| Simple Web Server, UTF-8 default | 18 | |
| Pattern matching for switch (preview), Virtual threads (preview), Structured concurrency (incubator) | 19/20 | |
| Virtual threads (standard), Pattern matching for switch (standard), Record patterns, Sequenced Collections, Generational ZGC | 21 | LTS |
| String templates (preview), Stream gatherers (preview), Scoped values (preview) | 21-22 | |

Java 25 (LTS) lands Sep 2025 — many previews graduating to standard.
