# 04 — JVM Memory & Garbage Collection

Heap structure, GC algorithms (G1/ZGC/Shenandoah), classloading, JIT, OOM debugging. Required for any prod-grade work.

---

## Q1. Describe the JVM memory areas.

```
+-----------------------------------------+
|              JVM Memory                 |
+-----------------------------------------+
| Heap        (shared, GC-managed)        |
|   - Young Gen                           |
|     - Eden                              |
|     - Survivor S0, S1                   |
|   - Old Gen (Tenured)                   |
|                                         |
| Metaspace   (since Java 8; was PermGen) |
|   - Class metadata, method bytecode     |
|   - Native memory, NOT heap             |
|                                         |
| Code Cache  (JIT-compiled native code)  |
|                                         |
| Thread Stacks (one per thread)          |
|   - Local primitives, refs              |
|   - Method call frames                  |
|                                         |
| Direct Memory (ByteBuffer.allocateDirect)|
|   - Off-heap, used by NIO, Netty        |
+-----------------------------------------+
```

### Per area
| Area | Purpose | Limit |
|---|---|---|
| Heap | Objects, arrays | `-Xms`, `-Xmx` |
| Metaspace | Class metadata | `-XX:MaxMetaspaceSize` (default: unlimited until OS OOM) |
| Code Cache | JIT output | `-XX:ReservedCodeCacheSize` |
| Thread Stack | Per thread | `-Xss` (default 1MB on 64-bit) |
| Direct Memory | NIO buffers | `-XX:MaxDirectMemorySize` |

###  Common confusion
- **PermGen is GONE** (removed in Java 8). Replaced by Metaspace.
- Metaspace lives in **native memory**, not heap.
- `String` literals were in PermGen pre-Java 7, moved to heap (interned in the String pool) Java 7+.

---

## Q2. Heap generations — why split?

**Generational hypothesis**: most objects die young. Confirmed empirically — 90%+ of new allocations are short-lived (loop temps, builder objects, etc.).

### Young Generation
- **Eden**: where new objects are born
- **Survivor S0, S1**: two equal-size spaces; objects survive a Young GC by being copied between them
- Young GC (minor GC) — frequent, fast (ms), STW (stop-the-world)
- Copy-collection: only live objects copied; dead objects implicitly discarded

### Old Generation
- Objects that survive enough Young GCs are **promoted/tenured** to Old
- Promotion threshold: `-XX:MaxTenuringThreshold` (default 15, adaptive)
- Old GC (major / full GC) — slower, larger pauses

### The flow
```
new Object()
    -> Eden
[Eden full]
    -> Young GC: live objects -> S0 (age=1)
[next GC]: S0 + new Eden survivors -> S1 (age=2)
[next GC]: S1 + new Eden survivors -> S0 (age=3)
...
[age == MaxTenuringThreshold]
    -> Old Gen (promoted)
```

### Premature promotion = bad
If objects move to Old Gen before they die, you've defeated the optimization. Tune `-XX:NewRatio`, `-XX:SurvivorRatio`, watch GC logs for age distribution.

---

## Q3. TLAB and object allocation.

### Thread-Local Allocation Buffer
Each thread gets a slice of Eden — its own TLAB. Allocation = bump a pointer (no contention).

```
Eden
+------+------+------+------+--------+
| T1   | T2   | T3   | T4   |  free  |
| TLAB | TLAB | TLAB | TLAB |        |
+------+------+------+------+--------+
        ^
        bump-pointer per thread
```

- `-XX:+UseTLAB` (default on)
- Allocation is ~10 CPU instructions, faster than malloc
- When TLAB fills, thread grabs another slice atomically

### Escape Analysis
JIT analyzes if an object's reference escapes a method/thread. If not:
- **Stack allocation** — entire object on the stack, no heap pressure
- **Scalar replacement** — break object into individual fields stored in registers

```java
void m() {
    Point p = new Point(1, 2);    // may never hit the heap
    int x = p.x + p.y;
}
```

This is why micro-benchmarks of "is `new` slow?" mislead — modern JIT often elides the allocation entirely.

---

## Q4. The major GC algorithms.

### Serial GC (`-XX:+UseSerialGC`)
- Single-threaded, STW for both young and old
- Suited for small heaps (< 100 MB) and single-CPU
- Default for client mode

### Parallel GC (`-XX:+UseParallelGC`)
- Multi-threaded STW collection
- Optimizes throughput (more useful work per unit time)
- Was default before Java 9
- Suitable for batch jobs

### CMS (Concurrent Mark Sweep, `-XX:+UseConcMarkSweepGC`)
- **REMOVED in Java 14**
- Concurrent old-gen collection, parallel young
- Suffered from fragmentation, no compaction → eventual full-GC

### G1 GC (`-XX:+UseG1GC`)
- **Default since Java 9**
- Region-based: heap split into ~2000 regions of 1-32 MB
- Young/Old/Humongous regions, mix-and-match
- Predictable pauses: `-XX:MaxGCPauseMillis=200` (target, not guarantee)
- Concurrent marking + parallel STW evacuation
- **Suitable for: 4-64 GB heaps, latency-sensitive but tolerant of ~100ms pauses**

### ZGC (`-XX:+UseZGC`)
- Sub-millisecond pause times, regardless of heap size
- Concurrent everything (mark, relocate, remap) — uses colored pointers + load barriers
- Production-ready since Java 15; generational since Java 21
- **Suitable for: large heaps (up to multi-TB), latency-critical apps**

### Shenandoah (`-XX:+UseShenandoahGC`)
- Similar goals to ZGC (sub-ms pauses)
- Uses Brooks pointers / load reference barriers
- OpenJDK contribution from Red Hat
- **Suitable for: same as ZGC**

### Epsilon (`-XX:+UseEpsilonGC`)
- "No-op" collector — never collects
- For ultra-short-lived JVMs (e.g., serverless) or perf testing

### How to choose (rough guide)
| Heap | Latency sensitivity | Pick |
|---|---|---|
| < 4 GB, batch | Throughput | Parallel |
| 4-32 GB, general server | Mixed | **G1** (default) |
| Any size, latency-critical | Low latency | **ZGC** or Shenandoah |
| Single-threaded tiny | Throughput | Serial |

---

## Q5. G1 GC — describe how it works.

```
Heap split into ~2000 regions:
+---+---+---+---+---+---+---+---+
| E | E | O | S | E | H | F | O |
+---+---+---+---+---+---+---+---+
| O | E | O | O | F | E | S | E |
+---+---+---+---+---+---+---+---+
...
E = Eden, S = Survivor, O = Old, H = Humongous, F = Free
```

### Phases

1. **Young Collection** (STW, parallel)
   - Collects only young regions
   - Evacuates live objects to survivor or old regions

2. **Concurrent Marking** (mostly concurrent)
   - Triggered when Old occupancy > `InitiatingHeapOccupancyPercent` (default 45%)
   - Initial mark (STW, piggybacked on young GC)
   - Concurrent root scan
   - Concurrent marking
   - Remark (STW, short)
   - Cleanup — identify regions with most garbage

3. **Mixed Collection** (STW)
   - Collects young + some old regions ("collection set")
   - Picks regions with most garbage first (Garbage-First!)

4. **Humongous Allocation**
   - Objects > half a region size allocated directly to contiguous Old regions
   - Reclaimed by concurrent cycle

### Key tunings
```
-XX:MaxGCPauseMillis=200             # target pause
-XX:G1HeapRegionSize=8M               # region size (auto by default)
-XX:InitiatingHeapOccupancyPercent=45 # trigger concurrent cycle
-XX:G1MixedGCCountTarget=8            # spread mixed work over N cycles
-XX:G1ReservePercent=10               # keep 10% free
```

---

## Q6. ZGC — how does it achieve sub-millisecond pauses?

ZGC uses **colored pointers** and **load barriers** to do nearly everything concurrently.

### Colored pointers
On 64-bit, pointers have unused bits. ZGC uses bits to encode object state (Marked0, Marked1, Remapped, etc.). A load barrier (extra instruction on each reference load) handles the bit pattern.

### Concurrent phases
1. Pause Mark Start (sub-ms)
2. Concurrent mark
3. Pause Mark End (sub-ms)
4. Concurrent relocation preparation
5. Pause Relocate Start (sub-ms)
6. Concurrent relocation — objects moved while mutators read/write them via load barriers
7. Concurrent remap (folded into next mark)

### Cost
- Slightly higher CPU overhead (load barriers, extra threads)
- Larger memory footprint (~10% extra)
- Worth it for latency SLAs

### Generational ZGC (Java 21+)
Adds young/old separation to ZGC — significantly better throughput. Now the recommended ZGC mode.

---

## Q7. Stop-the-world (STW) vs concurrent phases.

**STW**: all application threads paused, only GC threads run. Pauses range from ms (G1 young) to seconds (full GC on large heap).

**Concurrent**: GC threads run alongside application threads. App may slow slightly (load barriers, write barriers) but doesn't pause.

### Why ANY GC has STW phases
- Need a consistent root snapshot (registers, thread stacks)
- Pointer-updating must be atomic from the mutator's perspective (or behind a barrier)

### What you can't escape
Even ZGC has microsecond-scale STW phases for mark start/end. Sub-millisecond, but not zero.

---

## Q8. OutOfMemoryError — what are the variants?

| OOM Type | Cause |
|---|---|
| `Java heap space` | Heap is full; live set exceeds `-Xmx` |
| `GC overhead limit exceeded` | Spending >98% time GCing, recovering <2% — give up |
| `Metaspace` | Too many loaded classes (classloader leak common cause) |
| `Direct buffer memory` | Off-heap NIO buffers exhausted |
| `unable to create new native thread` | OS thread limit (`ulimit -u`) or memory for stacks |
| `Compressed class space` | Subset of metaspace for compressed class pointers |
| `Requested array size exceeds VM limit` | Array > 2^31 - 8 |

### Heap OOM debugging workflow
1. Add `-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/path/to/dump.hprof`
2. Reproduce; collect the dump
3. Open in Eclipse MAT (Memory Analyzer Tool) or VisualVM
4. Use **Leak Suspects** report — usually points at the dominating tree
5. Look for unbounded growth — caches without eviction, listeners not unregistered, ThreadLocal not removed in pools, ClassLoader retained

---

## Q9. ClassLoader hierarchy and delegation.

```
Bootstrap (rt.jar, native; loads java.*)
    ^
    |
Platform (Java 9+; was Extension) — loads JDK modules
    ^
    |
Application (System; loads classpath / modulepath)
    ^
    |
Custom loaders (web container per-app, OSGi, etc.)
```

### Parent-first delegation
A classloader asks its parent before loading itself. This:
- Prevents `java.lang.String` from being reloaded by user code
- Causes "child-first" web containers (Tomcat) to load WAR libs before container libs

### Why one class can exist twice in a JVM
Two different classloaders can each load `com.example.Foo`. The **runtime class identity** is `(name, loader)` pair. Instances from one Foo are NOT assignable to the other Foo. This causes confusing `ClassCastException`.

### When to write a custom ClassLoader
- Hot-swapping classes at runtime (frameworks, IDE plugins)
- Plugin systems (each plugin in isolation)
- Web containers (war isolation)
- Mostly: don't. It's hard to get right.

### Classloader leak — Metaspace OOM
- App redeployed in container; old classloader can't be GC'd because something (static field, ThreadLocal, JDBC driver) holds a strong reference
- All classes that loader loaded stay loaded
- Metaspace grows without bound

---

## Q10. JIT compilation tiers.

JVM starts in interpreter mode, then JIT-compiles hot methods.

### Tiers (HotSpot)
- **Tier 0**: interpreter
- **Tier 1-3**: C1 (client compiler) — fast compile, less optimization
- **Tier 4**: C2 (server compiler) — slow compile, aggressive optimization

### Method gets hot
- Counts invocations + back-edges (loop iterations)
- Hits a threshold → queued for C1
- Profiles execution → queued for C2 with profile-guided optimization

### Inlining
- Small methods get inlined into callers
- Then more optimizations open up (escape analysis, dead code elim, etc.)
- This is why "small private methods" is rarely a perf concern — JIT inlines them

### Deoptimization
If C2 made assumptions (e.g., "this branch never taken") that turn out wrong, it deoptimizes back to interpreter and re-profiles. Hot-then-deopt cycle is a perf red flag.

###  Tuning
- `-XX:+PrintCompilation` — see what gets JIT'd
- `-XX:+UnlockDiagnosticVMOptions -XX:+PrintInlining` — see inlining decisions
- `-XX:+UseEpsilonGC -Xint` — disable JIT for comparison

---

## Q11. Common memory leaks in Java.

1. **Unbounded caches** — HashMap that grows forever. Use Caffeine or Guava Cache with eviction.
2. **Listener / callback leaks** — registered a listener, never unregister. Use `WeakReference` or unregister explicitly.
3. **ThreadLocal without remove()** in thread pools — value lives as long as thread.
4. **Closing failures** — connections, streams, file handles not closed. Use try-with-resources.
5. **Inner class outer reference** — `new Runnable() {...}` non-static inside an Activity holds the Activity. Make it `static` or use a top-level class.
6. **ClassLoader leaks** — JDBC driver registered statically, ThreadLocal in shared pool, JMX, etc. — prevents WAR redeploy cleanup.
7. **String.intern()** abuse — pre-Java 7, leaks into PermGen.
8. **`ByteBuffer.allocateDirect`** never freed — direct memory only freed when the wrapping ByteBuffer is GC'd; can lag.

---

## Q12. -Xms vs -Xmx vs -XX:NewSize.

- `-Xms` — initial heap size (start)
- `-Xmx` — max heap size (ceiling)
- `-Xmn` — young generation size (fixed)
- `-XX:NewRatio=2` — Old:Young ratio (default 2 means Young = 1/3 of heap)
- `-XX:SurvivorRatio=8` — Eden:Survivor ratio
- `-Xss` — per-thread stack size

###  Prod tip
Set `-Xms == -Xmx`. Pre-allocating avoids:
- Heap resize pauses
- Surprising OOM from another process grabbing memory
- Memory fragmentation in the OS

### Container-aware
JVM 8u131+ recognizes container memory limits with `-XX:+UseCGroupMemoryLimitForHeap` (8u131-191), or just respects cgroup limits automatically in 10+. Or set explicitly with `-XX:MaxRAMPercentage=75.0` (uses 75% of container memory for heap).

---

## Q13. What is a soft / weak / phantom reference?

```java
Object obj = new Object();          // strong reference — never GC'd while reachable

SoftReference<Object> soft = new SoftReference<>(obj);
WeakReference<Object> weak = new WeakReference<>(obj);
PhantomReference<Object> phantom = new PhantomReference<>(obj, queue);
```

| Reference | GC behavior |
|---|---|
| Strong | Never reclaimed while reachable. Standard refs. |
| Soft | Reclaimed when memory is tight. Good for **caches**. |
| Weak | Reclaimed at next GC. Good for **canonical maps**, weak-keyed maps. |
| Phantom | Object is collected normally; phantom enqueued for post-mortem cleanup. Replaces `finalize()`. |

### WeakHashMap
Entries with no other strong reference to the key are auto-removed. Useful for canonical caches keyed by ephemeral objects.

### ReferenceQueue
You can register a queue; the GC enqueues your reference object when the referent becomes weakly/softly reachable. Lets you do cleanup without polling.

---

## Q14. What's in a thread stack?

Per-thread:
- **Frames** — one per method call
- Each frame has:
  - Local variables (primitives + object references — NOT objects themselves)
  - Operand stack (for bytecode operations)
  - Reference to current class's constant pool

### Stack vs heap
| | Stack | Heap |
|---|---|---|
| Allocated | At method entry | At `new` |
| Released | At method return | At GC |
| Thread-shared? | No (per-thread) | Yes |
| Size | Small (~MB) | Large (GB) |
| Allocation cost | ~0 (pointer bump) | More (TLAB, then GC pressure) |

### StackOverflowError
Stack exhausted. Common from:
- Infinite recursion
- Very deep recursion on small `-Xss`
- Mutual recursion through frameworks

Fix: rewrite as iteration, or increase `-Xss` (carefully — affects every thread).

---

## Q15. Direct memory and ByteBuffer.

```java
ByteBuffer heap = ByteBuffer.allocate(1024);            // on-heap
ByteBuffer direct = ByteBuffer.allocateDirect(1024);    // off-heap
```

### Direct buffers
- Native memory, NOT counted in `-Xmx`
- Used by NIO, Netty, GPU/native libs
- Faster I/O — no copy needed for OS read/write
- Slower allocation, faster repeated I/O
- **Reclaimed when ByteBuffer wrapper is GC'd** (which can lag)
- Limit with `-XX:MaxDirectMemorySize=512m`

###  Direct memory OOM
Symptom: `OutOfMemoryError: Direct buffer memory`. Cause: not freeing wrapping ByteBuffer fast enough. Netty has its own pooling (`PooledByteBufAllocator`); for raw NIO, force `System.gc()` rarely (or better: pool your buffers).

---

## Q16. Important JVM flags for production.

```bash
# Heap
-Xms4g -Xmx4g
-XX:MaxRAMPercentage=75.0       # for containers; auto-sizes based on cgroup

# GC
-XX:+UseG1GC                    # or ZGC for low-latency
-XX:MaxGCPauseMillis=200
-XX:InitiatingHeapOccupancyPercent=35

# GC logging (Java 9+ unified)
-Xlog:gc*:file=gc.log:time,uptime,level,tags:filecount=10,filesize=10M

# OOM
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/var/log/heap-dumps/
-XX:+ExitOnOutOfMemoryError     # let orchestrator restart
# OR
-XX:+CrashOnOutOfMemoryError    # core dump

# Performance
-XX:+UseStringDeduplication     # G1 only; cuts memory for dup strings
-XX:+UseCompressedOops          # default on 64-bit < 32GB heap
-XX:+AlwaysPreTouch             # touch all heap pages at startup; predictable RSS

# Debugging
-XX:+PrintFlagsFinal            # see all flags + values
-XX:+UnlockDiagnosticVMOptions
```

###  -XshowSettings:vm
Quick sanity check at startup — shows JVM picks for heap size, GC, etc.

---

## Q17. How do I analyze a heap dump?

1. **Capture**
   ```bash
   jcmd <pid> GC.heap_dump /path/to/dump.hprof
   # or
   jmap -dump:live,format=b,file=dump.hprof <pid>
   ```

2. **Open in Eclipse MAT** (best free tool)
   - **Leak Suspects** report — automated finding of large dominators
   - **Histogram** — instances grouped by class, sorted by retained size
   - **Dominator Tree** — what's holding what alive
   - **OQL** — SQL-like queries over the heap

3. **Look for**:
   - Single object retaining huge tree
   - Unexpectedly many instances (e.g., 10M `String`)
   - Caches without size limit
   - Multiple ClassLoaders (deploy leak)

---

## Q18. How to profile a running JVM in prod.

### Async-profiler (best modern option)
- Sampling profiler, low overhead (~1-3%)
- CPU, allocations, locks, perf-events
- Generates flame graphs

```bash
async-profiler -d 30 -f profile.html <pid>
```

### Java Flight Recorder (JFR) — built-in
```bash
jcmd <pid> JFR.start duration=60s filename=record.jfr
jcmd <pid> JFR.stop
# Open in JDK Mission Control (JMC)
```
Always-on capable with `-XX:StartFlightRecording`. Almost zero overhead. Standard since Java 11.

### jstack — thread dumps
```bash
jstack <pid>          # plain text
jcmd <pid> Thread.print
```
Look for: many BLOCKED threads (lock contention), threads stuck in same I/O call, deadlocks (printed at end if present).

### jstat — GC/memory stats
```bash
jstat -gcutil <pid> 1s 10        # 10 samples, 1s apart
# S0 S1 E O M CCS YGC YGCT FGC FGCT GCT
```

### jcmd — Swiss army knife
```bash
jcmd <pid> help                  # see everything available
jcmd <pid> VM.flags               # current JVM flags
jcmd <pid> GC.class_histogram     # quick class histogram, no dump
jcmd <pid> VM.system_properties
```

---

## Q19. What is `final` good for at the JVM level?

Beyond "can't reassign":
- **JIT optimization**: `final` fields are eligible for inlining of their value (especially `static final` primitives/Strings — true constants)
- **Memory model**: final fields have a special freeze semantic — once constructor returns, they're safely visible without synchronization (the JMM 'final field guarantee')
- **`final` parameters**: enables capture in inner classes/lambdas (pre-Java 8 required explicit `final`; Java 8+ allows "effectively final")

###  static final primitive vs static final reference
```java
static final int MAX = 100;                       // compiled into callers as literal
static final List<Integer> LIST = List.of(1, 2);  // reference is final; LIST itself constant
                                                  // but if LIST were mutable, mutation visible
```

---

## Q20. What is Compressed Oops?

On 64-bit JVMs, references are normally 64 bits. For heaps < 32 GB, they can be **compressed to 32 bits** using `-XX:+UseCompressedOops` (default enabled).

How: object addresses are 8-byte aligned, so the bottom 3 bits are always 0. Store the address divided by 8 in 32 bits. JVM shifts at load time.

### Benefits
- Half the size for each reference
- Fewer cache misses, smaller working set
- Faster heap scanning

### Tradeoffs
- Limited to ~32 GB heap (some report 31.99 GB)
- Slight CPU overhead for shift on access (negligible)

###  Don't size your heap to 33 GB
You lose compressed oops and waste ~1.5x more memory than necessary on references. Either stay under 32 GB or go significantly above (50+ GB) to get net benefit.

---

## Q21. What metrics should you monitor for JVM health?

### Memory
- Heap used / committed / max
- Old Gen utilization (% over time — growing = leak)
- Metaspace used
- Direct memory used

### GC
- GC count (young vs full)
- GC time per minute (target: <5%)
- Pause time distribution (p99, max)
- Allocation rate (bytes/sec — high = pressure)

### Threads
- Thread count (creep = leak)
- Daemon vs user
- Blocked/waiting counts
- Per-pool active threads + queue size

### Compilation
- Code cache utilization
- JIT compilation activity (sudden spikes = something deopted)

### Latency / throughput
- App-level: p50/p99/p999 request latency
- Errors by type

### Tooling
- Prometheus + Micrometer (Spring Boot Actuator exposes it)
- JFR for deep dives
- APM (Datadog, New Relic, Splunk)
