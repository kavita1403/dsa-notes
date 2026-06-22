# 03 — Concurrency & Multithreading

The topic that separates seniors from mid-level. JMM rules, locks, executors, CompletableFuture, virtual threads.

---

## Q1. Thread vs Runnable vs Callable.

| | `Thread` | `Runnable` | `Callable<V>` |
|---|---|---|---|
| Run method | `run()` | `run()` | `call()` |
| Return value | No | No | Yes |
| Checked exceptions | No | No | Yes |
| Used with | `start()` directly | `new Thread(r).start()` or `ExecutorService` | `ExecutorService.submit()` |

```java
// Runnable — fire and forget
Runnable r = () -> System.out.println("hi");
executor.execute(r);

// Callable — has a return value, can throw
Callable<Integer> c = () -> { Thread.sleep(100); return 42; };
Future<Integer> f = executor.submit(c);
Integer result = f.get();  // blocks until done; throws ExecutionException on failure
```

###  Always prefer ExecutorService over `new Thread()`
- Thread creation is expensive (~MB stack, ~ms to create)
- Hard to limit concurrency
- No backpressure
- Hard to wait for completion

###  But wait — virtual threads (Java 21+) change this
With virtual threads, `Thread.ofVirtual().start(runnable)` is essentially free. We'll cover this in Q22.

---

## Q2. Thread lifecycle states.

```
NEW --[start()]--> RUNNABLE <--+
                      |        |
                [yield()/scheduling]
                      |
                      v
              +-------+--------+--------+
              |                |        |
        [wait/sleep]    [I/O block]  [synchronized contention]
              |                |        |
              v                v        v
           WAITING       BLOCKED    BLOCKED
           TIMED_WAITING
              |                         |
              +------[notify/timeout]---+
                              |
                              v
                          TERMINATED  (after run() returns)
```

States from `Thread.State` enum:
1. `NEW` — created but not started
2. `RUNNABLE` — eligible to run (may or may not be actually running)
3. `BLOCKED` — waiting to acquire a monitor lock (`synchronized`)
4. `WAITING` — indefinitely waiting (e.g., `wait()`, `join()`)
5. `TIMED_WAITING` — waiting with timeout (`sleep`, `wait(ms)`, `join(ms)`)
6. `TERMINATED` — `run()` returned

###  Trap: `BLOCKED` vs `WAITING`
- `BLOCKED` = waiting for a `synchronized` monitor specifically
- `WAITING` = waiting on a condition (`Object.wait`, `LockSupport.park`, `Lock.lock()` for j.u.c locks)
This matters for thread-dump analysis.

---

## Q3. What is `synchronized`? What does it actually guarantee?

`synchronized` provides **two** guarantees:
1. **Mutual exclusion** — only one thread holds the monitor at a time
2. **Memory visibility** — happens-before relationship; changes by Thread A before releasing the lock are visible to Thread B after acquiring the same lock

```java
public synchronized void increment() { count++; }
// equivalent to:
public void increment() { synchronized (this) { count++; } }

// static synchronized locks the Class object
public static synchronized void doStatic() { ... }
// equivalent to:
public static void doStatic() { synchronized (MyClass.class) { ... } }
```

### Critical: locks are reentrant
A thread can re-acquire a lock it already holds (otherwise calling `synchronized` method from another `synchronized` method would deadlock with itself).

###  Common pitfalls

```java
// 1. Locking on String literal — VERY BAD (string pool dedup!)
synchronized ("lock") { ... }  // another class may use the same literal

// 2. Locking on Integer/Boolean — same problem (cache)
synchronized (Integer.valueOf(1)) { ... }

// 3. Locking on `this` for public API — anyone can also lock on your instance
public synchronized void m() { ... }
// vs.
private final Object lock = new Object();
public void m() { synchronized (lock) { ... } }
```

###  Best practice: private final lock object
Keep your lock private and final. Makes intent clear, prevents external interference.

---

## Q4. What does `volatile` actually guarantee?

`volatile` provides:
1. **Visibility** — reads/writes go straight to main memory, not thread-local caches
2. **Ordering** — establishes happens-before; no reordering across volatile access

It does **NOT** provide atomicity for compound operations:
```java
volatile int counter = 0;
counter++;   // STILL RACE-y — read-modify-write is 3 ops
```

###  When volatile IS enough
- Single-writer flag: `volatile boolean stopped = false;` — one thread sets, others read
- Singleton double-checked locking:
  ```java
  private static volatile Singleton instance;
  public static Singleton get() {
      Singleton s = instance;
      if (s == null) {
          synchronized (Singleton.class) {
              s = instance;
              if (s == null) instance = s = new Singleton();
          }
      }
      return s;
  }
  ```
  Without `volatile`, another thread could see a partially-constructed object.

###  When volatile is NOT enough
- Counter incrementing → use `AtomicInteger`
- Compound state → use a lock or `AtomicReference`

---

## Q5. Explain the Java Memory Model (JMM).

The JMM specifies which writes by one thread are visible to reads by another thread. Without it, the JVM/CPU/compiler could reorder operations in surprising ways.

### Happens-before relationships

If action A *happens-before* action B, A's effects are visible to B. These create happens-before:

1. **Program order** within a single thread
2. **Monitor unlock** happens-before subsequent **lock** on the same monitor
3. **Volatile write** happens-before subsequent **volatile read** of the same variable
4. **Thread.start()** happens-before any action in the started thread
5. **Thread.join()** — actions in the joined thread happen-before join() returns
6. **Constructor completion** happens-before `finalize()` (and instance available via safe publication)
7. **Final field freeze** — `final` field writes in constructor happen-before any thread sees the object (provided no `this`-leak)
8. **Transitivity** — if A → B and B → C, then A → C

### The classic surprise
```java
// shared
int x = 0, y = 0;
boolean a = false, b = false;

// Thread 1               // Thread 2
x = 1;                    y = 1;
a = true;                 b = true;
if (b) print(y);          if (a) print(x);
```

Without proper synchronization, **both threads could print 0** — the JVM is allowed to reorder writes within a thread because there's no data dependency. Mark `a`/`b` as `volatile` to fix.

---

## Q6. wait() / notify() / notifyAll() — how do they work?

These are `Object` methods (legacy thread coordination). Three rules:

1. **Must hold the monitor**: must be called inside `synchronized` on the same object
2. **wait() releases the lock** while waiting; reacquires before returning
3. **Spurious wakeups happen** — always wait in a loop checking the condition

```java
// Producer-consumer with wait/notify
public class Buffer {
    private final List<Integer> queue = new ArrayList<>();
    private final int capacity = 10;

    public synchronized void put(int item) throws InterruptedException {
        while (queue.size() == capacity) {     // loop, not if!
            wait();                             // releases the lock
        }
        queue.add(item);
        notifyAll();                            // wake all consumers
    }

    public synchronized int take() throws InterruptedException {
        while (queue.isEmpty()) wait();
        int item = queue.remove(0);
        notifyAll();
        return item;
    }
}
```

###  notify() vs notifyAll()
- `notify()` wakes ONE arbitrary waiter — risk of missed signal if waiters wait on different conditions
- `notifyAll()` wakes ALL — safer default

###  Almost always use higher-level alternatives
- `java.util.concurrent.locks.Lock` + `Condition` for fine-grained control
- `BlockingQueue` for producer-consumer
- `CountDownLatch` / `CyclicBarrier` / `Semaphore` for coordination

---

## Q7. Lock, ReentrantLock, ReadWriteLock, StampedLock.

`java.util.concurrent.locks` — more flexible than `synchronized`.

### ReentrantLock
```java
private final ReentrantLock lock = new ReentrantLock();   // default unfair
private final ReentrantLock fair = new ReentrantLock(true);  // FIFO order

public void m() {
    lock.lock();
    try {
        // critical section
    } finally {
        lock.unlock();   // must use try-finally; no auto-release
    }
}
```

**Advantages over synchronized**:
- `tryLock()` / `tryLock(timeout)` — non-blocking acquisition
- `lockInterruptibly()` — abortable
- Fairness option (FIFO)
- Multiple `Condition`s per lock (vs single wait set with synchronized)
- Lock acquisition from one thread, release from another (rare use)

**Cost**: must manually unlock in `finally`. Easy to leak.

### ReadWriteLock
```java
ReentrantReadWriteLock rw = new ReentrantReadWriteLock();
Lock readLock = rw.readLock();
Lock writeLock = rw.writeLock();

readLock.lock();
try { return cache.get(key); }
finally { readLock.unlock(); }

writeLock.lock();
try { cache.put(k, v); }
finally { writeLock.unlock(); }
```

Multiple readers can hold the read lock simultaneously, but writers are exclusive. Good for read-heavy caches.

### StampedLock (Java 8+)
```java
StampedLock sl = new StampedLock();
long stamp = sl.tryOptimisticRead();
double currentX = x, currentY = y;
if (!sl.validate(stamp)) {                  // someone wrote — fall back
    stamp = sl.readLock();
    try { currentX = x; currentY = y; }
    finally { sl.unlockRead(stamp); }
}
```

**Optimistic reading** — no lock, but verify nothing changed. Lower overhead for read-heavy workloads. **Not reentrant**, more complex API.

---

## Q8. Atomic classes — when and how to use.

`java.util.concurrent.atomic` — lock-free, CAS-based primitives.

```java
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();                       // atomic ++
counter.getAndAdd(5);                            // returns old, adds 5
counter.compareAndSet(expected, newValue);       // CAS — atomic compare-then-set

// AtomicReference for objects
AtomicReference<Config> config = new AtomicReference<>(initial);
config.updateAndGet(c -> c.withTimeout(30));     // atomic apply function

// LongAdder for high contention counters
LongAdder hits = new LongAdder();                // multiple cells reduce contention
hits.increment();
long total = hits.sum();                          // approximate, sums all cells
```

### CAS = Compare-And-Swap
Hardware-level atomic instruction (e.g., `LOCK CMPXCHG` on x86). Compare the current value to expected; if equal, swap to new value; return success/failure. Loop on failure (spinning).

###  ABA problem
CAS only checks value equality, not "has it changed and changed back". If you push 'A', someone pops 'A', pushes 'B', pushes 'A', your CAS thinks nothing happened. Solve with `AtomicStampedReference` (adds a version counter).

###  LongAdder vs AtomicLong
Under high contention, all CAS retries on AtomicLong burn CPU. `LongAdder` shards into multiple cells; threads update different cells; `sum()` aggregates. Trade exact-value-read for write throughput.

---

## Q9. ExecutorService — what, why, configurations.

```java
// Various factory methods
ExecutorService fixed = Executors.newFixedThreadPool(10);
ExecutorService cached = Executors.newCachedThreadPool();      // unbounded — DANGEROUS
ExecutorService single = Executors.newSingleThreadExecutor();
ScheduledExecutorService sched = Executors.newScheduledThreadPool(2);

// Submit work
Future<Integer> f = fixed.submit(() -> compute());
List<Future<Integer>> all = fixed.invokeAll(callables);   // wait for all

// Shutdown — ALWAYS do this
fixed.shutdown();                                  // no new tasks; wait for existing
fixed.awaitTermination(60, TimeUnit.SECONDS);
List<Runnable> dropped = fixed.shutdownNow();      // interrupt running tasks
```

### Why prefer ExecutorService
- Thread reuse (no spawn overhead per task)
- Bounded concurrency
- Backpressure via bounded queue
- Lifecycle management
- Decoupled task submission from execution policy

### Roll your own with ThreadPoolExecutor
```java
ExecutorService pool = new ThreadPoolExecutor(
    8,                                                 // core pool size
    32,                                                // max pool size
    60L, TimeUnit.SECONDS,                             // keep-alive for non-core
    new ArrayBlockingQueue<>(1000),                    // bounded queue
    new ThreadFactoryBuilder().setNameFormat("worker-%d").build(),
    new ThreadPoolExecutor.CallerRunsPolicy()          // backpressure
);
```

###  Avoid Executors.newCachedThreadPool()
Unbounded queue + unbounded threads. Under load = OOM. Always use a bounded queue with sane rejection policy.

### Rejection policies
- `AbortPolicy` (default) — throws `RejectedExecutionException`
- `CallerRunsPolicy` — runs on submitter thread (natural backpressure)
- `DiscardPolicy` — silently drops
- `DiscardOldestPolicy` — drops oldest queued task

---

## Q10. Future vs CompletableFuture.

### Future — limited
```java
Future<Integer> f = executor.submit(() -> 42);
Integer v = f.get();              // blocking
Integer v = f.get(5, SECONDS);    // blocking with timeout
f.cancel(true);                   // cancel + interrupt
```
- Cannot compose
- Only blocking retrieval
- No exception handling without try/catch around get

### CompletableFuture (Java 8+) — composable async
```java
CompletableFuture<Integer> cf = CompletableFuture
    .supplyAsync(() -> fetchUser(id), executor)        // async
    .thenApply(user -> user.getAge())                  // transform (sync)
    .thenApplyAsync(this::expensiveTransform, ex2)     // transform (async on ex2)
    .thenCompose(age -> ageToCategoryAsync(age))       // flatMap-style
    .exceptionally(ex -> 0)                            // recover
    .handle((value, ex) -> ex != null ? -1 : value);   // recover OR transform

// Combine
CompletableFuture<String> combined = a.thenCombine(b, (av, bv) -> av + bv);

// Wait for any/all
CompletableFuture.allOf(f1, f2, f3).join();
CompletableFuture<Object> first = CompletableFuture.anyOf(f1, f2);
```

###  Critical: provide your own Executor
```java
CompletableFuture.supplyAsync(task);          // uses ForkJoinPool.commonPool() — dangerous
CompletableFuture.supplyAsync(task, myExec);  // explicit pool — preferred
```
`commonPool()` is shared JVM-wide. Blocking on it can starve every parallel stream and CompletableFuture in your app.

###  Java 19+ — Structured Concurrency (preview)
```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var userFut  = scope.fork(() -> fetchUser(id));
    var orderFut = scope.fork(() -> fetchOrders(id));
    scope.join().throwIfFailed();
    return new Response(userFut.resultNow(), orderFut.resultNow());
}
```
Treats parallel tasks as a unit of work — cancellation, exception handling become trivial.

---

## Q11. ForkJoinPool and work-stealing.

`ForkJoinPool` powers parallel streams and `CompletableFuture` defaults. Each worker has its own deque. When idle, it **steals** tasks from the back of other workers' deques.

```java
class SumTask extends RecursiveTask<Long> {
    private final long[] arr;
    private final int start, end;
    private static final int THRESHOLD = 1000;

    SumTask(long[] arr, int start, int end) { ... }

    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            long sum = 0;
            for (int i = start; i < end; i++) sum += arr[i];
            return sum;
        }
        int mid = (start + end) >>> 1;
        SumTask left = new SumTask(arr, start, mid);
        SumTask right = new SumTask(arr, mid, end);
        left.fork();
        long r = right.compute();    // run right inline, don't fork both
        long l = left.join();
        return l + r;
    }
}

ForkJoinPool pool = new ForkJoinPool();
long total = pool.invoke(new SumTask(arr, 0, arr.length));
```

###  Don't block in ForkJoinPool tasks
Tasks should be pure CPU work. Blocking (I/O, sleep, locks) starves the pool. If you must, wrap in `ForkJoinPool.ManagedBlocker`.

---

## Q12. ThreadLocal — what, when, and the leak risk.

A `ThreadLocal<T>` stores a per-thread value, isolated from other threads.

```java
private static final ThreadLocal<DateFormat> df =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

public String format(Date d) {
    return df.get().format(d);     // each thread has its own SimpleDateFormat
}
```

### Common uses
- Caching non-thread-safe objects (SimpleDateFormat, Random — though `ThreadLocalRandom` exists)
- Request-scoped data (tracing/MDC, user context)
- Database transaction binding (Spring uses ThreadLocal internally)

###  Memory leak danger
ThreadLocal entries live as long as the Thread. In a thread pool, threads live forever. So if you put a big object into ThreadLocal and never `remove()`, it stays in memory until the pool dies.

```java
try {
    ctxHolder.set(new RequestContext());
    handle(request);
} finally {
    ctxHolder.remove();           // critical in pool environments!
}
```

###  InheritableThreadLocal
Child threads inherit a copy. Mostly broken with thread pools (only inherited at thread creation, not per-task).

###  Modern alternative — ScopedValue (Java 21+ preview)
```java
static final ScopedValue<RequestContext> CONTEXT = ScopedValue.newInstance();

ScopedValue.where(CONTEXT, ctx).run(() -> handle(request));
// CONTEXT.get() works inside; auto-cleared after run() returns
```
Immutable, structured, works well with virtual threads.

---

## Q13. Deadlock — what, how to detect, how to prevent.

**Deadlock** = two+ threads each holding a resource the other needs, neither can proceed.

```java
// Classic example
Thread t1 = new Thread(() -> {
    synchronized (lockA) {
        Thread.sleep(100);
        synchronized (lockB) { ... }       // wants B, holds A
    }
});
Thread t2 = new Thread(() -> {
    synchronized (lockB) {
        Thread.sleep(100);
        synchronized (lockA) { ... }       // wants A, holds B
    }
});
```

### Four conditions (Coffman conditions)
All four must hold:
1. Mutual exclusion (resource non-shareable)
2. Hold and wait (hold one, wait for another)
3. No preemption (resources released voluntarily)
4. Circular wait (cycle of waiters)

### Prevention
- **Lock ordering**: always acquire locks in a consistent global order (e.g., by `System.identityHashCode`)
- **tryLock with timeout**: `lock.tryLock(1, SECONDS)` — back off on failure
- **Lock-free algorithms** with CAS
- **Minimize lock scope** — hold for as little code as possible
- **Don't call foreign code while holding locks** — they might call back into your locks

### Detection
- `jstack <pid>` — explicitly identifies deadlocks at the bottom of the dump
- `ThreadMXBean.findDeadlockedThreads()` — programmatic
- JMX / Mission Control / VisualVM

---

## Q14. Livelock and starvation.

**Livelock**: threads keep retrying / yielding to each other but make no progress. Like two people stepping aside in a hallway in sync.

**Starvation**: thread can't get CPU/lock because higher-priority threads always win. Common with unfair locks under heavy contention.

###  Fix
- Add randomized backoff (`Thread.sleep(rand.nextInt(100))`)
- Use fair locks (`new ReentrantLock(true)`) when fairness matters more than throughput

---

## Q15. Synchronizers — CountDownLatch, CyclicBarrier, Semaphore, Phaser.

### CountDownLatch — one-shot countdown
```java
CountDownLatch latch = new CountDownLatch(3);
for (int i = 0; i < 3; i++) {
    new Thread(() -> { doWork(); latch.countDown(); }).start();
}
latch.await();                  // blocks until count hits 0
// can't be reset; use CyclicBarrier for reusable
```
Use for: "wait for N initialization steps", "start gate" pattern, integration tests.

### CyclicBarrier — N threads meet, then go
```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> System.out.println("All arrived"));
// each thread calls
barrier.await();                // blocks until 3 have called; runs the action; resets
```

### Semaphore — permits / connection pool
```java
Semaphore sem = new Semaphore(10);    // max 10 concurrent
sem.acquire();
try { useDbConnection(); }
finally { sem.release(); }
```

### Phaser — flexible multi-phase coordination
Like CyclicBarrier but dynamic party count, multi-phase. More complex but more powerful.

---

## Q16. ConcurrentHashMap atomic operations.

Don't `if (!map.containsKey) map.put(...)`. Use atomic compound ops:

```java
map.putIfAbsent(k, v);                   // returns existing, doesn't replace
map.computeIfAbsent(k, key -> create(key));  // atomic load-or-create
map.computeIfPresent(k, (key, val) -> val + 1);
map.compute(k, (key, val) -> val == null ? 1 : val + 1);
map.merge(k, 1, Integer::sum);           // increment counter atomically
```

```java
// Word-count with merge
words.forEach(w -> counts.merge(w, 1, Integer::sum));
```

###  computeIfAbsent gotcha
The lambda should be fast and non-recursive. Inside the lambda, the bucket is locked. Reentrant calls to the same bucket (or recursive computeIfAbsent on the same map) can deadlock.

---

## Q17. Parallel streams — when do they actually help?

```java
list.parallelStream().filter(...).map(...).reduce(...);
```

Parallel streams use the `ForkJoinPool.commonPool()` to split and process the source.

### Heuristics for when parallel helps
| Factor | Want |
|---|---|
| Data size | Large (millions of elements) |
| Per-element work | Expensive |
| Source | ArrayList, array, IntStream.range (splits well) |
| Operations | Stateless, associative |
| Reduce identity | Easy/cheap to combine |

### When parallel HURTS
- Small data (overhead > work)
- I/O or blocking inside lambdas (starves the pool)
- LinkedList or `Stream.iterate` source (can't split)
- Boxing-heavy (use `IntStream`/`LongStream`)
- Order-sensitive operations

###  Always benchmark
JMH benchmarks beat intuition. Parallel streams often look attractive but lose to sequential because of the splitting overhead.

---

## Q18. Producer-consumer with BlockingQueue.

The canonical pattern:

```java
BlockingQueue<Task> queue = new LinkedBlockingQueue<>(1000);
ExecutorService producers = Executors.newFixedThreadPool(4);
ExecutorService consumers = Executors.newFixedThreadPool(8);

// Producers
for (int i = 0; i < 4; i++) {
    producers.submit(() -> {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                Task t = produce();
                queue.put(t);                // blocks if queue full -> backpressure
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return;
            }
        }
    });
}

// Consumers
for (int i = 0; i < 8; i++) {
    consumers.submit(() -> {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                Task t = queue.take();        // blocks if queue empty
                process(t);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return;
            }
        }
    });
}
```

###  Choose your queue
- `ArrayBlockingQueue(cap)` — bounded, FIFO, locks (single lock for put+take)
- `LinkedBlockingQueue(cap?)` — optionally bounded, 2 locks (better throughput)
- `SynchronousQueue` — zero capacity, hand-off; producer blocks until consumer ready
- `PriorityBlockingQueue` — unbounded, priority order
- `DelayQueue` — elements with delays, only released after delay

---

## Q19. Why we should NOT call Thread.stop(), suspend(), resume().

All deprecated since Java 1.2 / removed in 21. Why:
- `stop()`: throws ThreadDeath at an arbitrary point — locks left held, half-mutated state
- `suspend()`: doesn't release locks — easy deadlock
- `resume()`: paired with suspend, same problem

### Correct cooperative cancellation
```java
public class Worker implements Runnable {
    private volatile boolean stop = false;

    public void stop() { stop = true; }

    @Override
    public void run() {
        while (!stop && !Thread.currentThread().isInterrupted()) {
            try {
                Task t = queue.take();
                process(t);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();    // restore flag
                break;                                  // exit gracefully
            }
        }
    }
}
```

###  InterruptedException — never swallow
If you catch it and don't re-throw or re-interrupt, the cancellation signal is lost. Either:
1. `Thread.currentThread().interrupt()` to restore the flag
2. Re-throw as a runtime exception
3. Propagate `throws InterruptedException` up the call stack

---

## Q20. ConcurrentModificationException — how to avoid in multi-threaded contexts.

CME is fail-fast detection, NOT a thread-safety guarantee. The opposite — its absence doesn't mean your code is correct.

```java
// 1. Single-threaded mutation during iteration
list.removeIf(x -> x.isExpired());      // use the proper API

// 2. Multi-threaded — use a concurrent collection
ConcurrentHashMap<K, V> map = new ConcurrentHashMap<>();
// iterators are weakly consistent — never throw CME, may or may not see concurrent updates

// 3. Snapshot
List<X> snapshot = new ArrayList<>(sharedList);
for (X x : snapshot) { ... }            // can't see concurrent updates, but safe
```

---

## Q21. Memory consistency in double-checked locking — why was it broken pre-Java 5?

Pre-Java 5, this seemingly-correct code was broken:
```java
class Singleton {
    private static Singleton instance;
    public static Singleton get() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) instance = new Singleton();
            }
        }
        return instance;
    }
}
```

Reason: `instance = new Singleton()` is roughly:
1. Allocate memory
2. Run constructor
3. Assign reference

The JVM was free to reorder 2 and 3. Another thread could see a non-null `instance` whose constructor hadn't run → NPE / corrupted state.

**Fix (Java 5+)**: `private static volatile Singleton instance;` — volatile prevents the reordering and ensures visibility.

###  Cleaner alternative — initialization-on-demand holder
```java
class Singleton {
    private Singleton() {}
    private static class Holder { static final Singleton INSTANCE = new Singleton(); }
    public static Singleton get() { return Holder.INSTANCE; }
}
```
Class loading is already thread-safe by JLS. No volatile needed.

---

## Q22. Virtual threads (Java 21+) — what changes?

A virtual thread is a lightweight thread managed by the JVM, not the OS. Millions per process possible.

```java
// Java 21+
Thread.startVirtualThread(() -> handleRequest(req));

// In ExecutorService form
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 1_000_000; i++) {
        executor.submit(() -> handleTask());
    }
}
```

### Why virtual threads matter
- Carrier threads (OS threads) park while a virtual thread is blocked on I/O
- Other virtual threads can mount on the freed carrier
- **Blocking I/O becomes free** — no need for reactive style for scalability
- Same `Thread` API, same `synchronized`, same try/catch/throw — minimal code changes

###  Pinning — when virtual threads can't unmount
- Inside `synchronized` (Java 21; relaxed in Java 24+)
- Native code / JNI
- File system operations (some)

Fix: use `ReentrantLock` instead of `synchronized` for code that blocks on I/O.

###  Don't pool virtual threads
Pooling is for expensive resources. Virtual threads are cheap — `Executors.newVirtualThreadPerTaskExecutor()` creates one per task, on purpose.

###  Don't use ThreadLocal heavily with virtual threads
Millions of virtual threads, each with thread-local storage = OOM. Use `ScopedValue` (preview) instead.

---

## Q23. False sharing — the cache-line trap.

CPU caches operate on **cache lines** (typically 64 bytes). If two threads update different variables that share a cache line, the CPU treats them as conflict, invalidating the line on every write — destroying performance.

```java
class Bad {
    long a;   // thread A writes
    long b;   // thread B writes
    // both fit in one 64-byte cache line — false sharing
}

// Fix — @Contended (Java 8+)
class Good {
    @jdk.internal.vm.annotation.Contended long a;
    @jdk.internal.vm.annotation.Contended long b;
}
// or manually pad with 7 extra longs between fields
```

Use `-XX:-RestrictContended` to enable user-code `@Contended`. Used internally by `LongAdder`, `ConcurrentHashMap`'s counter cells, etc.

---

## Q24. ThreadPoolExecutor sizing — how do you pick the right thread count?

**For CPU-bound work**: `numThreads = numCores + 1`
**For I/O-bound work**: `numThreads = numCores * (1 + waitTime/computeTime)`

Examples:
- 8-core machine, CPU-bound: ~8-9 threads
- 8-core machine, I/O = 90% wait: 8 * (1 + 9) = 80 threads
- 8-core machine, mixed at 50/50: 8 * (1 + 1) = 16 threads

### Reality
- Measure first (`-Xlog:gc`, JFR, async-profiler)
- Start with a guess, tune with metrics (CPU utilization, queue depth, latency)
- Use bounded queues with reasonable capacity
- Different pools for different workloads (DB, HTTP, internal work) — avoids cross-contamination

---

## Q25. The most common concurrency bugs in real systems.

1. **Missing synchronization** — shared mutable state without locks or volatile
2. **Lock-and-call-foreign-code** — calling user code (callbacks, listeners) while holding a lock → deadlock
3. **Compound check-then-act** — `if (!map.containsKey) map.put` — race window
4. **Iterating without protection** — CME or worse silent corruption
5. **Catching InterruptedException and swallowing it**
6. **Storing context in ThreadLocal but not removing in `finally`**
7. **Spawning unbounded threads** under load → OOM
8. **Blocking inside ForkJoinPool tasks** — starves parallel streams app-wide
9. **Using `synchronized` on `this` for a public class** — external code can lock you
10. **Not handling `RejectedExecutionException`** when pool is full/shut down
