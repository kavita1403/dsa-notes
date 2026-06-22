# HashMap Patterns - Cheat Sheet

**Decision fork:** Count occurrences -> frequency map (`merge(k, 1, Integer::sum)`); use `int[26]` for ASCII. Find pair with target -> two-sum (check complement BEFORE put). Count subarrays with sum K -> prefix sum + map of `prefix -> count`. Longest subarray with property -> prefix state + map of `state -> firstIndex` (use `putIfAbsent`). Group by derived key -> `computeIfAbsent(key, x -> new ArrayList<>()).add(v)`. Two-way mapping -> two maps OR map + set. Top K frequent -> freq map + min-heap of size K. Detect duplicate / visited -> HashSet, use `add()`'s boolean return. Range queries / floor-ceiling -> TreeMap. Cache with eviction -> HashMap + DLL (LRU) or LinkedHashMap with `accessOrder=true`.

## Java idioms cheat
```java
map.merge(k, 1, Integer::sum);                       // frequency increment, NPE-free
map.computeIfAbsent(k, x -> new ArrayList<>()).add(v);  // get-or-create container
map.getOrDefault(k, 0);                               // safe read, no NPE
map.putIfAbsent(k, v);                                // only put if absent (preserve first occurrence)
int[] freq = new int[26];                             // 10x faster than HashMap for ASCII lowercase
```

## Frequency Counting
```java
Map<Character, Integer> freq = new HashMap<>();
for (char c : s.toCharArray()) freq.merge(c, 1, Integer::sum);

// First unique char
for (int i = 0; i < s.length(); i++)
    if (freq.get(s.charAt(i)) == 1) return i;
```
Use `int[26]` for lowercase ASCII. Don't use `put(k, freq.get(k) + 1)` - NPE if k absent.

## Two-Sum (unsorted, indices)
```java
Map<Integer, Integer> seen = new HashMap<>();
for (int i = 0; i < n; i++) {
    int need = target - arr[i];
    if (seen.containsKey(need)) return new int[]{seen.get(need), i};
    seen.put(arr[i], i);                              // CHECK FIRST, then put
}
```
Put-before-check lets element match itself when `target = 2 * arr[i]`.

## Count Pairs With Target Sum (allows duplicates)
```java
Map<Integer, Integer> count = new HashMap<>();
long pairs = 0;
for (int x : arr) {
    pairs += count.getOrDefault(target - x, 0);     // count partners seen SO FAR
    count.merge(x, 1, Integer::sum);                  // then add this one
}
```

## Prefix Sum + HashMap (count subarrays with sum K)
```java
Map<Integer, Integer> prefixCount = new HashMap<>();
prefixCount.put(0, 1);                                // CRITICAL - empty prefix sums to 0
int sum = 0, count = 0;
for (int x : arr) {
    sum += x;
    count += prefixCount.getOrDefault(sum - k, 0);
    prefixCount.merge(sum, 1, Integer::sum);
}
```
For "divisible by K": store `((sum % k) + k) % k` as state (handles negatives).

## Prefix State + First-Occurrence (LONGEST subarray)
```java
Map<Integer, Integer> firstSeen = new HashMap<>();
firstSeen.put(0, -1);
int sum = 0, best = 0;
for (int i = 0; i < n; i++) {
    sum += arr[i];
    if (firstSeen.containsKey(sum - k))
        best = Math.max(best, i - firstSeen.get(sum - k));
    firstSeen.putIfAbsent(sum, i);                    // only first occurrence -> longest span
}
```
**Count** subarrays -> `prefix -> count`. **Longest** subarray -> `prefix -> firstIndex` (via `putIfAbsent`).

## Contiguous Array (equal 0s and 1s)
```java
firstSeen.put(0, -1);
int balance = 0, best = 0;
for (int i = 0; i < n; i++) {
    balance += (arr[i] == 0 ? -1 : 1);                // treat 0 as -1
    if (firstSeen.containsKey(balance))
        best = Math.max(best, i - firstSeen.get(balance));
    else firstSeen.put(balance, i);
}
```

## Grouping (Group Anagrams)
```java
Map<String, List<String>> groups = new HashMap<>();
for (String w : words) {
    char[] arr = w.toCharArray();
    Arrays.sort(arr);
    groups.computeIfAbsent(new String(arr), k -> new ArrayList<>()).add(w);
}
return new ArrayList<>(groups.values());
```
For long words: build `int[26]` signature, encode with delimiter (`f + "#"`) to avoid collision (`1,12` vs `11,2`).

## Bidirectional (Isomorphic Strings)
```java
Map<Character, Character> s2t = new HashMap<>(), t2s = new HashMap<>();
for (int i = 0; i < s.length(); i++) {
    char a = s.charAt(i), b = t.charAt(i);
    if (s2t.containsKey(a) && s2t.get(a) != b) return false;
    if (t2s.containsKey(b) && t2s.get(b) != a) return false;
    s2t.put(a, b); t2s.put(b, a);
}
return true;
```
ONE map only -> can't catch "two source chars mapping to same target".

## Sliding Window + Frequency (At Most K Distinct)
```java
Map<Character, Integer> freq = new HashMap<>();
int l = 0, best = 0;
for (int r = 0; r < s.length(); r++) {
    freq.merge(s.charAt(r), 1, Integer::sum);
    while (freq.size() > k) {
        char left = s.charAt(l++);
        if (freq.merge(left, -1, Integer::sum) == 0) freq.remove(left);
    }
    best = Math.max(best, r - l + 1);
}
```
MUST remove key when count hits 0, else `freq.size()` is wrong.

## Top K Frequent (Min-Heap of size K)
```java
Map<Integer, Integer> freq = new HashMap<>();
for (int x : arr) freq.merge(x, 1, Integer::sum);

PriorityQueue<int[]> minHeap = new PriorityQueue<>(Comparator.comparingInt(a -> a[1]));
for (var e : freq.entrySet()) {
    minHeap.offer(new int[]{e.getKey(), e.getValue()});
    if (minHeap.size() > k) minHeap.poll();
}
```
Min-heap of size K -> O(n log k). Bucket sort (`List[]` indexed by freq) -> O(n).

## LRU Cache (use LinkedHashMap shortcut)
```java
class LRUCache extends LinkedHashMap<Integer, Integer> {
    private final int cap;
    LRUCache(int cap) { super(cap, 0.75f, true); this.cap = cap; }    // accessOrder = true
    public int get(int k) { return super.getOrDefault(k, -1); }
    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer,Integer> e) { return size() > cap; }
}
```
Full DLL + HashMap implementation needed if interviewer asks "no built-in shortcuts".

## Visited Set (Contains Duplicate / Happy Number)
```java
Set<Integer> seen = new HashSet<>();
for (int x : arr) if (!seen.add(x)) return true;     // add() returns false if already there
```

## Multi-Field Keys (BFS visited, state hashing)
```java
// Best (Java 16+): use record
record Coord(int r, int c) {}
Map<Coord, Integer> visited = new HashMap<>();
visited.put(new Coord(i, j), val);

// Fast: int encoding for small ranges
int key = i * 1000 + j;            // grids where i, j < 1000
// Or long encoding for larger:
long key = ((long) i << 32) | (j & 0xFFFFFFFFL);
```
`int[]` as key -> compares by REFERENCE, not content. Use record / encoded int / long.

## TreeMap (floor / ceiling for time-based KV)
```java
TreeMap<Integer, String> tm = new TreeMap<>();
tm.floorEntry(t);     // largest key <= t (use for time-based "get value at or before time t")
tm.ceilingEntry(t);   // smallest key >= t
tm.subMap(low, high); // [low, high)
```

## Universal traps
- **Boxing**: `Map<Integer, Integer>` boxes every int. For hot paths, prefer `int[]` (bounded key) or specialized libs.
- **NPE on auto-unbox**: `int x = map.get(k)` throws if absent. Use `getOrDefault(k, 0)`.
- **`==` on Integer**: identity; works only for -128..127 cache. Use `.equals()` or `.intValue()`.
- **HashMap iteration order isn't insertion order** (use LinkedHashMap if you need that).
- **Mutable keys**: mutate hash -> can never find again. Use immutable types (String, Integer, record).
- **HashMap NOT thread-safe** - use `ConcurrentHashMap` if needed.
- **`Arrays.asList(intArr)`** returns `List<int[]>` (size 1!). Use `Arrays.stream(intArr).boxed().toList()`.
- **Prefix-sum trap**: forgetting `map.put(0, 1)` (or `map.put(0, -1)`) - subarrays from index 0 go uncounted.
