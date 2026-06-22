# HashMap Patterns - Master Reference Note

```java
// Java HashMap idioms cheat-sheet
Map<K, V> map = new HashMap<>();
map.put(k, v);                                   // insert / overwrite
map.get(k);                                       // null if absent
map.getOrDefault(k, defaultVal);                  // safe read
map.containsKey(k);
map.putIfAbsent(k, v);                            // only put if not present; returns old (or null)
map.merge(k, 1, Integer::sum);                    // ATOMIC-style increment; counts frequency
map.computeIfAbsent(k, x -> new ArrayList<>());   // get-or-create container
map.computeIfPresent(k, (key, val) -> val * 2);   // modify only if present
map.remove(k);
map.forEach((k, v) -> ...);
map.entrySet(); map.keySet(); map.values();

// Specialized variants
LinkedHashMap<K, V>   // preserves insertion (or access) order; used for LRU
TreeMap<K, V>         // sorted by key; floor/ceiling/headMap/tailMap
HashSet<T>            // map keys only; same hashing semantics as HashMap

// ASCII / lowercase shortcut - 10x faster than HashMap
int[] freq = new int[26];
for (char c : s.toCharArray()) freq[c - 'a']++;
```

---

## Decision Fork (ask yourself this before writing any code)

> "Count occurrences" / "frequency of each X" -> **Frequency Map** with `merge(k, 1, Integer::sum)`. For ASCII lowercase use `int[26]`.
>
> "Find pair (or triplet) with target sum/difference" -> **Two-Sum Family**: HashMap of `value -> index`, lookup complement as you iterate.
>
> "Count / find subarray with sum equal to K" (allows negatives) -> **Prefix Sum + HashMap**. Store `prefix -> count` (or `prefix -> first index`).
>
> "Longest subarray with property X" (sum K, equal 0s/1s, etc.) -> **Prefix State + first-occurrence map**. Store `state -> earliest index`.
>
> "Group elements by some derived key" (anagrams, by mod, by category) -> **Grouping Map** `Map<Key, List<T>>` with `computeIfAbsent`.
>
> "Two-way mapping consistent" (isomorphic strings, word pattern) -> **Two HashMaps** (or one map + one set).
>
> "Top K frequent / most common" -> **Frequency Map + Min-Heap of size K** (or bucket sort by frequency).
>
> "Cache with eviction policy" (LRU / LFU) -> **HashMap + Doubly Linked List** (LRU) or **HashMap + Buckets** (LFU).
>
> "Detect duplicate / visited / cycle in stream" -> **HashSet** (or `Map<T, Boolean>`).
>
> "Subarray with sum divisible by K / equal 0s and 1s" -> **Prefix Sum mod K** (or treat 0 as -1) **+ HashMap**.
>
> "Multi-field key" (coords, tuples) -> **Encoded String key** (e.g., `i + "," + j`) OR `record` OR `Map<List<Integer>, V>`.

Everything else in hashmap-land is a variant or combination of these.

---

## Quick Lookup Table (read this first when revisiting after a gap)

| # | Pattern | Trigger phrases | Anchor problem | Core idea |
|---|---|---|---|---|
| 1 | Frequency Counting | "count occurrences", "most frequent", "first non-repeating", "anagram" | First Unique Character in a String | `map.merge(k, 1, Integer::sum)`; `int[26]` for ASCII |
| 2 | Two-Sum Family | "two sum / 3-sum unsorted", "find pair with target", "Hamming pair" | Two Sum | Iterate once; for each x, check if `target - x` already seen |
| 3 | Prefix Sum + HashMap (count subarrays) | "subarray sum equals K", "binary subarrays with sum", "count of subarrays divisible by K" | Subarray Sum Equals K | `prefix - K` lookup; map stores `prefix -> count` |
| 4 | Prefix State + first-occurrence (longest) | "longest subarray with sum K", "contiguous array equal 0s and 1s" | Contiguous Array | Map stores `state -> earliest index`; longest = i - earliest |
| 5 | Grouping / Bucketing | "group anagrams", "group by property", "group strings by pattern" | Group Anagrams | `computeIfAbsent(key, x -> new ArrayList<>()).add(v)` |
| 6 | Bidirectional / Two-Map | "isomorphic strings", "word pattern", "bijection check" | Isomorphic Strings | Two maps (s->t and t->s) - reject if either disagrees |
| 7 | Sliding Window + Frequency | "longest substring with K distinct", "permutation in string", "min window substring" | Longest Substring with At Most K Distinct Characters | Frequency map + expand/shrink window |
| 8 | Map + Heap (Top K) | "top K frequent", "K most common", "Kth most frequent" | Top K Frequent Elements | Build frequency map; pick top K via heap or bucket sort |
| 9 | LRU / LFU Cache | "design LRU", "design LFU", "evict least recently used" | LRU Cache | HashMap + doubly-linked list (LRU); freq -> bucket (LFU) |
| 10 | Visited / Dedup Set | "detect duplicate", "cycle in graph", "happy number" | Contains Duplicate | HashSet for O(1) membership; one pass |
| 11 | Multi-Field / Encoded Keys | "tuple-keyed map", "coord -> value", "state hashing" | Number of Islands II | Encode as `"i,j"` string, or use `record`, or `Map<List<Integer>, V>` |
| 12 | TreeMap-Specific Patterns | "floor / ceiling key", "range query", "sorted iteration" | Snapshot Array | TreeMap for sorted keys + log-N floor/ceiling lookups |
| 13 (design) | Frequency-Backed Designs | "tweet counts per frequency", "logger rate limiter", "time-based key-value" | Time Based Key-Value Store | Map<key, TreeMap<timestamp, value>> + ceilingKey/floorKey |

---

## 1. Frequency Counting

**When to use it:** "Count how many times each X appears" - characters, numbers, words, anything hashable. Foundation for many other patterns (anagrams, top K frequent, first unique).

**Trigger phrases:** "count occurrences", "frequency of each", "most/least frequent", "first non-repeating character", "valid anagram".

**Anchor problem:** First Unique Character in a String.

**Template (basic frequency map):**
```java
Map<Character, Integer> freq = new HashMap<>();
for (char c : s.toCharArray()) {
    freq.merge(c, 1, Integer::sum);          // one-liner; clean and correct
}

// First non-repeating character: walk string again
for (int i = 0; i < s.length(); i++) {
    if (freq.get(s.charAt(i)) == 1) return i;
}
return -1;
```

**Template (ASCII lowercase - use array, not HashMap):**
```java
int[] freq = new int[26];
for (char c : s.toCharArray()) freq[c - 'a']++;

// Compare two strings' char distributions (anagram)
for (char c : t.toCharArray()) freq[c - 'a']--;
for (int f : freq) if (f != 0) return false;
return true;
```
**10x faster** than HashMap for ASCII lowercase. Also zero per-element boxing.

**Template (frequency of integers in an int[]):**
```java
Map<Integer, Integer> freq = new HashMap<>();
for (int x : arr) freq.merge(x, 1, Integer::sum);

// Get the max-frequency key in one stream call
int mode = freq.entrySet().stream()
    .max(Map.Entry.comparingByValue())
    .get().getKey();
```

**Variations:**
- **Find All Numbers Appearing Twice** (one pass + frequency map).
- **Majority Element** (Boyer-Moore voting beats HashMap for O(1) space; HashMap works too).
- **Count Pairs With Equal Sum**: frequency map of sums.
- **Sort Array By Increasing Frequency**: build freq map, then sort with custom comparator.

**Common bugs:**
- Using `put(k, freq.get(k) + 1)` instead of `merge` - NPE if `k` absent. Either use `getOrDefault(k, 0) + 1` or just `merge`.
- For ASCII, using `Map` when `int[26]` would do - slow and noisy.
- After mutating freq down to 0, `containsKey` STILL returns true. Either `remove(k)` when count hits 0, or check `freq.getOrDefault(k, 0) > 0`.

---

## 2. Two-Sum Family (complement lookup)

**When to use it:** "Find a pair (or triplet, etc.) that satisfies a relationship" on an UNSORTED array, in O(n). The trick: as you iterate, you've already seen the answer's "partner" - look it up in O(1).

**Trigger phrases:** "two sum (unsorted)", "find pair with target sum", "find pair with target diff", "subarray with sum K (variant)", "count pairs (a, b) such that f(a, b) = X".

**Anchor problem:** Two Sum.

**Template (Two Sum - return indices):**
```java
Map<Integer, Integer> seen = new HashMap<>();    // value -> index
for (int i = 0; i < n; i++) {
    int complement = target - arr[i];
    if (seen.containsKey(complement)) {
        return new int[]{seen.get(complement), i};
    }
    seen.put(arr[i], i);                          // PUT AFTER CHECK
}
return new int[]{-1, -1};
```

**Template (Count Pairs With Target Sum - duplicates count):**
```java
Map<Integer, Integer> count = new HashMap<>();
long pairs = 0;
for (int x : arr) {
    pairs += count.getOrDefault(target - x, 0);  // count how many partners seen so far
    count.merge(x, 1, Integer::sum);
}
return pairs;
```
Note: count BEFORE incrementing - prevents self-pair when x == target / 2.

**Template (Two Sum III - Data Structure Design):**
```java
class TwoSum {
    Map<Integer, Integer> counts = new HashMap<>();

    void add(int v) { counts.merge(v, 1, Integer::sum); }

    boolean find(int target) {
        for (Map.Entry<Integer, Integer> e : counts.entrySet()) {
            int x = e.getKey(), need = target - x;
            if (x == need && e.getValue() >= 2) return true;   // need same element twice
            if (x != need && counts.containsKey(need)) return true;
        }
        return false;
    }
}
```

**Variations:**
- **4-Sum II** (count tuples (a,b,c,d) from 4 arrays summing to 0): split into two halves; map sums of (a+b), then for each (c+d) check `0 - (c+d)` in map. O(n^2) instead of O(n^4).
- **K-Diff Pairs in an Array**: HashMap for k=0 case, HashSet otherwise.
- **Subarray Sum Equals K** (the inverse - covered in pattern 3).
- **Pairs of Songs With Total Durations Divisible by 60**: same template, mod 60 as key.

**Common bugs:**
- Putting BEFORE checking - same element matches itself when `target = 2 * x`.
- Forgetting to handle duplicates correctly when counting pairs - `count BEFORE increment` is the safe pattern.
- For 4-Sum II, NOT splitting into halves -> O(n^4) brute force.

---

## 3. Prefix Sum + HashMap (count subarrays with sum K)

**When to use it:** Count or find subarrays whose sum equals (or is divisible by) K - even when array has NEGATIVES (where sliding window breaks). The map stores `prefix_sum -> count` so that for current prefix `P`, you look up how many earlier prefixes equal `P - K`.

**Trigger phrases:** "count subarrays with sum equal to K", "subarray sums divisible by K", "binary subarrays with sum", "count of subarrays with at most K odd numbers".

**Anchor problem:** Subarray Sum Equals K.

**Template (Subarray Sum Equals K):**
```java
Map<Integer, Integer> prefixCount = new HashMap<>();
prefixCount.put(0, 1);                         // empty prefix sums to 0; CRITICAL
int sum = 0, count = 0;
for (int x : arr) {
    sum += x;
    count += prefixCount.getOrDefault(sum - k, 0);   // partners that close subarrays = K
    prefixCount.merge(sum, 1, Integer::sum);
}
return count;
```
**Intuition:** If `prefix[i] - prefix[j] = K`, then `arr[j..i-1]` sums to K. For each i, count j's such that `prefix[j] = prefix[i] - K`.

**Template (Subarrays Divisible by K):**
```java
Map<Integer, Integer> modCount = new HashMap<>();
modCount.put(0, 1);
int sum = 0, count = 0;
for (int x : arr) {
    sum += x;
    int mod = ((sum % k) + k) % k;             // normalize negatives
    count += modCount.getOrDefault(mod, 0);
    modCount.merge(mod, 1, Integer::sum);
}
return count;
```
Two prefixes with the same `sum % k` -> subarray between them is divisible by k.

**Template (Continuous Subarray Sum - existence of one of length >= 2 with sum % k == 0):**
```java
Map<Integer, Integer> firstSeenIdx = new HashMap<>();
firstSeenIdx.put(0, -1);                       // sum 0 at index -1 means subarrays starting at 0
int sum = 0;
for (int i = 0; i < n; i++) {
    sum += arr[i];
    int mod = ((sum % k) + k) % k;
    if (firstSeenIdx.containsKey(mod)) {
        if (i - firstSeenIdx.get(mod) >= 2) return true;
    } else {
        firstSeenIdx.put(mod, i);              // first occurrence only - we want LONGEST gap
    }
}
return false;
```

**Variations:**
- **Binary Subarrays With Sum** (0/1 only, count subarrays with sum = K): same template, simpler.
- **Path Sum III** (binary tree): same prefix-sum-map trick with DFS + undo on the way back.
- **Number of Submatrices That Sum to Target**: for each row pair, collapse to 1D and use this template.

**Common bugs:**
- Forgetting `prefixCount.put(0, 1)` - subarrays starting at index 0 with sum = K go uncounted.
- Java's `%` on negatives returns negative - use `((x % k) + k) % k` to normalize.
- Mixing `prefix -> count` (for COUNTING subarrays) with `prefix -> firstIndex` (for LONGEST). They're different patterns.

---

## 4. Prefix State + First-Occurrence Map (longest subarray with property)

**When to use it:** Find the LONGEST subarray satisfying a property (sum = K, equal 0s and 1s, sum divisible by K, etc.). The map stores `state -> earliest index seen`. When we see the same state again, the subarray between is the answer candidate.

**Trigger phrases:** "longest subarray with sum K", "contiguous array (equal 0s and 1s)", "longest subarray sum divisible by K", "find longest balanced".

**Anchor problem:** Contiguous Array (equal number of 0s and 1s).

**Template (Contiguous Array):**
```java
Map<Integer, Integer> firstSeen = new HashMap<>();
firstSeen.put(0, -1);                          // sum 0 at index -1
int sum = 0, best = 0;
for (int i = 0; i < n; i++) {
    sum += (arr[i] == 0 ? -1 : 1);             // treat 0 as -1
    if (firstSeen.containsKey(sum)) {
        best = Math.max(best, i - firstSeen.get(sum));
    } else {
        firstSeen.put(sum, i);                 // FIRST occurrence only - longest gap
    }
}
return best;
```

**Template (Longest Subarray With Sum K - allows negatives):**
```java
Map<Integer, Integer> firstSeen = new HashMap<>();
firstSeen.put(0, -1);
int sum = 0, best = 0;
for (int i = 0; i < n; i++) {
    sum += arr[i];
    if (firstSeen.containsKey(sum - k)) {
        best = Math.max(best, i - firstSeen.get(sum - k));
    }
    firstSeen.putIfAbsent(sum, i);             // putIfAbsent = only first occurrence
}
return best;
```

**Variations:**
- **Longest Subarray Sum Divisible by K**: same template with `sum % k` as state.
- **Maximum Size Subarray Sum Equals K**: same template, generic version of contiguous-array.
- **Longest Well-Performing Interval** (hours > 8 -> +1, else -1): state = running balance.

**Common bugs:**
- Using `put` instead of `putIfAbsent` - overwrites the earliest index, giving shorter spans.
- Using `prefix -> count` (good for COUNTING) instead of `prefix -> firstIndex` (right for LONGEST).
- Forgetting `firstSeen.put(0, -1)` - subarrays starting at index 0 not considered.

---

## 5. Grouping / Bucketing

**When to use it:** Group input elements by some derived key (anagram signature, word pattern, hash of structure, mod result).

**Trigger phrases:** "group anagrams", "group strings by pattern", "group people by birth month", "group connected components".

**Anchor problem:** Group Anagrams.

**Template (Group Anagrams - sorted signature):**
```java
Map<String, List<String>> groups = new HashMap<>();
for (String w : words) {
    char[] arr = w.toCharArray();
    Arrays.sort(arr);
    String key = new String(arr);
    groups.computeIfAbsent(key, k -> new ArrayList<>()).add(w);
}
return new ArrayList<>(groups.values());
```

**Template (Group Anagrams - frequency signature, faster for long words):**
```java
Map<String, List<String>> groups = new HashMap<>();
for (String w : words) {
    int[] freq = new int[26];
    for (char c : w.toCharArray()) freq[c - 'a']++;
    StringBuilder sb = new StringBuilder();
    for (int f : freq) sb.append(f).append('#');     // delimiter prevents collision (1,12 vs 11,2)
    groups.computeIfAbsent(sb.toString(), k -> new ArrayList<>()).add(w);
}
```

**Variations:**
- **Group Shifted Strings** (`abc`, `bcd`, `xyz` are same shift pattern): key = differences between consecutive chars mod 26.
- **Custom Sort String** (group, then output by frequency): combine grouping with sorted output.
- **Find Duplicate File in System**: key = file content; group by content.

**Common bugs:**
- Using bare string concatenation (`sb.append(f)`) without a delimiter - `freq=[1,12,...]` and `freq=[11,2,...]` collide on signature `112...`.
- Wasting allocations: use `int[]` signature when possible (encoded as String with delimiter).
- Returning `groups.values()` directly - return type is `Collection<List<String>>`; wrap as `new ArrayList<>(groups.values())` if `List<List<String>>` required.

---

## 6. Bidirectional / Two-Map Mapping

**When to use it:** Check or build a one-to-one (bijection) mapping between two domains - characters, words, anything. Need BOTH directions to detect violations.

**Trigger phrases:** "isomorphic strings", "word pattern", "valid bijection", "translate uniquely".

**Anchor problem:** Isomorphic Strings.

**Template (Isomorphic Strings):**
```java
if (s.length() != t.length()) return false;
Map<Character, Character> s2t = new HashMap<>();
Map<Character, Character> t2s = new HashMap<>();
for (int i = 0; i < s.length(); i++) {
    char a = s.charAt(i), b = t.charAt(i);
    if (s2t.containsKey(a) && s2t.get(a) != b) return false;
    if (t2s.containsKey(b) && t2s.get(b) != a) return false;
    s2t.put(a, b);
    t2s.put(b, a);
}
return true;
```

**Template (Word Pattern - one map char->word, one set tracks used words):**
```java
String[] words = s.split(" ");
if (pattern.length() != words.length) return false;
Map<Character, String> charToWord = new HashMap<>();
Set<String> usedWords = new HashSet<>();
for (int i = 0; i < pattern.length(); i++) {
    char c = pattern.charAt(i);
    String w = words[i];
    if (charToWord.containsKey(c)) {
        if (!charToWord.get(c).equals(w)) return false;
    } else {
        if (usedWords.contains(w)) return false;     // word already taken by another char
        charToWord.put(c, w);
        usedWords.add(w);
    }
}
return true;
```

**Variations:**
- **Verifying an Alien Dictionary**: map char -> rank, then compare adjacent words.
- **Find and Replace Pattern**: same as isomorphic, but checked per candidate.

**Common bugs:**
- Using ONE map only - can map two different chars to the same target. The reverse map catches this.
- Using `==` on Character/String - **always** `.equals()` for content.

---

## 7. Sliding Window + Frequency Map

**When to use it:** Substring/subarray problems with character-distribution constraints - "at most K distinct", "contains all of T", "permutation match".

**Trigger phrases:** "longest substring with at most K distinct characters", "minimum window substring", "permutation in string", "find all anagrams in string".

**Anchor problem:** Longest Substring with At Most K Distinct Characters.

**Template (At Most K Distinct):**
```java
Map<Character, Integer> freq = new HashMap<>();
int l = 0, best = 0;
for (int r = 0; r < s.length(); r++) {
    freq.merge(s.charAt(r), 1, Integer::sum);
    while (freq.size() > k) {                            // too many distinct chars
        char left = s.charAt(l++);
        if (freq.merge(left, -1, Integer::sum) == 0) freq.remove(left);
    }
    best = Math.max(best, r - l + 1);
}
```

**Template (Min Window Substring - need all chars of T):**
```java
Map<Character, Integer> need = new HashMap<>();
for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);

Map<Character, Integer> have = new HashMap<>();
int required = need.size();                              // distinct chars in T needed
int formed = 0;                                          // distinct chars currently satisfied
int l = 0, bestLen = Integer.MAX_VALUE, bestL = 0;

for (int r = 0; r < s.length(); r++) {
    char c = s.charAt(r);
    have.merge(c, 1, Integer::sum);
    if (need.containsKey(c) && have.get(c).intValue() == need.get(c).intValue()) {
        formed++;
    }
    while (formed == required) {
        if (r - l + 1 < bestLen) { bestLen = r - l + 1; bestL = l; }
        char left = s.charAt(l++);
        have.merge(left, -1, Integer::sum);
        if (need.containsKey(left) && have.get(left) < need.get(left)) formed--;
    }
}
return bestLen == Integer.MAX_VALUE ? "" : s.substring(bestL, bestL + bestLen);
```

**Variations:**
- **Permutation in String / Find All Anagrams**: fixed-size window with frequency arrays.
- **Subarrays with K Different Integers**: `exactly K = atMost(K) - atMost(K-1)` trick.
- **Fruit Into Baskets**: at-most-2-distinct version.

**Common bugs:**
- Comparing boxed `Integer` with `==` - auto-unboxing only happens in some contexts. Use `.intValue()` or `.equals()` for clarity.
- Not removing keys when count hits 0 - `freq.size()` becomes wrong, breaking "at most K distinct".
- For min-window, returning `r - l + 1` from the WRONG `l` (after final shrink past optimum). Capture `bestL`, `bestLen` at each improvement.

---

## 8. Map + Heap (Top K Frequent)

**When to use it:** Find the K most frequent (or rarest) elements - O(n log k) with heap, or O(n) with bucket sort.

**Trigger phrases:** "top K frequent elements", "K most common words", "Kth most frequent", "K closest points".

**Anchor problem:** Top K Frequent Elements.

**Template (frequency map + min-heap of size K):**
```java
Map<Integer, Integer> freq = new HashMap<>();
for (int x : arr) freq.merge(x, 1, Integer::sum);

PriorityQueue<int[]> minHeap = new PriorityQueue<>(Comparator.comparingInt(a -> a[1]));
for (var e : freq.entrySet()) {
    minHeap.offer(new int[]{e.getKey(), e.getValue()});
    if (minHeap.size() > k) minHeap.poll();
}

int[] result = new int[k];
for (int i = k - 1; i >= 0; i--) result[i] = minHeap.poll()[0];
return result;
```
Min-heap of size K means "smallest of top K" is at peek - pop when overflow.

**Template (bucket sort by frequency - O(n)):**
```java
Map<Integer, Integer> freq = new HashMap<>();
for (int x : arr) freq.merge(x, 1, Integer::sum);

List<Integer>[] buckets = new List[arr.length + 1];   // freq can be at most n
for (var e : freq.entrySet()) {
    int f = e.getValue();
    if (buckets[f] == null) buckets[f] = new ArrayList<>();
    buckets[f].add(e.getKey());
}

int[] result = new int[k];
int idx = 0;
for (int f = buckets.length - 1; f >= 0 && idx < k; f--) {
    if (buckets[f] != null) for (int x : buckets[f]) {
        result[idx++] = x;
        if (idx == k) break;
    }
}
return result;
```

**Variations:**
- **Top K Frequent Words**: comparator breaks ties alphabetically. Be careful with the min-heap direction.
- **Sort Characters By Frequency**: build freq, then sort by descending freq, then rebuild string.

**Common bugs:**
- For top K, using a MAX-heap of size N (sorts everything) - O(n log n) instead of O(n log k).
- For tied frequencies, comparator direction matters - in a min-heap, alphabetically LATER words should be at the top (so they get popped first). Easy to get backwards.

---

## 9. LRU / LFU Cache (Map + Auxiliary Structure)

**When to use it:** Designing a fixed-capacity cache with an eviction policy.

**Trigger phrases:** "design LRU cache", "design LFU cache", "least recently used eviction".

**Anchor problem:** LRU Cache.

**Template (LRU - HashMap + Doubly Linked List, both O(1)):**
```java
class LRUCache {
    class Node { int key, val; Node prev, next; Node(int k, int v) { key = k; val = v; } }

    private final Map<Integer, Node> map = new HashMap<>();
    private final int cap;
    private final Node head = new Node(0, 0), tail = new Node(0, 0);   // sentinels

    LRUCache(int cap) {
        this.cap = cap;
        head.next = tail; tail.prev = head;
    }

    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        Node n = map.get(key);
        moveToFront(n);
        return n.val;
    }

    public void put(int key, int val) {
        if (map.containsKey(key)) {
            Node n = map.get(key);
            n.val = val;
            moveToFront(n);
        } else {
            if (map.size() == cap) {
                Node lru = tail.prev;
                remove(lru);
                map.remove(lru.key);
            }
            Node n = new Node(key, val);
            map.put(key, n);
            addToFront(n);
        }
    }

    private void addToFront(Node n) {
        n.next = head.next; n.prev = head;
        head.next.prev = n; head.next = n;
    }
    private void remove(Node n) {
        n.prev.next = n.next; n.next.prev = n.prev;
    }
    private void moveToFront(Node n) { remove(n); addToFront(n); }
}
```

**Shortcut (LinkedHashMap - built-in LRU):**
```java
class LRUCache extends LinkedHashMap<Integer, Integer> {
    private final int cap;
    LRUCache(int cap) {
        super(cap, 0.75f, true);                    // accessOrder = true
        this.cap = cap;
    }
    public int get(int key) { return super.getOrDefault(key, -1); }
    public void put(int key, int val) { super.put(key, val); }
    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > cap;
    }
}
```

**LFU outline:** `Map<key, val>` + `Map<key, freq>` + `Map<freq, LinkedHashSet<key>>` + `minFreq` counter. On get/put, increment freq, move key between freq buckets. On eviction, remove from `freqMap.get(minFreq)`'s set.

**Variations:**
- **Insert Delete GetRandom O(1)**: `Map<val, index>` + `ArrayList<val>`. For O(1) delete: swap with last, pop.
- **Insert Delete GetRandom O(1) - Duplicates Allowed**: same, but value -> Set of indices.

**Common bugs:**
- Forgetting sentinel head/tail - boundary checks explode in `remove` / `addToFront`.
- For LinkedHashMap shortcut: forgetting `accessOrder = true` - falls back to insertion order.
- For LFU: forgetting to update `minFreq` after eviction or after incrementing the last key in the `minFreq` bucket.

---

## 10. Visited / Dedup Set

**When to use it:** Track seen elements - cycle detection, deduplication, uniqueness checks. HashSet is the dedicated tool.

**Trigger phrases:** "contains duplicate", "happy number (detect cycle)", "first duplicate", "intersection of arrays", "longest substring without repeating characters".

**Anchor problem:** Contains Duplicate.

**Template (basic dedup):**
```java
Set<Integer> seen = new HashSet<>();
for (int x : arr) {
    if (!seen.add(x)) return true;             // add() returns false if already present
}
return false;
```

**Template (Happy Number - cycle detection):**
```java
public boolean isHappy(int n) {
    Set<Integer> seen = new HashSet<>();
    while (n != 1 && seen.add(n)) {
        n = sumOfSquares(n);
    }
    return n == 1;
}
```
Or use Floyd's tortoise-and-hare for O(1) space.

**Template (Linked List Cycle - using set):**
```java
Set<ListNode> seen = new HashSet<>();
ListNode cur = head;
while (cur != null) {
    if (!seen.add(cur)) return true;
    cur = cur.next;
}
return false;
```
(Floyd's slow/fast is preferred for O(1) space.)

**Variations:**
- **Intersection of Two Arrays**: add one to set; iterate other, keep what's in set.
- **Longest Consecutive Sequence**: put all in set; for each x where `x-1` not in set, count run length forward.
- **Valid Sudoku**: 27 sets (9 rows, 9 cols, 9 boxes), add and check duplicates per cell.

**Common bugs:**
- Adding before checking - lose the duplicate signal. Use `Set.add(x)` (returns boolean).
- For `Set<int[]>` - arrays use reference equality, not content. Wrap in `String.valueOf` or use `Arrays.asList`.

---

## 11. Multi-Field / Encoded Keys

**When to use it:** You need a HashMap whose key is a tuple - coordinates, multi-field state, etc. Java's `Map<List<Integer>, V>` works but is slow; `record` (Java 16+) or encoded String is usually better.

**Trigger phrases:** "coord -> value", "tuple-keyed map", "state hashing", "BFS with visited tuples".

**Anchor problem:** Number of Islands II.

**Template (encoded String key):**
```java
Map<String, Integer> visited = new HashMap<>();
String key = i + "," + j;
visited.put(key, count);
```

**Template (record-based key, Java 16+):**
```java
record Coord(int r, int c) {}      // equals/hashCode auto-generated

Map<Coord, Integer> visited = new HashMap<>();
visited.put(new Coord(i, j), count);
```
Clean, fast, no string concat. **Preferred when on Java 16+.**

**Template (single int encoding for small ranges):**
```java
// For grids where i, j < 1000:
int key = i * 1000 + j;            // unique encoding
visited.put(key, count);
```
Fastest - no allocation. Watch overflow for large grids.

**Template (Long encoding for larger ranges):**
```java
long key = ((long) i << 32) | (j & 0xFFFFFFFFL);
```

**Variations:**
- **Sudoku Solver visited state**: encode (row, col, num) as single int.
- **Word Ladder BFS visited**: use `String` directly as key.
- **Snakes and Ladders**: just an int (position 1..n^2).

**Common bugs:**
- Using `int[]` as map key - arrays compare by reference, not content. Either encode or use `Arrays.toString(arr)` (slow) or `IntBuffer.wrap(arr)`.
- For multi-int encoding, integer overflow when `i * BIG + j` overflows. Use `long`.

---

## 12. TreeMap-Specific Patterns

**When to use it:** Need sorted keys, range queries, or floor/ceiling lookups. TreeMap is `Red-Black tree`, O(log n) operations.

**Trigger phrases:** "find floor / ceiling key", "smallest key >= X", "largest key <= X", "range queries", "time-based key-value".

**Anchor problem:** Time Based Key-Value Store.

**Template (Time Based Key-Value Store):**
```java
class TimeMap {
    private final Map<String, TreeMap<Integer, String>> map = new HashMap<>();

    public void set(String key, String value, int timestamp) {
        map.computeIfAbsent(key, k -> new TreeMap<>()).put(timestamp, value);
    }

    public String get(String key, int timestamp) {
        TreeMap<Integer, String> tm = map.get(key);
        if (tm == null) return "";
        Map.Entry<Integer, String> e = tm.floorEntry(timestamp);   // largest ts <= given
        return e == null ? "" : e.getValue();
    }
}
```

**Useful TreeMap methods:**
```java
TreeMap<K, V> tm = new TreeMap<>();
tm.firstKey(); tm.lastKey();
tm.floorKey(K);     tm.floorEntry(K);     // largest key <= K
tm.ceilingKey(K);   tm.ceilingEntry(K);   // smallest key >= K
tm.higherKey(K);                          // strictly greater
tm.lowerKey(K);                           // strictly less
tm.subMap(low, high);                     // [low, high)
tm.headMap(K);                            // strictly < K
tm.tailMap(K);                            // >= K
tm.descendingMap();                       // reverse view
```

**Variations:**
- **My Calendar I/II/III**: TreeMap of bookings, floor/ceiling to detect overlap.
- **Snapshot Array**: per-index TreeMap of `snapId -> value`; floorEntry on query.

**Common bugs:**
- Using HashMap when you need range queries - O(n) instead of O(log n).
- TreeMap doesn't allow nulls (in default Comparator). Custom Comparator can permit if explicit.
- TreeMap equality is by `compareTo() == 0`, NOT `.equals()` - same trap as TreeSet. Ensure your Comparator distinguishes everything that should be distinct.

---

## 13 (design). Frequency / Time-Backed Designs

**When to use it:** System-design-style problems where you maintain a stream of events and answer queries quickly. Often combines a HashMap with another structure (queue, TreeMap, deque, set).

**Trigger phrases:** "logger rate limiter", "tweet counts per frequency", "time-based key-value store", "design hit counter".

**Anchor problem:** Logger Rate Limiter.

**Template (Logger Rate Limiter):**
```java
class Logger {
    private final Map<String, Integer> lastLogged = new HashMap<>();

    public boolean shouldPrintMessage(int timestamp, String message) {
        Integer last = lastLogged.get(message);
        if (last == null || timestamp - last >= 10) {
            lastLogged.put(message, timestamp);
            return true;
        }
        return false;
    }
}
```

**Template (Hit Counter - last 5 minutes):**
```java
class HitCounter {
    private final Deque<int[]> hits = new ArrayDeque<>();   // {timestamp, count}

    public void hit(int t) {
        if (!hits.isEmpty() && hits.peekLast()[0] == t) hits.peekLast()[1]++;
        else hits.offerLast(new int[]{t, 1});
    }

    public int getHits(int t) {
        while (!hits.isEmpty() && hits.peekFirst()[0] <= t - 300) hits.pollFirst();
        return hits.stream().mapToInt(h -> h[1]).sum();
    }
}
```

**Variations:**
- **Design Twitter**: `Map<user, List<Tweet>>` + `Map<user, Set<followees>>`; merge K sorted lists for feed.
- **Insert Delete GetRandom O(1)**: covered in pattern 9.

**Common bugs:**
- For rate limiter, using `>` instead of `>=` for the time window.
- For hit counter, NOT bucketing same-timestamp hits - O(n) per hit instead of O(1).

---

## Suggested revision order if relearning from scratch

1. **Frequency Counting** - the easiest entry; foundation for many other patterns.
2. **Two-Sum Family** - the iconic HashMap problem; internalize "iterate + check complement first".
3. **Prefix Sum + HashMap (count)** - the workhorse for "count subarrays with sum = K".
4. **Prefix State + first-occurrence map (longest)** - close cousin of #3; learn the distinction.
5. **Grouping / Bucketing** - simple but everywhere; `computeIfAbsent` is your friend.
6. **Bidirectional / Two-Map** - small set of problems but they're all the same trick.
7. **Sliding Window + Frequency Map** - bridge between hashmap and array patterns.
8. **Visited / Dedup Set** - basic; covered fast.
9. **Top K Frequent (Map + Heap)** - common interview ask; learn both heap and bucket-sort solutions.
10. **Multi-Field / Encoded Keys** - small trick that unlocks BFS-on-grid, state hashing.
11. **TreeMap-Specific Patterns** - learn the floor/ceiling API; powers a few hard problems.
12. **LRU / LFU Cache** - design-question staple; spend extra time here.
13. **Frequency-Backed Designs** - rate limiter, hit counter, time-based KV - similar feel.

**Universal traps across every pattern:**
- **Boxing overhead**: `Map<Integer, Integer>` boxes every int. For hot paths, prefer `int[]` (if key is bounded) or specialized libs (Eclipse Collections, Fastutil).
- **NullPointerException on auto-unbox**: `int x = map.get(k)` throws NPE if `k` absent. Use `getOrDefault(k, 0)`.
- **`==` on boxed Integer**: identity comparison; works only for cached range -128..127. ALWAYS use `.equals()` or `.intValue()`.
- **HashMap is NOT thread-safe**: use `ConcurrentHashMap` for multi-threaded contexts.
- **HashMap iteration order is not insertion order** (except for LinkedHashMap).
- **Mutable keys**: if you mutate an object's hashcode after insertion, you can NEVER find it again. Always use immutable types as keys (String, Integer, record, etc.).
- **`Arrays.asList(intArr)`** returns `List<int[]>` (one element!) for primitives. Use `Arrays.stream(intArr).boxed().toList()`.
- **HashMap vs ConcurrentHashMap put-then-check race** (multithreaded): use `putIfAbsent`, `computeIfAbsent`, or `merge` for atomic-like behavior.
