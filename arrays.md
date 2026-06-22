# Array Patterns - Master Reference Note

```java
// Standard assumptions
int[] arr = ...;
int n = arr.length;
Arrays.sort(arr);                        // O(n log n), primitive sort - ascending only
Arrays.stream(arr).sum();                 // for int[]
List<Integer> list = Arrays.stream(arr).boxed().toList();
int[] back = list.stream().mapToInt(Integer::intValue).toArray();
```

---

## Decision Fork (ask yourself this before writing any code)

> "Find a pair / triplet with target sum" -> **Two Pointers** (sort first) OR **Hash Map** for O(n) two-sum.
>
> "Maximum / minimum subarray sum" -> **Kadane's Algorithm**. One pass, two running values.
>
> "Subarray sum equals K" (count or existence) -> **Prefix Sum + HashMap**. Store `prefix -> count`.
>
> "Find missing / duplicate number in `[1..n]`" with O(1) extra space -> **Cyclic Sort** OR XOR trick OR Floyd's cycle on indices.
>
> "Search in sorted / rotated sorted array" -> **Binary Search** (carefully handle the rotation pivot).
>
> "Next greater / smaller element" or "stock span" -> **Monotonic Stack**.
>
> "Sliding window maximum / minimum" -> **Monotonic Deque** (different from stack - need both ends).
>
> "Remove duplicates / move zeros / dutch flag" -> **Two Pointers same direction** (write index lags read index).
>
> "Kth largest / smallest" -> **Heap** (size K) OR **Quickselect** (avg O(n)).
>
> "Merge sorted arrays / find median of two sorted" -> **Two Pointers** OR **Binary Search on Split**.
>
> "2D matrix: rotate, spiral, set zeros, search 2D" -> **In-place patterns** specific to each problem.
>
> "Range query (sum / min / max) with updates" -> **Segment Tree / Fenwick Tree** (advanced).

Everything else in array-land is a variant or combination of these.

---

## Quick Lookup Table (read this first when revisiting after a gap)

| # | Pattern | Trigger phrases | Anchor problem | Core idea |
|---|---|---|---|---|
| 1 | Two Pointers (opposite ends) | "pair with target sum", "container with most water", "3-sum", "trap rainwater" | Two Sum II (sorted) | Sort, converge `l` and `r`; move based on comparison vs target |
| 2 | Two Pointers (same direction) | "remove duplicates in place", "move zeros", "partition / dutch flag" | Remove Duplicates from Sorted Array | `write` index lags `read` index; copy only when valid |
| 3 | Sliding Window | "longest / shortest contiguous", "max sum of size K", "subarray with at most K distinct" | Maximum Sum Subarray of Size K | Expand r, shrink l on violation; track answer per step |
| 4 | Kadane's Algorithm | "max subarray sum", "circular subarray", "max product subarray" | Maximum Subarray | At each i, `bestEndingHere = max(arr[i], bestEndingHere + arr[i])` |
| 5 | Prefix Sum (1D + 2D) | "range sum", "subarray sum equals K", "continuous subarray sum divisible by K" | Subarray Sum Equals K | `prefix[i] - prefix[j] = sum(j..i)`; HashMap for O(n) |
| 6 | Cyclic Sort | "find missing / duplicate in [1..n]", "first missing positive" | Find All Missing Numbers in Array | Place each element at index `arr[i] - 1`; mismatch = answer |
| 7 | Binary Search (on array) | "search in sorted / rotated sorted", "first/last occurrence", "peak element" | Search in Rotated Sorted Array | Identify which half is sorted, then binary search within bounds |
| 8 | Binary Search on Answer | "min largest sum after split", "Koko eating bananas", "ship packages in D days" | Capacity to Ship Packages | Binary search the answer space; check feasibility in O(n) |
| 9 | Monotonic Stack | "next greater / smaller element", "largest rectangle in histogram", "daily temperatures" | Next Greater Element | Stack holds INDICES; pop while incoming breaks monotonic invariant |
| 10 | Monotonic Deque | "sliding window max / min" | Sliding Window Maximum | Deque of indices; front = current max; pop back while smaller |
| 11 | Sorting Tricks | "merge intervals", "meeting rooms", "minimum platforms", "custom comparator" | Merge Intervals | Sort by start (or by custom key); sweep linearly |
| 12 | Hashing on Arrays | "two sum (unsorted)", "longest consecutive sequence", "group anagrams" | Two Sum | Map value -> index or count; lookup complement |
| 13 | Quickselect / Partition | "Kth largest", "top K", "Dutch national flag" | Kth Largest Element in an Array | Lomuto / Hoare partition; recurse only on the side that contains K |
| 14 | Matrix Patterns | "rotate 90", "spiral order", "set zeros", "search 2D matrix" | Rotate Image | In-place transpose + reverse rows for 90 rotate; layer-by-layer for spiral |
| 15 (advanced) | Segment / Fenwick Tree | "range sum/min/max with updates", "count of smaller after self" | Range Sum Query - Mutable | Tree where each node stores aggregate over a range |

---

## 1. Two Pointers (opposite ends - converging)

**When to use it:** Sorted array, looking for a pair/triplet satisfying a condition based on sum/comparison. Or comparing both ends symmetrically (palindrome, container).

**Trigger phrases:** "two sum sorted", "3-sum", "4-sum", "container with most water", "trapping rain water", "valid palindrome (array of chars)".

**Anchor problem:** Two Sum II - Input Array is Sorted.

**Template:**
```java
int l = 0, r = n - 1;
while (l < r) {
    int sum = arr[l] + arr[r];
    if (sum == target) return new int[]{l, r};
    else if (sum < target) l++;
    else r--;
}
return new int[]{-1, -1};
```

**Variations:**
- **3-Sum**: fix `i`, run two-pointer on `[i+1, n-1]` for `target - arr[i]`. Skip duplicates carefully.
  ```java
  Arrays.sort(arr);
  for (int i = 0; i < n - 2; i++) {
      if (i > 0 && arr[i] == arr[i - 1]) continue;       // skip dup at i
      int l = i + 1, r = n - 1;
      while (l < r) {
          int sum = arr[i] + arr[l] + arr[r];
          if (sum == 0) {
              result.add(List.of(arr[i], arr[l], arr[r]));
              while (l < r && arr[l] == arr[l + 1]) l++;  // skip dup at l
              while (l < r && arr[r] == arr[r - 1]) r--;  // skip dup at r
              l++; r--;
          } else if (sum < 0) l++; else r--;
      }
  }
  ```
- **Container With Most Water**: `area = min(h[l], h[r]) * (r - l)`; move the SHORTER side inward.
- **Trapping Rain Water**: maintain `leftMax` and `rightMax`; the side with smaller max determines current cell's water.

**Common bugs:**
- Forgetting to sort before two-pointer on unsorted input.
- Duplicate handling in 3-Sum - either skip in 3 places (above) or use a Set; skipping is faster.
- For trap-rain-water two-pointer: comparing `arr[l]` and `arr[r]` is enough; no need to compute both maxes upfront.

---

## 2. Two Pointers (same direction - read/write)

**When to use it:** Modify an array in place where some elements need to be kept, others removed/shifted. `read` advances through input, `write` lags behind and only advances when we keep a value.

**Trigger phrases:** "remove duplicates in place", "move zeros to end", "remove element equal to X", "dutch national flag (sort 0s, 1s, 2s)".

**Anchor problem:** Remove Duplicates from Sorted Array.

**Template:**
```java
int write = 0;
for (int read = 0; read < n; read++) {
    if (keep(arr[read])) {
        arr[write++] = arr[read];
    }
}
return write;             // new length
```

**Variations:**
- **Remove Duplicates from Sorted Array**: keep if `read == 0 || arr[read] != arr[read - 1]`.
- **Remove Duplicates (allow up to 2)**: keep if `write < 2 || arr[read] != arr[write - 2]`.
- **Move Zeros**: keep non-zero. After the loop, fill remaining positions with 0.
- **Dutch National Flag** (0/1/2 sort): three pointers `low`, `mid`, `high`. Swap `arr[mid]` with `arr[low]` (and l++, mid++) if 0; `mid++` if 1; swap with `arr[high]` (and high--) if 2.
  ```java
  int low = 0, mid = 0, high = n - 1;
  while (mid <= high) {
      if (arr[mid] == 0) swap(arr, low++, mid++);
      else if (arr[mid] == 1) mid++;
      else swap(arr, mid, high--);
  }
  ```

**Common bugs:**
- For Dutch flag, when swapping with `high`, do NOT increment `mid` - the swapped value hasn't been processed.
- Resetting elements after `write` for "move zeros" is required if the problem says original length must equal new length with zeros at end.

---

## 3. Sliding Window (on integer arrays)

**When to use it:** Find a CONTIGUOUS subarray satisfying a constraint (max sum of size K, longest with property X, smallest with sum >= target).

**Trigger phrases:** "maximum sum subarray of size K", "minimum size subarray with sum >= target", "longest subarray with at most K zeros", "fruits into baskets".

**Anchor problem:** Maximum Sum Subarray of Size K.

**Template (fixed size K):**
```java
int sum = 0, best = 0;
for (int r = 0; r < n; r++) {
    sum += arr[r];
    if (r >= k) sum -= arr[r - k];           // slide left edge
    if (r >= k - 1) best = Math.max(best, sum);
}
return best;
```

**Template (variable size, find shortest/longest with constraint):**
```java
int l = 0, sum = 0, best = Integer.MAX_VALUE;       // for shortest
for (int r = 0; r < n; r++) {
    sum += arr[r];
    while (sum >= target) {
        best = Math.min(best, r - l + 1);
        sum -= arr[l++];
    }
}
return best == Integer.MAX_VALUE ? 0 : best;
```

**Variations:**
- **Min Size Subarray Sum >= target**: variable window template above.
- **Longest Subarray with Sum K** (positives only): variable window.
- **Longest Subarray with At Most K Zeros**: track count of zeros in window; shrink when count > k.
- **Max Consecutive Ones III** (flip at most K zeros): same as above.

**Common bugs:**
- Mixing fixed and variable patterns - if size is fixed, you don't need a while-shrink loop.
- Forgetting `r >= k - 1` guard for fixed window before recording answer.
- Using sliding window on arrays with **negative numbers** for "sum >= target" - sums don't grow monotonically with r. Use prefix sum + monotonic deque or sorted structure instead.

---

## 4. Kadane's Algorithm (max subarray sum)

**When to use it:** Find a contiguous subarray with max (or min) sum. Single pass, O(n) time, O(1) space.

**Trigger phrases:** "maximum subarray sum", "max product subarray", "circular subarray sum", "min subarray sum".

**Anchor problem:** Maximum Subarray.

**Template:**
```java
int best = arr[0];
int bestEndingHere = arr[0];
for (int i = 1; i < n; i++) {
    bestEndingHere = Math.max(arr[i], bestEndingHere + arr[i]);
    best = Math.max(best, bestEndingHere);
}
return best;
```

**Intuition:** at each index, decide: do I extend the existing subarray, or start a new one at `arr[i]`?

**Variations:**
- **Max Product Subarray**: track both max AND min (since multiplying two negatives gives positive). Swap them when `arr[i] < 0`.
  ```java
  int maxP = arr[0], minP = arr[0], best = arr[0];
  for (int i = 1; i < n; i++) {
      if (arr[i] < 0) { int t = maxP; maxP = minP; minP = t; }
      maxP = Math.max(arr[i], maxP * arr[i]);
      minP = Math.min(arr[i], minP * arr[i]);
      best = Math.max(best, maxP);
  }
  ```
- **Max Circular Subarray Sum**: `max(kadaneMax, totalSum - kadaneMin)`. Edge case: all negative -> return kadaneMax.
- **Max Subarray Length with Sum = K** (allows negatives): NOT Kadane, use prefix sum + HashMap.

**Common bugs:**
- Initializing `best = 0` instead of `arr[0]` breaks all-negative inputs.
- For circular, forgetting the all-negative edge case.

---

## 5. Prefix Sum (1D + 2D)

**When to use it:** Multiple range-sum queries on an immutable array, OR finding subarrays with target sum (especially with negatives where sliding window fails).

**Trigger phrases:** "range sum query immutable", "subarray sum equals K", "number of subarrays with sum K", "continuous subarray sum (divisible by K)", "range sum 2D matrix".

**Anchor problem:** Subarray Sum Equals K.

**Template (1D prefix sum, build):**
```java
int[] prefix = new int[n + 1];
for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + arr[i];
// Sum of arr[l..r] = prefix[r + 1] - prefix[l]
```

**Template (Subarray Sum Equals K):**
```java
Map<Integer, Integer> prefixCount = new HashMap<>();
prefixCount.put(0, 1);                          // empty prefix has sum 0
int sum = 0, count = 0;
for (int x : arr) {
    sum += x;
    count += prefixCount.getOrDefault(sum - k, 0);   // any prior prefix with sum (sum - k)
    prefixCount.merge(sum, 1, Integer::sum);
}
return count;
```
The trick: if `currentPrefix - earlierPrefix = K`, then subarray between them sums to K.

**Template (2D prefix sum / Range Sum Query 2D):**
```java
int[][] prefix = new int[m + 1][n + 1];
for (int i = 0; i < m; i++) {
    for (int j = 0; j < n; j++) {
        prefix[i + 1][j + 1] = matrix[i][j]
            + prefix[i][j + 1] + prefix[i + 1][j] - prefix[i][j];   // inclusion-exclusion
    }
}
// Sum of rectangle (r1, c1) to (r2, c2):
int rectSum = prefix[r2 + 1][c2 + 1] - prefix[r1][c2 + 1] - prefix[r2 + 1][c1] + prefix[r1][c1];
```

**Variations:**
- **Continuous Subarray Sum (Divisible by K)**: store `prefix % K` in the map; same prefix % K twice = subarray divisible by K.
- **Contiguous Array (equal 0s and 1s)**: treat 0 as -1, look for two prefixes with same value.
- **Range Sum Query - Immutable**: just build prefix once, answer in O(1).
- **Path Sum III (binary tree)**: same prefix sum trick, with DFS recursion and undo on the way up.

**Common bugs:**
- Initializing prefix with size `n` instead of `n + 1` - off-by-one in queries.
- Forgetting `prefixCount.put(0, 1)` - subarrays starting at index 0 with sum K go uncounted.
- For 2D: inclusion-exclusion sign error - subtract once, add the doubly-subtracted region back.

---

## 6. Cyclic Sort (in-place rearrangement for [1..n])

**When to use it:** Array of size n containing numbers in range `[1..n]` (or `[0..n-1]`). Find missing / duplicate / first missing positive WITHOUT extra space.

**Trigger phrases:** "find all missing numbers in array of [1..n]", "find duplicate number", "first missing positive", "set mismatch".

**Anchor problem:** Find All Numbers Disappeared in an Array.

**Template:**
```java
int i = 0;
while (i < n) {
    int correctIdx = arr[i] - 1;                 // value v belongs at index v-1
    if (arr[i] != arr[correctIdx]) {
        swap(arr, i, correctIdx);                // place arr[i] at correct spot
    } else {
        i++;                                      // already in place OR duplicate; move on
    }
}
// After this, index i should hold value i+1; mismatches are answers
List<Integer> missing = new ArrayList<>();
for (int j = 0; j < n; j++) {
    if (arr[j] != j + 1) missing.add(j + 1);
}
```

**Variations:**
- **Find the Duplicate Number** (only one duplicate, n+1 numbers in [1..n]): cyclic sort finds it, OR use Floyd's tortoise-and-hare on indices treating array as a linked list.
- **First Missing Positive**: same cyclic sort, but ignore values <= 0 or > n during placement.
- **Find All Duplicates**: cyclic sort, then values not at their index are dupes.

**Common bugs:**
- Using `for (int i = 0; i < n; i++)` instead of `while` - after swap, the same index `i` may need another swap.
- For "first missing positive", filtering out invalid values before the swap is required.

---

## 7. Binary Search on a Sorted Array (and rotated variant)

**When to use it:** Search a value or boundary in O(log n) on a sorted (or rotated sorted) array.

**Trigger phrases:** "search in sorted array", "search in rotated sorted array", "first/last occurrence", "find peak element", "find min in rotated sorted".

**Anchor problem:** Search in Rotated Sorted Array.

**Template (standard binary search):**
```java
int l = 0, r = n - 1;
while (l <= r) {
    int mid = l + (r - l) / 2;                   // avoid integer overflow
    if (arr[mid] == target) return mid;
    if (arr[mid] < target) l = mid + 1;
    else r = mid - 1;
}
return -1;
```

**Template (search in rotated sorted - identify sorted half):**
```java
int l = 0, r = n - 1;
while (l <= r) {
    int mid = l + (r - l) / 2;
    if (arr[mid] == target) return mid;
    if (arr[l] <= arr[mid]) {                    // left half is sorted
        if (arr[l] <= target && target < arr[mid]) r = mid - 1;
        else l = mid + 1;
    } else {                                      // right half is sorted
        if (arr[mid] < target && target <= arr[r]) l = mid + 1;
        else r = mid - 1;
    }
}
return -1;
```

**Template (first occurrence of target / leftmost):**
```java
int l = 0, r = n;
while (l < r) {
    int mid = l + (r - l) / 2;
    if (arr[mid] >= target) r = mid;
    else l = mid + 1;
}
return l < n && arr[l] == target ? l : -1;
```

**Variations:**
- **Find Min in Rotated Sorted Array**: compare `arr[mid]` with `arr[r]`. If `arr[mid] > arr[r]`, min is in right half.
- **Find Peak Element**: compare `arr[mid]` with `arr[mid + 1]`. If `arr[mid] > arr[mid + 1]`, peak is in left (including mid).
- **Search Insert Position**: leftmost binary search.

**Common bugs:**
- `(l + r) / 2` overflows for huge arrays - always `l + (r - l) / 2`.
- Off-by-one in loop bounds: `l <= r` vs `l < r` depends on what `r` means. Be consistent.
- For rotated sorted with duplicates - the algorithm above can degrade to O(n); handle equal case by `l++` to skip.

---

## 8. Binary Search on Answer

**When to use it:** Problem asks for the minimum / maximum X satisfying some monotonic feasibility check. You binary-search X, and for each candidate, check feasibility in O(n).

**Trigger phrases:** "minimum largest sum after splitting array", "Koko eating bananas - min speed", "ship packages within D days - min capacity", "split array largest sum", "magnetic force between balls".

**Anchor problem:** Capacity to Ship Packages Within D Days.

**Template:**
```java
int lo = max(weights);                            // smallest viable capacity
int hi = sum(weights);                            // largest needed (one ship)
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (feasible(weights, days, mid)) hi = mid;   // can ship with capacity mid; try smaller
    else lo = mid + 1;
}
return lo;

private boolean feasible(int[] w, int days, int cap) {
    int needed = 1, current = 0;
    for (int x : w) {
        if (current + x > cap) { needed++; current = 0; }
        current += x;
    }
    return needed <= days;
}
```

**Variations:**
- **Koko Eating Bananas**: search for min eating speed K such that all piles can be eaten in H hours.
- **Split Array Largest Sum**: minimize the max sum among M splits.
- **Magnetic Force Between Balls**: maximize min force; binary search the min distance.

**Common bugs:**
- Wrong `lo` / `hi` bounds: `lo` must be feasible-OR-needs-checking, `hi` must always be feasible.
- Using `hi = mid - 1` when looking for leftmost feasible - skip the answer. Use `hi = mid`.
- Forgetting that the feasibility check must be O(n) (or close); otherwise overall O(n^2) or worse.

---

## 9. Monotonic Stack

**When to use it:** "For each element, find the next (or previous) greater (or smaller) element." Single-pass O(n).

**Trigger phrases:** "next greater element", "next smaller element", "daily temperatures", "stock span", "largest rectangle in histogram", "trapping rain water (stack variant)".

**Anchor problem:** Next Greater Element I (or Daily Temperatures).

**Template (next greater - decreasing stack of indices):**
```java
int[] result = new int[n];
Arrays.fill(result, -1);                        // or 0 for "days until warmer"
Deque<Integer> stack = new ArrayDeque<>();      // indices
for (int i = 0; i < n; i++) {
    while (!stack.isEmpty() && arr[stack.peek()] < arr[i]) {
        int idx = stack.pop();
        result[idx] = i;                         // or arr[i] for value
    }
    stack.push(i);
}
return result;
```

**Template (Largest Rectangle in Histogram):**
```java
Deque<Integer> stack = new ArrayDeque<>();
int best = 0;
for (int i = 0; i <= n; i++) {
    int h = (i == n) ? 0 : arr[i];              // sentinel zero forces flush at end
    while (!stack.isEmpty() && arr[stack.peek()] > h) {
        int top = stack.pop();
        int width = stack.isEmpty() ? i : i - stack.peek() - 1;
        best = Math.max(best, arr[top] * width);
    }
    stack.push(i);
}
```

**Variations:**
- **Daily Temperatures**: same template, result stores days difference (`i - idx`).
- **Stock Span Problem**: previous greater element template (stack monotonic decreasing from bottom up).
- **Maximal Rectangle (2D)**: build histogram per row, run histogram algorithm.
- **Sum of Subarray Minimums**: for each element, find left/right boundaries via monotonic stack.

**Common bugs:**
- Pushing values instead of indices - you lose position info needed for width / distance.
- For histogram, forgetting the sentinel `0` at end - never flushes remaining stack.
- Confusing "next greater" (use decreasing stack) with "previous greater" (use decreasing stack, iterate right-to-left or use stack content differently).

---

## 10. Monotonic Deque (sliding window max/min)

**When to use it:** Need max (or min) in a sliding window of size K over array. Single pass O(n).

**Trigger phrases:** "sliding window maximum", "sliding window minimum", "shortest subarray with sum at least K".

**Anchor problem:** Sliding Window Maximum.

**Template (max in window of size K):**
```java
Deque<Integer> dq = new ArrayDeque<>();          // indices, decreasing values from front
int[] result = new int[n - k + 1];
for (int i = 0; i < n; i++) {
    // remove indices that fell out of window
    while (!dq.isEmpty() && dq.peekFirst() <= i - k) dq.pollFirst();
    // remove indices whose values are smaller than current (they'll never be max)
    while (!dq.isEmpty() && arr[dq.peekLast()] < arr[i]) dq.pollLast();
    dq.offerLast(i);
    if (i >= k - 1) result[i - k + 1] = arr[dq.peekFirst()];
}
return result;
```

**Variations:**
- **Sliding Window Min**: invert comparison (`arr[dq.peekLast()] > arr[i]`).
- **Shortest Subarray with Sum at Least K** (allows negatives): monotonic deque on prefix sums.

**Common bugs:**
- Confusing stack (only one end) with deque (both ends needed for window expiry).
- For window size K, expiry check uses `<= i - k`, not `< i - k`.

---

## 11. Sorting Tricks

**When to use it:** Whenever sorting reduces the problem to a linear sweep, OR you need a custom comparator for greedy reasoning.

**Trigger phrases:** "merge intervals", "meeting rooms", "minimum platforms", "non-overlapping intervals", "task scheduling", "largest number from array".

**Anchor problem:** Merge Intervals.

**Template (Merge Intervals):**
```java
Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]));
List<int[]> merged = new ArrayList<>();
for (int[] cur : intervals) {
    if (merged.isEmpty() || merged.get(merged.size() - 1)[1] < cur[0]) {
        merged.add(cur);
    } else {
        merged.get(merged.size() - 1)[1] = Math.max(merged.get(merged.size() - 1)[1], cur[1]);
    }
}
return merged.toArray(new int[0][]);
```

**Template (Custom Comparator - Largest Number):**
```java
String[] strs = Arrays.stream(arr).mapToObj(String::valueOf).toArray(String[]::new);
Arrays.sort(strs, (a, b) -> (b + a).compareTo(a + b));    // 9 before 30 because "930" > "309"
if (strs[0].equals("0")) return "0";
return String.join("", strs);
```

**Variations:**
- **Meeting Rooms II** (min rooms needed): sort by start; min-heap of end times; pop while top end <= cur start.
- **Non-overlapping Intervals** (min removals): sort by END time; greedy keep next that doesn't overlap.
- **Minimum Number of Arrows to Burst Balloons**: sort by END; arrow at each new endpoint.

**Common bugs:**
- Forgetting to handle empty input.
- Sorting by START vs END changes the algorithm - mix them up and the greedy fails.
- Using `a[0] - b[0]` in comparator with values that may differ by >= `Integer.MAX_VALUE` - use `Integer.compare(a[0], b[0])`.

---

## 12. Hashing on Arrays

**When to use it:** Lookup / dedup / frequency in O(1). Solves many "two sum"-family problems in O(n).

**Trigger phrases:** "two sum", "longest consecutive sequence", "happy number", "contains duplicate", "first unique element".

**Anchor problem:** Two Sum.

**Template (Two Sum):**
```java
Map<Integer, Integer> seen = new HashMap<>();
for (int i = 0; i < n; i++) {
    int need = target - arr[i];
    if (seen.containsKey(need)) return new int[]{seen.get(need), i};
    seen.put(arr[i], i);
}
return new int[]{-1, -1};
```

**Template (Longest Consecutive Sequence):**
```java
Set<Integer> set = new HashSet<>();
for (int x : arr) set.add(x);
int best = 0;
for (int x : set) {
    if (!set.contains(x - 1)) {              // x is start of a run
        int len = 1;
        while (set.contains(x + len)) len++;
        best = Math.max(best, len);
    }
}
return best;
```

**Variations:**
- **Contains Duplicate within K Distance**: sliding window of size K + HashSet.
- **Subarray Sum Equals K** (covered in prefix sum).
- **Count Number of Nice Subarrays** (exactly K odd numbers): same prefix-count idea.

**Common bugs:**
- Putting value in map BEFORE checking complement - same element matches itself.
- For longest-consecutive, NOT checking `!set.contains(x - 1)` first - turns O(n) into O(n^2).

---

## 13. Quickselect / Partition (Kth element)

**When to use it:** Find Kth largest / smallest element, OR top-K - without sorting the whole array. Average O(n).

**Trigger phrases:** "Kth largest element", "Kth smallest element", "top K frequent", "median of array".

**Anchor problem:** Kth Largest Element in an Array.

**Template (Quickselect - average O(n)):**
```java
public int findKthLargest(int[] arr, int k) {
    return quickselect(arr, 0, arr.length - 1, arr.length - k);    // Kth largest = (n-k)th smallest
}

private int quickselect(int[] arr, int lo, int hi, int targetIdx) {
    if (lo == hi) return arr[lo];
    int pivotIdx = partition(arr, lo, hi);
    if (pivotIdx == targetIdx) return arr[pivotIdx];
    if (targetIdx < pivotIdx) return quickselect(arr, lo, pivotIdx - 1, targetIdx);
    return quickselect(arr, pivotIdx + 1, hi, targetIdx);
}

private int partition(int[] arr, int lo, int hi) {              // Lomuto, last-element pivot
    int pivot = arr[hi], i = lo;
    for (int j = lo; j < hi; j++) {
        if (arr[j] < pivot) swap(arr, i++, j);
    }
    swap(arr, i, hi);
    return i;
}
```

**Alternative (Heap - O(n log k)):**
```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>(k);
for (int x : arr) {
    minHeap.offer(x);
    if (minHeap.size() > k) minHeap.poll();
}
return minHeap.peek();
```

**Variations:**
- **Top K Frequent Elements**: bucket sort (by frequency) OR min-heap of size K.
- **Median from Data Stream**: two heaps (max-heap of lower half, min-heap of upper half), keep balanced.

**Common bugs:**
- Quickselect worst case O(n^2) on already-sorted input with first/last pivot. Use random pivot or median-of-three.
- For Kth LARGEST in heap: use a MIN-heap of size K (keep top K, pop smallest when overflow). The opposite for Kth smallest.

---

## 14. Matrix Patterns

**When to use it:** 2D array transformations - rotate, spiral, set zeros, search 2D matrix, etc.

**Trigger phrases:** "rotate image 90 degrees", "spiral matrix", "set matrix zeros", "search 2D matrix", "word search", "diagonal traverse".

**Anchor problem:** Rotate Image.

**Template (Rotate Image 90 CW - transpose + reverse each row):**
```java
int n = matrix.length;
// transpose
for (int i = 0; i < n; i++) {
    for (int j = i + 1; j < n; j++) {
        int t = matrix[i][j]; matrix[i][j] = matrix[j][i]; matrix[j][i] = t;
    }
}
// reverse each row
for (int[] row : matrix) {
    for (int l = 0, r = n - 1; l < r; l++, r--) {
        int t = row[l]; row[l] = row[r]; row[r] = t;
    }
}
```
For 90 CCW: reverse each row FIRST, then transpose. (Or transpose then reverse each column.)

**Template (Spiral Matrix):**
```java
List<Integer> out = new ArrayList<>();
int top = 0, bot = m - 1, left = 0, right = n - 1;
while (top <= bot && left <= right) {
    for (int j = left; j <= right; j++) out.add(matrix[top][j]);
    top++;
    for (int i = top; i <= bot; i++) out.add(matrix[i][right]);
    right--;
    if (top <= bot) {
        for (int j = right; j >= left; j--) out.add(matrix[bot][j]);
        bot--;
    }
    if (left <= right) {
        for (int i = bot; i >= top; i--) out.add(matrix[i][left]);
        left++;
    }
}
```

**Template (Set Matrix Zeros - O(1) space):**
```java
// Use first row and first column as markers
boolean firstRowHasZero = false, firstColHasZero = false;
for (int j = 0; j < n; j++) if (matrix[0][j] == 0) firstRowHasZero = true;
for (int i = 0; i < m; i++) if (matrix[i][0] == 0) firstColHasZero = true;
for (int i = 1; i < m; i++) {
    for (int j = 1; j < n; j++) {
        if (matrix[i][j] == 0) { matrix[i][0] = 0; matrix[0][j] = 0; }
    }
}
for (int i = 1; i < m; i++) {
    for (int j = 1; j < n; j++) {
        if (matrix[i][0] == 0 || matrix[0][j] == 0) matrix[i][j] = 0;
    }
}
if (firstRowHasZero) for (int j = 0; j < n; j++) matrix[0][j] = 0;
if (firstColHasZero) for (int i = 0; i < m; i++) matrix[i][0] = 0;
```

**Variations:**
- **Search 2D Matrix** (rows sorted, first of each row > last of prev): treat as 1D sorted, binary search.
- **Search 2D Matrix II** (rows + cols sorted): start at top-right, move left if larger, down if smaller.
- **Word Search**: backtracking with visited marker (or mark in place with sentinel).

**Common bugs:**
- For rotate, mixing transpose direction (i, j) vs (j, i).
- For spiral, forgetting `if (top <= bot)` and `if (left <= right)` guards after shrinking - double-counts the middle row/col.
- For set zeros, the in-place marker approach requires careful order - mark first, then zero, finally zero out the marker row/col.

---

## 15 (advanced). Segment Tree / Fenwick Tree

**When to use it:** Range queries (sum, min, max, GCD, etc.) on an array that ALSO supports point or range updates - O(log n) for both.

**Trigger phrases:** "range sum query mutable", "count of smaller numbers after self", "range minimum query with updates", "interval / range update / query".

**Anchor problem:** Range Sum Query - Mutable.

**Template (Fenwick / Binary Indexed Tree for sum):**
```java
class Fenwick {
    int[] bit;
    Fenwick(int n) { bit = new int[n + 1]; }

    void update(int i, int delta) {                      // i is 1-indexed
        for (; i < bit.length; i += i & -i) bit[i] += delta;
    }
    int prefix(int i) {                                  // sum of [1..i], 1-indexed
        int s = 0;
        for (; i > 0; i -= i & -i) s += bit[i];
        return s;
    }
    int rangeSum(int l, int r) { return prefix(r) - prefix(l - 1); }
}
```

**Template (Segment Tree - array-based):**
```java
int[] tree;
int n;

void build(int[] arr) {
    n = arr.length;
    tree = new int[4 * n];
    build(arr, 0, 0, n - 1);
}
void build(int[] arr, int node, int l, int r) {
    if (l == r) { tree[node] = arr[l]; return; }
    int mid = (l + r) >>> 1;
    build(arr, 2 * node + 1, l, mid);
    build(arr, 2 * node + 2, mid + 1, r);
    tree[node] = tree[2 * node + 1] + tree[2 * node + 2];
}
int query(int node, int l, int r, int ql, int qr) {
    if (qr < l || r < ql) return 0;                      // disjoint
    if (ql <= l && r <= qr) return tree[node];           // fully covered
    int mid = (l + r) >>> 1;
    return query(2 * node + 1, l, mid, ql, qr)
         + query(2 * node + 2, mid + 1, r, ql, qr);
}
```

**Variations:**
- **Count of Smaller Numbers After Self**: Fenwick indexed by value (after rank compression).
- **Range Update Range Query**: lazy propagation on segment tree.
- **2D Fenwick**: nested loops for 2D range sum with point updates.

**Common bugs:**
- Fenwick is 1-indexed; off-by-one errors are easy.
- Segment tree size: `4 * n` is the safe upper bound; smaller can overflow on non-power-of-2 sizes.
- Forgetting lazy propagation when updates are range-based.

---

## Suggested revision order if relearning from scratch

1. **Two Pointers (opposite + same direction)** - quick wins; warmup.
2. **Sliding Window** - sister of two pointers, used everywhere.
3. **Kadane's Algorithm** - one of the most elegant DP-on-array; short to memorize.
4. **Prefix Sum (1D + the SubarraySum=K idea)** - unlocks a huge family.
5. **Sorting + Greedy / Custom Comparator** - intervals, meeting rooms, largest number.
6. **Hashing** - two-sum family, longest-consecutive, dedup.
7. **Binary Search (sorted + rotated + on answer)** - all three variants.
8. **Cyclic Sort** - niche but elegant; few problems, high reward.
9. **Monotonic Stack** - next greater / histogram / trap rain water.
10. **Monotonic Deque** - sliding window max.
11. **Quickselect** - Kth element in average O(n).
12. **Matrix Patterns** - rotate, spiral, set zeros.
13. **Segment / Fenwick Tree** - advanced; only if interview signals it.

**Universal traps across every pattern:**
- **Integer overflow** when summing large arrays - use `long` for accumulators.
- **`Arrays.sort(int[])` has NO Comparator overload** - cannot sort descending directly; box to `Integer[]` first.
- **Modifying array while iterating** - keep a separate `write` index instead.
- **`int[]` vs `Integer[]`** - `Arrays.asList(intArr)` gives `List<int[]>` (one-element list of the array!). Use `Arrays.stream(intArr).boxed().toList()`.
- **Mixing 0-indexed array with 1-indexed math** (e.g., for Kth) - pick one convention and stick to it.
- **Sliding window with negative numbers** for "sum >= target" - doesn't work. Use prefix sum + monotonic deque or sorted structure.
- **Off-by-one in binary search bounds** - decide upfront whether `r = n` or `r = n - 1`, and whether the loop is `l < r` or `l <= r`. Be consistent within one function.
