# 01 — Core Java

OOP, equals/hashCode, immutability, generics, exceptions, primitives/wrappers. The foundation.

---

## Q1. What's the difference between `==` and `.equals()`?

**Short answer**: `==` compares **references** (memory addresses) for objects, **values** for primitives. `.equals()` compares **logical equality** as defined by the class.

```java
String a = new String("hello");
String b = new String("hello");
System.out.println(a == b);       // false  (different objects)
System.out.println(a.equals(b));  // true   (same content)

String x = "hello";
String y = "hello";
System.out.println(x == y);       // true   (string pool — same reference)
```

###  Common pitfall — autoboxing + Integer cache
```java
Integer a = 127, b = 127;
Integer c = 128, d = 128;
System.out.println(a == b);   // true   — Integer cache (-128 to 127)
System.out.println(c == d);   // false  — outside cache, new objects
```
**Always use `.equals()` for boxed types.** The `-128..127` cache is in `Integer.IntegerCache.cache[]`.

---

## Q2. Explain the `equals()` / `hashCode()` contract.

The **single most violated contract** in Java. Four rules:

1. **Reflexive**: `x.equals(x)` is always `true`
2. **Symmetric**: `x.equals(y) == y.equals(x)`
3. **Transitive**: if `x.equals(y)` and `y.equals(z)`, then `x.equals(z)`
4. **Consistent**: repeated calls return the same result (assuming no field changes)

**The killer rule**: **`a.equals(b) ⇒ a.hashCode() == b.hashCode()`**

Breaking this → `HashMap`/`HashSet` silently lose entries. Two equal objects landing in different buckets means `get()` will never find one of them.

```java
public final class User {
    private final long id;
    private final String email;

    public User(long id, String email) {
        this.id = id;
        this.email = email;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                  // fast path
        if (!(o instanceof User u)) return false;    // type check + pattern var (Java 16+)
        return id == u.id && Objects.equals(email, u.email);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, email);  // never roll your own — use Objects.hash
    }
}
```

###  Pro tip — use `record` for value classes (Java 16+)
```java
public record User(long id, String email) {}
// equals(), hashCode(), toString(), accessors all generated. Done.
```

###  Asymmetric equals with inheritance
```java
class Point { int x, y; }
class ColoredPoint extends Point { Color color; }
// If ColoredPoint.equals checks color but Point.equals doesn't,
// p.equals(cp) might be true but cp.equals(p) is false → contract broken
```
**Fix**: use `getClass() == o.getClass()` instead of `instanceof` if you have subclasses — or use composition over inheritance.

---

## Q3. What's the difference between `String`, `StringBuilder`, and `StringBuffer`?

| | `String` | `StringBuilder` | `StringBuffer` |
|---|---|---|---|
| Mutable? | No | Yes | Yes |
| Thread-safe? | Yes (immutable) | **No** | Yes (synchronized) |
| Performance | Slow for concat | Fastest | Slower (lock overhead) |
| Use when | Read-only / few changes | Loop building, single thread | Multi-threaded mutation (rare) |

```java
// BAD — creates n intermediate String objects, O(n²) overall
String s = "";
for (int i = 0; i < 1000; i++) s += i;

// GOOD — O(n)
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) sb.append(i);
String s = sb.toString();
```

###  Compiler optimization
Single-line `"a" + "b" + var` → compiler uses `StringBuilder` for you (Java 8). On Java 9+, it uses `invokedynamic` + `StringConcatFactory` which is even faster. So loop concatenation is the only place this matters.

###  String pool
String literals live in the **String Constant Pool** (part of the heap since Java 7). `intern()` puts a runtime-created string into the pool. Rarely needed — JVM handles dedup with `-XX:+UseStringDeduplication` (G1).

---

## Q4. Why is `String` immutable?

Five reasons:

1. **Security** — passwords/URLs can't be modified after validation (TOCTOU attacks)
2. **String pool sharing** — safe to dedupe identical strings across the JVM
3. **Thread safety** — share freely without synchronization
4. **HashMap key safety** — hashcode is computed once, cached, never changes
5. **Class loading** — class names must be stable

The field is `private final char[] value` (Java 8) or `private final byte[] value` (Java 9+ Compact Strings — latin1 strings take half the memory).

---

## Q5. Explain `final`, `finally`, and `finalize()`.

| Keyword | Meaning |
|---|---|
| `final` (variable) | Cannot reassign (still mutable if it's an object) |
| `final` (method) | Cannot override |
| `final` (class) | Cannot extend (e.g., `String`, `Integer`) |
| `finally` | Block that always runs after try/catch (even on return, except `System.exit`) |
| `finalize()` | **DEPRECATED since Java 9, removed in Java 18.** Use `try-with-resources` or `Cleaner` instead. |

###  finalize() pitfalls (historical)
- Not guaranteed to run
- Runs on a single GC thread → can become bottleneck
- Can resurrect objects → unpredictable
- Replaced by `java.lang.ref.Cleaner` (Java 9+) for native resource cleanup

---

## Q6. What is `try-with-resources` and why use it?

Auto-closes any resource implementing `AutoCloseable` (or `Closeable`). Syntactic sugar for try/finally with proper exception suppression.

```java
// OLD — error-prone, exception in close() can mask original exception
BufferedReader r = null;
try {
    r = new BufferedReader(new FileReader("f.txt"));
    return r.readLine();
} finally {
    if (r != null) r.close();
}

// NEW — clean, safe, properly suppresses secondary exceptions
try (BufferedReader r = new BufferedReader(new FileReader("f.txt"))) {
    return r.readLine();
}

// Multiple resources — closed in REVERSE order of declaration
try (var conn = ds.getConnection();
     var stmt = conn.prepareStatement(sql);
     var rs   = stmt.executeQuery()) {
    // use rs
}
```

###  Subtle detail — exception suppression
If both the try block AND the close() throw, the close() exception is **suppressed** (attached via `Throwable.getSuppressed()`), and the original is rethrown. Without this, you'd silently lose the real error.

---

## Q7. Checked vs unchecked exceptions — when to use each?

| Type | Extends | Must catch/declare? | Examples |
|---|---|---|---|
| **Checked** | `Exception` (not `RuntimeException`) | Yes (compile error otherwise) | `IOException`, `SQLException` |
| **Unchecked** | `RuntimeException` | No | `NullPointerException`, `IllegalArgumentException` |
| **Error** | `Error` | No (don't catch) | `OutOfMemoryError`, `StackOverflowError` |

###  Modern guidance
- **Unchecked is the default** — checked exceptions create boilerplate and don't compose with lambdas/streams
- Use checked **only** when the caller can reasonably recover (e.g., retry network failure)
- Spring, JOOQ, modern frameworks → all unchecked
- **Never** throw `Exception` or `Throwable` directly; never catch `Exception` (too broad)

###  Anti-patterns
```java
catch (Exception e) { }                          // swallowing — never do this
catch (Exception e) { e.printStackTrace(); }     // not enough; use a logger
catch (Exception e) { throw new RuntimeException(e); }  // chain it properly, don't lose stack
throw new RuntimeException("oops");              // no cause, no message details
```

###  Good pattern
```java
catch (IOException e) {
    log.error("Failed to read config from {}", path, e);
    throw new ConfigLoadException("Cannot load " + path, e);
}
```

---

## Q8. Explain generics and type erasure.

Java generics are **compile-time only** — at runtime, the type parameter is erased to its bound (default `Object`). This enables backward compatibility with pre-generics code.

```java
List<String> a = new ArrayList<>();
List<Integer> b = new ArrayList<>();
System.out.println(a.getClass() == b.getClass());  // true — both are ArrayList at runtime
```

###  Consequences of erasure

1. **No `new T()`** — type info is gone at runtime. Workaround: pass `Class<T>` or `Supplier<T>`.
2. **No `instanceof List<String>`** — only `instanceof List<?>`
3. **No generic arrays** — `new T[10]` doesn't compile
4. **Method signatures clash**:
   ```java
   void m(List<String> l) {}
   void m(List<Integer> l) {}  // ERROR — same erasure
   ```

###  PECS — Producer Extends, Consumer Super

```java
// You READ from src (producer) → ? extends T
// You WRITE to dst (consumer)  → ? super T
public static <T> void copy(List<? extends T> src, List<? super T> dst) {
    for (T t : src) dst.add(t);
}
```

This lets `copy(List<Integer>, List<Number>)` work — `Integer` produces (extends Number), `Number` consumes (super Integer).

###  Bounded type parameter
```java
public static <T extends Comparable<T>> T max(List<T> list) {
    T best = list.get(0);
    for (T t : list) if (t.compareTo(best) > 0) best = t;
    return best;
}
```

---

## Q9. What's the difference between an abstract class and an interface?

| | Abstract class | Interface |
|---|---|---|
| Fields | Any (instance + static) | `public static final` only |
| Methods | Any visibility, abstract or concrete | `public` (default/static/private since Java 8/9) |
| Constructor | Yes | No |
| Multiple inheritance | No | Yes (multi-implement) |
| State | Yes | No (only constants) |
| Use case | "is-a" with shared state/behavior | Capability / contract |

###  Java 8+ blurred the lines
- Interfaces can have `default` methods (shared default implementations)
- Interfaces can have `static` methods
- Interfaces can have `private` methods (Java 9+) to share code between defaults

###  When to choose which
- **Interface** by default — favor composition, easier to mock/test, supports multiple inheritance
- **Abstract class** when you have **shared state** or want to lock down a template method pattern
- **`sealed`** (Java 17+) when you want a closed hierarchy — see modern-java notes

---

## Q10. Explain Java's pass-by-value behavior.

**Java is ALWAYS pass-by-value.** No exceptions. But for objects, the value being passed is a **reference** (essentially a pointer).

```java
void swap(int a, int b) { int t = a; a = b; b = t; }  // doesn't swap caller's vars

void mutate(List<String> l) { l.add("hi"); }          // caller's list IS modified

void reassign(List<String> l) { l = new ArrayList<>(); l.add("bye"); }
// caller's list NOT changed — we reassigned a local copy of the reference
```

The mental model: when you pass an object, you pass a **copy of the reference**. Both the caller and callee point at the same object, so mutations are visible. But reassigning the local var only changes the local copy.

###  Interview trap
"Java passes objects by reference" — **false**. It passes the reference *by value*. Subtle but matters.

---

## Q11. What is the difference between `this` and `super`?

- `this` → current instance reference
- `super` → parent class reference (enables calling overridden methods / parent constructor)

```java
class Parent {
    String name = "parent";
    void greet() { System.out.println("Hello from " + name); }
}

class Child extends Parent {
    String name = "child";

    Child() {
        super();              // must be first statement (implicit if omitted)
    }

    void greet() {
        super.greet();        // calls Parent.greet()
        System.out.println("Hello from " + name);    // "child"
        System.out.println("And from " + super.name); // "parent" (field hiding)
    }
}
```

###  Field hiding vs method overriding
- Fields are **hidden** (resolved at compile time, based on reference type)
- Methods are **overridden** (resolved at runtime, based on actual object type — polymorphism)

```java
Parent p = new Child();
p.greet();        // "Hello from parent" then "Hello from child" — method overridden
System.out.println(p.name);  // "parent" — field hidden, not overridden!
```

**Never hide fields in real code.** It's a footgun.

---

## Q12. Explain method overloading vs overriding.

| | Overloading | Overriding |
|---|---|---|
| Where | Same class | Subclass overrides parent |
| Signatures | Different (param types/count) | Identical |
| Return type | Can vary | Same or covariant |
| Resolution | Compile-time (static dispatch) | Runtime (dynamic dispatch) |
| Annotation | None | `@Override` (use it!) |

```java
// Overloading
class Calc {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
}

// Overriding
class Animal { void speak() { System.out.println("..."); } }
class Dog extends Animal {
    @Override
    void speak() { System.out.println("Woof"); }
}
```

###  `@Override` is not optional
It's a **safety net** — if you misspell the method name or get the signature wrong, the compiler catches it. Without it, you silently create a new method that never gets called via polymorphism.

---

## Q13. What are the access modifiers in Java?

| Modifier | Class | Package | Subclass | World |
|---|---|---|---|---|
| `public` |  |  |  |  |
| `protected` |  |  |  |  |
| (default / package-private) |  |  |  |  |
| `private` |  |  |  |  |

###  Modern modules (Java 9+)
Even `public` doesn't mean "accessible to everyone" — it must also be `exports`ed in `module-info.java`. This enables strong encapsulation at the module level.

---

## Q14. Primitive vs wrapper types — performance and pitfalls.

| Primitive | Wrapper | Bytes |
|---|---|---|
| `boolean` | `Boolean` | 1 |
| `byte` | `Byte` | 1 |
| `short` | `Short` | 2 |
| `char` | `Character` | 2 |
| `int` | `Integer` | 4 |
| `long` | `Long` | 8 |
| `float` | `Float` | 4 |
| `double` | `Double` | 8 |

###  Performance
- Primitive `int[1000]` = 4 KB. `Integer[1000]` = 4 KB of refs + 16 KB of `Integer` objects = **20 KB and N cache misses**.
- Use primitives in hot paths. Use `long[]`, not `List<Long>`, for big numeric data.
- For primitive-keyed maps with no boxing: **Eclipse Collections** or **Fastutil** (`IntObjectHashMap`, `LongOpenHashSet`).

###  Autoboxing NPE
```java
Map<String, Integer> counts = new HashMap<>();
int n = counts.get("missing");  // NPE — null can't be unboxed
// Fix:
int n = counts.getOrDefault("missing", 0);
```

---

## Q15. What is the difference between `static` and instance members?

- **Instance** members belong to each object. Each new object has its own copy.
- **Static** members belong to the class. One copy shared by all instances.

```java
class Counter {
    private static int totalCreated = 0;
    private int id;

    Counter() {
        id = ++totalCreated;
    }
}
```

###  Static gotchas
- **Cannot override static methods** — they're hidden, not overridden. Resolution is based on the reference type.
- **Static init order**: parent class static blocks → child class static blocks → parent instance init + constructor → child instance init + constructor.
- **Avoid mutable static state** — it's hidden global state, hard to test, breaks parallel tests.

###  Static factory methods
Often better than constructors — they have names, can return subtypes, can cache:
```java
public static User of(long id, String email) {
    return new User(id, email);
}
List.of("a", "b", "c");       // Java 9+ immutable list
Optional.of(value);
```

---

## Q16. Immutability — why and how?

**Why**:
- Thread-safe by definition (no shared mutable state)
- Safe as `HashMap` keys (hashcode never changes)
- Easier to reason about (no aliasing bugs)
- Defensive copying not needed

**How** — six rules:
1. Class is `final` (or use sealed)
2. All fields are `private final`
3. No setters; only constructor sets state
4. Defensive-copy mutable inputs in the constructor
5. Defensive-copy mutable returns in getters
6. Don't `this`-leak in constructor (don't pass `this` to other methods before init done)

```java
public final class ImmutablePerson {
    private final String name;
    private final List<String> hobbies;       // List is mutable → need defensive copies

    public ImmutablePerson(String name, List<String> hobbies) {
        this.name = name;
        this.hobbies = List.copyOf(hobbies);  // Java 10+ immutable copy
    }

    public String name() { return name; }
    public List<String> hobbies() { return hobbies; }  // already immutable
}
```

###  records (Java 16+)
```java
public record Person(String name, List<String> hobbies) {
    public Person {                            // compact constructor
        hobbies = List.copyOf(hobbies);        // defensive copy
    }
}
```

That's a fully immutable, equals/hashCode/toString-correct value class in **3 lines**.

---

## Q17. What is reflection? When should you use it?

Reflection lets you inspect and manipulate classes/methods/fields at runtime via `java.lang.reflect`.

```java
Class<?> c = Class.forName("com.example.User");
Method m = c.getDeclaredMethod("setEmail", String.class);
m.setAccessible(true);
m.invoke(userInstance, "new@email.com");
```

###  Use cases
- ORM (Hibernate maps DB columns → fields)
- DI containers (Spring instantiates beans)
- Serialization (Jackson reads/writes private fields)
- Test mocking (Mockito)

###  Costs
- **Slow** — bypasses JIT optimization. Cache the `Method`/`Field` lookup.
- **Breaks encapsulation** — can access privates, violating the design.
- **Breaks at runtime** — refactoring tools won't catch string-based reflection.
- **Doesn't work in native images** (GraalVM) without explicit config.

###  Modern alternatives
- `MethodHandle` / `VarHandle` (Java 7/9+) — faster than reflection
- Annotation processors / records — compile-time codegen
- `java.lang.classfile` API (preview) — modern bytecode manipulation

---

## Q18. What is the difference between `Comparable` and `Comparator`?

| | `Comparable<T>` | `Comparator<T>` |
|---|---|---|
| Where defined | Inside the class | Outside (separate object) |
| Method | `compareTo(T other)` | `compare(T a, T b)` |
| Number of orderings | One (natural ordering) | Many |

```java
// Natural ordering
public class User implements Comparable<User> {
    String name; int age;
    @Override
    public int compareTo(User o) { return Integer.compare(age, o.age); }
}

// Custom orderings
Comparator<User> byName = Comparator.comparing(u -> u.name);
Comparator<User> byAgeDesc = Comparator.comparingInt((User u) -> u.age).reversed();
Comparator<User> byNameThenAge = Comparator.comparing((User u) -> u.name)
                                           .thenComparingInt(u -> u.age);

// Null-safe
Comparator<User> safe = Comparator.comparing(User::getName,
                          Comparator.nullsFirst(Comparator.naturalOrder()));
```

###  Subtle contract
`compareTo` should be **consistent with equals**: `a.compareTo(b) == 0` should imply `a.equals(b)`. Violating this breaks `TreeMap`/`TreeSet` (which use compareTo, not equals).

Classic gotcha: `BigDecimal("1.0").compareTo(BigDecimal("1.00")) == 0` but they're not `equals()`. Don't put `BigDecimal`s in a `TreeSet` if scale matters.

---

## Q19. What are the rules for constructor inheritance and chaining?

- Constructors are **NOT inherited**.
- Every constructor implicitly calls `super()` as the first statement (unless you call `this(...)` or `super(...)` explicitly).
- If parent has no no-arg constructor, child MUST explicitly call a `super(args)`.

```java
class Animal {
    Animal(String name) { ... }       // no no-arg ctor
}

class Dog extends Animal {
    Dog() {                            // ERROR — implicit super() doesn't exist
        // ...
    }
    Dog(String name) {
        super(name);                   // OK
    }
}
```

###  Order of init when `new Child()`
1. Allocate memory, set all fields to default (0/null/false)
2. Recursively call `super(...)` up to `Object`
3. Initialize instance fields top-to-bottom (incl. instance initializer blocks)
4. Run the constructor body

This is why calling overridable methods from constructors is dangerous — the child's fields aren't initialized yet.

---

## Q20. What is the `Object` class, and what methods does every object have?

`java.lang.Object` is the root of every class hierarchy. Methods every object inherits:

| Method | Purpose |
|---|---|
| `equals(Object)` | Logical equality |
| `hashCode()` | Hash for bucketing |
| `toString()` | String representation (default = `ClassName@hexHash` — almost always wrong) |
| `getClass()` | Runtime class object |
| `clone()` | Field-by-field copy (rarely used correctly — prefer copy constructors / records) |
| `finalize()` | DEPRECATED — removed in Java 18 |
| `wait()` / `notify()` / `notifyAll()` | Legacy thread coordination (prefer `java.util.concurrent`) |

###  Always override
- `equals()` + `hashCode()` together if you'll use as map key or in a set
- `toString()` for any debuggable domain object — your future-self at 2am will thank you

###  records do all this for free
```java
record Money(BigDecimal amount, Currency currency) {}
// equals, hashCode, toString, accessors — all generated correctly
```
