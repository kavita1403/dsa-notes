# Arrays - Cheat Sheet

**Decision fork:** Pair / triplet with target -> two pointers (sorted) or HashMap. Max subarray sum -> Kadane. Subarray sum = K -> prefix sum + HashMap. Missing/duplicate in [1..n] -> cyclic sort. Sorted (or rotated) search -> binary search. Min/max satisfying feasibility -> binary search on answer. Next greater / histogram -> monotonic stack. Sliding window max -> monotonic deque. Remove/partition in place -> two pointers same direction. Kth element -> quickselect or heap. 2D rotate/spiral/zeros -> in-place tricks.

## Two Pointers - Opposite Ends (sorted)
```java
int l = 0, r = n - 1;
while (l < r) {
    int sum = arr[l] + arr[r];
    if (sum == target) return new int[]{l, r};
    else if (sum < target) l++; else r--;
}
```
3-Sum: fix `i`, two-pointer on rest. Skip duplicates at `i`, `l`, `r` carefully.

## Two Pointers - Same Direction (in-place)
```java
int write = 0;
for (int read = 0; read < n; read++) {
    if (keep(arr[read])) arr[write++] = arr[read];
}
return write;
```
Dutch flag (0/1/2 sort): three pointers `low, mid, high`. On 2, swap with `high` and do NOT increment mid.

## Sliding Window - Fixed Size
```java
int sum = 0, best = 0;
for (int r = 0; r < n; r++) {
    sum += arr[r];
    if (r >= k) sum -= arr[r - k];
    if (r >= k - 1) best = Math.max(best, sum);
}
```

## Sliding Window - Variable (shortest with sum >= target)
```java
int l = 0, sum = 0, best = Integer.MAX_VALUE;
for (int r = 0; r < n; r++) {
    sum += arr[r];
    while (sum >= target) { best = Math.min(best, r - l + 1); sum -= arr[l++]; }
}
```
Doesn't work with negatives - sums no longer monotonic in `r`.

## Kadane's Algorithm
```java
int best = arr[0], bestEndingHere = arr[0];
for (int i = 1; i < n; i++) {
    bestEndingHere = Math.max(arr[i], bestEndingHere + arr[i]);
    best = Math.max(best, bestEndingHere);
}
```
Init `best = arr[0]`, NOT 0 (breaks all-negative). Max-product variant: track both max AND min.

## Prefix Sum - Subarray Sum Equals K
```java
Map<Integer, Integer> prefixCount = new HashMap<>();
prefixCount.put(0, 1);                              // empty prefix has sum 0
int sum = 0, count = 0;
for (int x : arr) {
    sum += x;
    count += prefixCount.getOrDefault(sum - k, 0);
    prefixCount.merge(sum, 1, Integer::sum);
}
```
Forgetting `put(0, 1)` -> subarrays starting at index 0 go uncounted. Same idea works for "divisible by K" (store `sum % K`) and "equal 0s and 1s" (treat 0 as -1).

## Prefix Sum - 2D
```java
prefix[i+1][j+1] = matrix[i][j] + prefix[i][j+1] + prefix[i+1][j] - prefix[i][j];
// Rectangle (r1, c1) -> (r2, c2):
sum = prefix[r2+1][c2+1] - prefix[r1][c2+1] - prefix[r2+1][c1] + prefix[r1][c1];
```

## Cyclic Sort (numbers in [1..n])
```java
int i = 0;
while (i < n) {
    int correct = arr[i] - 1;
    if (arr[i] != arr[correct]) swap(arr, i, correct);
    else i++;
}
// arr[j] != j + 1 means j + 1 is missing
```
Use `while`, NOT `for`. After swap, the same `i` may need another swap.

## Binary Search - Standard
```java
int l = 0, r = n - 1;
while (l <= r) {
    int mid = l + (r - l) / 2;                      // overflow-safe
    if (arr[mid] == target) return mid;
    if (arr[mid] < target) l = mid + 1; else r = mid - 1;
}
```

## Binary Search - Rotated Sorted (find target)
```java
while (l <= r) {
    int mid = l + (r - l) / 2;
    if (arr[mid] == target) return mid;
    if (arr[l] <= arr[mid]) {                       // left half sorted
        if (arr[l] <= target && target < arr[mid]) r = mid - 1;
        else l = mid + 1;
    } else {                                         // right half sorted
        if (arr[mid] < target && target <= arr[r]) l = mid + 1;
        else r = mid - 1;
    }
}
```

## Binary Search - Leftmost / First Occurrence
```java
int l = 0, r = n;
while (l < r) {
    int mid = l + (r - l) / 2;
    if (arr[mid] >= target) r = mid;
    else l = mid + 1;
}
return l < n && arr[l] == target ? l : -1;
```

## Binary Search on Answer (ship packages in D days)
```java
int lo = maxOf(weights), hi = sumOf(weights);
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (feasible(weights, days, mid)) hi = mid;
    else lo = mid + 1;
}
return lo;
```
Feasibility check is O(n). `lo` MUST be at least the max single element (or one ship can't fit it).

## Monotonic Stack - Next Greater
```java
Deque<Integer> stack = new ArrayDeque<>();          // indices
int[] result = new int[n]; Arrays.fill(result, -1);
for (int i = 0; i < n; i++) {
    while (!stack.isEmpty() && arr[stack.peek()] < arr[i]) result[stack.pop()] = i;
    stack.push(i);
}
```
Push INDICES, not values. For "largest rectangle in histogram", add a sentinel 0 at the end to flush stack.

## Monotonic Deque - Sliding Window Max
```java
Deque<Integer> dq = new ArrayDeque<>();
for (int i = 0; i < n; i++) {
    while (!dq.isEmpty() && dq.peekFirst() <= i - k) dq.pollFirst();      // expired
    while (!dq.isEmpty() && arr[dq.peekLast()] < arr[i]) dq.pollLast();   // worse, drop
    dq.offerLast(i);
    if (i >= k - 1) result[i - k + 1] = arr[dq.peekFirst()];
}
```

## Sorting + Greedy - Merge Intervals
```java
Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]));
for (int[] cur : intervals) {
    if (out.isEmpty() || out.get(out.size() - 1)[1] < cur[0]) out.add(cur);
    else out.get(out.size() - 1)[1] = Math.max(out.get(out.size() - 1)[1], cur[1]);
}
```
Sort by START for merge / overlap; sort by END for min-removals / arrows.

## Hashing - Two Sum
```java
Map<Integer, Integer> seen = new HashMap<>();
for (int i = 0; i < n; i++) {
    int need = target - arr[i];
    if (seen.containsKey(need)) return new int[]{seen.get(need), i};
    seen.put(arr[i], i);
}
```
Check FIRST, then put - otherwise an element matches itself when target = 2 * x.

## Quickselect (Kth largest, avg O(n))
```java
int quickselect(int[] arr, int lo, int hi, int targetIdx) {
    if (lo == hi) return arr[lo];
    int p = partition(arr, lo, hi);
    if (p == targetIdx) return arr[p];
    if (targetIdx < p) return quickselect(arr, lo, p - 1, targetIdx);
    return quickselect(arr, p + 1, hi, targetIdx);
}
// Kth largest = (n - k)th smallest. Use random pivot to avoid O(n^2) worst case.
```
Alternative: min-heap of size K -> O(n log k), simpler, guaranteed.

## Matrix - Rotate 90 CW (transpose + reverse rows)
```java
for (int i = 0; i < n; i++)
    for (int j = i + 1; j < n; j++)
        { int t = m[i][j]; m[i][j] = m[j][i]; m[j][i] = t; }
for (int[] row : m) reverse(row);
```
90 CCW: reverse rows first, then transpose.

## Universal traps
- **Integer overflow on sum** - use `long` accumulators for big arrays.
- **`Arrays.sort(int[])` has NO Comparator overload** - can't sort descending. Box to `Integer[]` first.
- **`Arrays.asList(intArr)`** returns `List<int[]>` (size 1!). Use `Arrays.stream(intArr).boxed().toList()`.
- **Sliding window assumes non-negative** for "sum >= target" - fails with negatives.
- **`(l + r) / 2` overflow** in binary search - always `l + (r - l) / 2`.
- **Off-by-one in binary search**: pick `l < r` (r = n) OR `l <= r` (r = n - 1) and stick to ONE in a function.
- **Modifying array while iterating** - use separate write index.
