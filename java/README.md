# Java — Senior Engineer Interview & Reference Notes

Deep-dive Q&A covering everything a senior Java engineer (8+ YoE) should be able to answer cold in an interview, design review, or war room.

##  Reading Order

| # | File | Focus | Why it matters |
|---|------|-------|----------------|
| 1 | [`01-core-java.md`](./01-core-java.md) | OOP, equals/hashCode, immutability, generics, exceptions | Foundation. Get this wrong and everything else falls apart. |
| 2 | [`02-collections.md`](./02-collections.md) | List/Set/Map families, HashMap internals, fail-fast iterators | The most asked topic. HashMap internals = table stakes. |
| 3 | [`03-concurrency.md`](./03-concurrency.md) | synchronized, volatile, JMM, locks, executors, CompletableFuture, virtual threads | This separates seniors from mid-level. Memorize the JMM rules. |
| 4 | [`04-jvm-memory.md`](./04-jvm-memory.md) | Heap/stack, GC algorithms (G1/ZGC), classloading, JIT, OOM debugging | Required for any prod-grade work. Expect deep questions on G1 + ZGC. |
| 5 | [`05-streams-functional.md`](./05-streams-functional.md) | Lambdas, Stream API internals, collectors, parallel streams, Optional | Java 8 features are now 11 years old — interviewers expect mastery. |
| 6 | [`06-modern-java.md`](./06-modern-java.md) | Records, sealed classes, pattern matching, text blocks, virtual threads (Java 9-21) | What's new and why each feature exists. |
| 7 | [`07-comparator-cheatsheet.md`](./07-comparator-cheatsheet.md) | Comparator deep-dive: PriorityQueue, Collections, int[], int[][], multi-criteria | The one note that ends all Comparator confusion. Quick reference. |

##  How to use these notes

1. **First pass**: skim each file to understand structure
2. **Second pass**: try to answer each question *out loud* before reading the answer
3. **Third pass**: write the code snippets yourself, don't just read them
4. **Before interviews**: re-read the "Common Pitfalls" callouts — those are interviewer favorites

##  Conventions used

- **Bold** = key term you should be able to define
- ` ` = "this is the trap interviewers want you to walk into"
- ` ` = pro tip / modern idiom
- ` ` = anti-pattern / what NOT to do
- Code blocks are runnable on Java 21 unless noted

##  Out of scope (intentionally)

- Spring/Spring Boot (separate file later if needed)
- JPA/Hibernate (separate file later)
- Build tools (Maven/Gradle)
- Microservices patterns
- DSA algorithms (covered in parent repo's other folders)

This folder is **pure language + runtime + standard library**.

---

*Last updated: 2026-06-22*
