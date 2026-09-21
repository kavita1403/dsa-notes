# Two Pointers - Master Reference Note

> **Note:** Two-pointers is a CROSS-CUTTING technique - it shows up in arrays, strings, linked lists, sliding windows. This file synthesizes ALL flavors in one place. For deeper coverage of specific applications, see `arrays.md`, `strings.md`, and `linked-lists.md`.

```java
// All two-pointer patterns boil down to one of these four skeletons:

// 1. CONVERGING (start at both ends, move toward middle)
int l = 0, r = n - 1;
while (l < r) { ... l++; r--; }

// 2. SAME-DIRECTION (one writes, one reads)
int slow = 0;
for (int fast = 0; fast < n; fast++) { ... arr[slow++] = arr[fast]; }

// 3. SLOW/FAST (different speeds - usually 1 vs 2)
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) { slow = slow.next; fast = fast.next.next; }

// 4. TWO SEQUENCES (one pointer per array)
int i = 0, j = 0;
while (i < a.length && j < b.length) { ... }
```

---

## Decision Fork (ask yourself this before writing any code)

> "Sorted array, find a pair / triplet with target" -> **Converging (opposite ends)**. Sort first if not sorted.
>
> "Modify array in place: remove / move / partition" -> **Same-direction (slow + fast)**. `slow` is the write index.
>
> "Linked list: cycle, middle, Nth from end" -> **Slow/fast** (different speeds OR gap of K).
>
> "Palindrome check (string or array)" -> **Converging**, compare ends.
>
> "Merge two sorted arrays / lists / find intersection" -> **Two sequences**, one pointer per.
>
> "Container with most water / trap rain water" -> **Converging**, decide based on height comparison.
>
> "Sort colors / Dutch national flag (3 buckets)" -> **Three pointers**, same-direction with a twist.
>
> "Sliding window with grows/shrinks" -> Technically a two-pointer pattern (`l`, `r`). See arrays.md / strings.md for the dedicated window patterns.
>
> "3-Sum, 4-Sum" -> Fix outer index, **converging** on the rest.

Everything else in two-pointer-land is a variant or combination of these.

---

## Quick Lookup Table (read this first when revisiting after a gap)

| # | Flavor | Trigger phrases | Anchor problem | Core idea |
|---|---|---|---|---|
| 1 | Converging (opposite ends) | "pair with target sum (sorted)", "palindrome", "container with most water", "trap rain water" | Two Sum II - Sorted | Sort + `l`, `r` converge; move based on comparison |
| 2 | Same-direction (slow + fast write) | "remove duplicates in place", "move zeros", "partition by predicate" | Remove Duplicates from Sorted Array | `slow` is write index, `fast` reads; copy when condition met |
| 3 | Three pointers (Dutch flag) | "sort colors (0/1/2)", "partition into three buckets" | Sort Colors | `low`, `mid`, `high`; swap based on `arr[mid]` value |
| 4 | Slow/fast (cycle detection - Floyd's) | "detect cycle", "find cycle start", "happy number", "find duplicate" | Linked List Cycle | Slow moves 1, fast moves 2; meet = cycle |
| 5 | Slow/fast (find middle) | "middle of linked list", "find Kth node" | Middle of Linked List | Slow 1 step, fast 2 steps; when fast ends, slow is at middle |
| 6 | Two-pointer with gap (Nth from end) | "remove Nth from end", "Kth from last" | Remove Nth Node From End | First advances N+1; then both move together |
| 7 | Two sequences (merge / intersect) | "merge two sorted arrays / lists", "intersection of two arrays", "is subsequence" | Merge Sorted Array | One pointer per sequence; advance smaller / both on match |
| 8 | Fixed-window sliding | "max sum of size K subarray", "permutation in string" | Maximum Sum Subarray of Size K | `l`, `r` move together at fixed offset |
| 9 | Variable-window sliding | "longest substring with at most K distinct", "min window substring", "shortest with sum >= K" | Longest Substring Without Repeating Characters | `r` expands; `l` shrinks while invariant broken |
| 10 | 3-Sum / 4-Sum (outer loop + converging) | "3-sum", "4-sum", "3-sum closest" | 3-Sum | Sort; fix outer index; converge on the rest. Skip duplicates carefully. |
| 11 | Reverse / In-place Swap | "reverse string", "reverse array in place", "reverse vowels only" | Reverse String | `l` and `r` swap and converge |

---

## 1. Converging Pointers (Opposite Ends)

**When to use it:** Sorted array (or string) where the answer depends on comparing two elements from opposite ends. Move based on whether you need a larger or smaller candidate.

**Trigger phrases:** "two sum sorted", "pair with target sum (sorted)", "valid palindrome", "container with most water", "trap rain water", "is rotation", "reverse vowels".

**Anchor problem:** Two Sum II - Input Array is Sorted.

**Template:**
```java
int l = 0, r = n - 1;
while (l < r) {
    int cur = combine(arr[l], arr[r]);          // sum, height-based area, etc.
    if (cur == target) return new int[]{l, r};
    else if (cur < target) l++;                  // need bigger -> move left up
    else r--;                                     // need smaller -> move right down
}
```

**Template (Container With Most Water):**
```java
int l = 0, r = n - 1, best = 0;
while (l < r) {
    int area = Math.min(height[l], height[r]) * (r - l);
    best = Math.max(best, area);
    if (height[l] < height[r]) l++;              // move the shorter side
    else r--;
}
return best;
```
**Intuition:** moving the TALLER side can only decrease (or equal) area; moving the shorter side at least has a chance of improvement.

**Template (Trapping Rain Water - two-pointer variant):**
```java
int l = 0, r = n - 1;
int leftMax = 0, rightMax = 0, water = 0;
while (l < r) {
    if (height[l] < height[r]) {
        leftMax = Math.max(leftMax, height[l]);
        water += leftMax - height[l];
        l++;
    } else {
        rightMax = Math.max(rightMax, height[r]);
        water += rightMax - height[r];
        r--;
    }
}
return water;
```
**Intuition:** the side with smaller height determines the current cell's water level (limited by the smaller wall).

**Template (Valid Palindrome - skip non-alphanumeric):**
```java
int l = 0, r = s.length() - 1;
while (l < r) {
    while (l < r && !Character.isLetterOrDigit(s.charAt(l))) l++;
    while (l < r && !Character.isLetterOrDigit(s.charAt(r))) r--;
    if (Character.toLowerCase(s.charAt(l)) != Character.toLowerCase(s.charAt(r))) return false;
    l++; r--;
}
return true;
```

**Common bugs:**
- Forgetting to sort first when input isn't sorted - the comparison logic breaks.
- Missing the `l < r` guard in inner skip loops - walks off the array.
- For container/trap-water: moving the WRONG side (tallest instead of shortest) gives wrong answer.

**Example problems:**
- [Two Sum II - Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/)
- [Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)
- [Container With Most Water](https://leetcode.com/problems/container-with-most-water/)
- [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)
- [Squares of a Sorted Array](https://leetcode.com/problems/squares-of-a-sorted-array/)

---

## 2. Same-Direction (slow + fast write)

**When to use it:** Modify an array in place where some elements need to be kept and others removed/shifted. `slow` is the WRITE index; `fast` walks the input.

**Trigger phrases:** "remove duplicates in place", "remove element", "move zeros", "remove duplicates allow up to K", "partition by predicate".

**Anchor problem:** Remove Duplicates from Sorted Array.

**Template:**
```java
int slow = 0;
for (int fast = 0; fast < n; fast++) {
    if (keep(arr[fast])) {
        arr[slow++] = arr[fast];
    }
}
return slow;                                     // new logical length
```

**Template (Move Zeros - keep non-zeros, then fill zeros at end):**
```java
int slow = 0;
for (int fast = 0; fast < n; fast++) {
    if (arr[fast] != 0) arr[slow++] = arr[fast];
}
while (slow < n) arr[slow++] = 0;
```

**Template (Remove Duplicates, allow up to 2 occurrences):**
```java
int slow = 0;
for (int fast = 0; fast < n; fast++) {
    if (slow < 2 || arr[fast] != arr[slow - 2]) {
        arr[slow++] = arr[fast];
    }
}
return slow;
```

**Variations:**
- **Remove Element**: keep if `arr[fast] != val`.
- **Remove Duplicates from Sorted Array**: keep if `slow == 0 || arr[fast] != arr[slow - 1]`.
- **Partition Array by Predicate**: keep if predicate true; everything kept comes first.

**Common bugs:**
- Confusing slow/fast roles - `slow` writes, `fast` reads. NEVER swap.
- For Move Zeros: forgetting to fill remaining with zeros if the problem says output length must match input.
- Iterating with `slow` instead of `fast` in the for-loop - O(n^2) instead of O(n).

**Example problems:**
- [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)
- [Remove Element](https://leetcode.com/problems/remove-element/)
- [Move Zeroes](https://leetcode.com/problems/move-zeroes/)
- [Remove Duplicates from Sorted Array II](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/)
- [Sort Array By Parity](https://leetcode.com/problems/sort-array-by-parity/)

---

## 3. Three Pointers (Dutch National Flag)

**When to use it:** Partition into THREE buckets in one pass: less-than-pivot, equal-to-pivot, greater-than-pivot. Classic for Sort Colors (0/1/2).

**Trigger phrases:** "sort colors (0, 1, 2)", "dutch national flag", "partition into three groups".

**Anchor problem:** Sort Colors.

**Template:**
```java
int low = 0, mid = 0, high = n - 1;
while (mid <= high) {
    if (arr[mid] == 0) {
        swap(arr, low++, mid++);
    } else if (arr[mid] == 1) {
        mid++;
    } else {                                      // arr[mid] == 2
        swap(arr, mid, high--);
        // do NOT increment mid - the swapped-in value hasn't been processed
    }
}
```
**Invariant maintained:** 
- `arr[0..low-1]` = all 0s
- `arr[low..mid-1]` = all 1s
- `arr[mid..high]` = unknown
- `arr[high+1..n-1]` = all 2s

**Variations:**
- **Wiggle Sort II**: more complex three-way partition with sorted-by-magnitude requirement.
- **Find K Closest Elements**: variant with distance-based comparison.

**Common bugs:**
- Incrementing `mid` after swapping with `high` - skips an unprocessed element.
- Using `mid < high` instead of `mid <= high` - off-by-one, misses last unprocessed element.

**Example problems:**
- [Sort Colors](https://leetcode.com/problems/sort-colors/)
- [Wiggle Sort II](https://leetcode.com/problems/wiggle-sort-ii/)
- [Partition Array According to Given Pivot](https://leetcode.com/problems/partition-array-according-to-given-pivot/)
- [Sort Array By Parity II](https://leetcode.com/problems/sort-array-by-parity-ii/)
- [Sort Array By Parity](https://leetcode.com/problems/sort-array-by-parity/)

---

## 4. Slow/Fast (Cycle Detection - Floyd's Tortoise & Hare)

**When to use it:** Detect a cycle in a linked list, OR in any sequence with `next`-like progression (e.g., `nums[nums[i]]` for Find Duplicate Number; sum-of-squares for Happy Number).

**Trigger phrases:** "detect cycle in linked list", "linked list cycle II (find start)", "happy number", "find duplicate number in [1..n]".

**Anchor problem:** Linked List Cycle.

**Template (detect cycle):**
```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow == fast) return true;
}
return false;
```

**Template (find cycle start - Floyd's Phase 2):**
```java
// Phase 1: detect (above)
// Phase 2: reset one pointer to head; walk both at speed 1; they meet at cycle start
slow = head;
while (slow != fast) {
    slow = slow.next;
    fast = fast.next;
}
return slow;
```
**Why phase 2 works:** see `linked-lists.md` pattern 1 for the math. Short version: when slow & fast meet, slow has walked exactly the distance from head to cycle start (mod cycle length).

**Template (Happy Number - cycle on sum-of-squares function):**
```java
public boolean isHappy(int n) {
    int slow = n, fast = next(n);
    while (fast != 1 && slow != fast) {
        slow = next(slow);
        fast = next(next(fast));
    }
    return fast == 1;
}
private int next(int n) {
    int sum = 0;
    while (n > 0) { sum += (n % 10) * (n % 10); n /= 10; }
    return sum;
}
```

**Template (Find Duplicate Number - Floyd's on indices):**
```java
// arr has n+1 elements in [1..n]; treat as linked list where next = arr[cur]
int slow = arr[0], fast = arr[0];
do {
    slow = arr[slow];
    fast = arr[arr[fast]];
} while (slow != fast);
slow = arr[0];
while (slow != fast) {
    slow = arr[slow];
    fast = arr[fast];
}
return slow;
```

**Common bugs:**
- Loop condition for linked list: needs BOTH `fast != null && fast.next != null` to avoid NPE on `fast.next.next`.
- For Find Duplicate Number: starting `slow = fast = 0` works only if values aren't 0. Start from `arr[0]`.

**Example problems:**
- [Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)
- [Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/)
- [Happy Number](https://leetcode.com/problems/happy-number/)
- [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)
- [Circular Array Loop](https://leetcode.com/problems/circular-array-loop/)

---

## 5. Slow/Fast (Find Middle / Halfway Point)

**When to use it:** Find the middle of a linked list (or any sequential structure) in one pass without knowing the length.

**Trigger phrases:** "middle of linked list", "delete middle node", "split list into halves", "find midpoint".

**Anchor problem:** Middle of Linked List.

**Template (returns second middle for even count):**
```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
}
return slow;                         // 1->2->3->4 returns 3 (second middle)
```

**Template (returns first middle for even count):**
```java
ListNode slow = head, fast = head.next;
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
}
return slow;                         // 1->2->3->4 returns 2 (first middle)
```

**Used in:** Palindrome Linked List, Reorder List, Sort List (merge sort splitting).

**Common bugs:**
- Mixing up first vs second middle - even count has TWO middles; pick the correct one for your problem.
- Starting both at head when even count handling matters - shift fast by one to choose first middle.

**Example problems:**
- [Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/)
- [Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/)
- [Reorder List](https://leetcode.com/problems/reorder-list/)
- [Sort List](https://leetcode.com/problems/sort-list/)
- [Delete the Middle Node of a Linked List](https://leetcode.com/problems/delete-the-middle-node-of-a-linked-list/)

---

## 6. Two-Pointer with Gap (Kth from End)

**When to use it:** Find or remove a node at position counted FROM THE END in a single pass. First pointer gets a K-step head start; then both walk together.

**Trigger phrases:** "remove Nth from end", "Kth node from last", "find Nth-to-last".

**Anchor problem:** Remove Nth Node From End of List.

**Template:**
```java
ListNode dummy = new ListNode(0, head);
ListNode first = dummy, second = dummy;
for (int i = 0; i < n + 1; i++) first = first.next;          // gap of n + 1
while (first != null) {
    first = first.next;
    second = second.next;
}
second.next = second.next.next;
return dummy.next;
```

**Why gap of n + 1 (not n)?** We want `second` to land at the node BEFORE the target so we can unlink it.

**Common bugs:**
- Off-by-one on the gap - test mentally with a list of 5 and n = 2.
- Not using dummy - head deletion becomes a special case.

**Example problems:**
- [Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)
- [Swapping Nodes in a Linked List](https://leetcode.com/problems/swapping-nodes-in-a-linked-list/)
- [Rotate List](https://leetcode.com/problems/rotate-list/)
- [Delete the Middle Node of a Linked List](https://leetcode.com/problems/delete-the-middle-node-of-a-linked-list/) (cross-ref Pattern 5)
- [Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/) (cross-ref Pattern 5)

---

## 7. Two Sequences (One Pointer per Array / List)

**When to use it:** Process two sequences in parallel - merge sorted, find intersection, check subsequence, two-sum-IV-on-BSTs.

**Trigger phrases:** "merge two sorted arrays", "merge two sorted lists", "intersection of two arrays", "is subsequence", "longest common prefix".

**Anchor problem:** Merge Sorted Array.

**Template (Merge Two Sorted Arrays - in-place merge of nums1 of capacity m+n):**
```java
int i = m - 1, j = n - 1, write = m + n - 1;
while (i >= 0 && j >= 0) {
    nums1[write--] = (nums1[i] >= nums2[j]) ? nums1[i--] : nums2[j--];
}
while (j >= 0) nums1[write--] = nums2[j--];     // i's leftovers already in place
```
**Trick:** write from the BACK to avoid overwriting unprocessed elements.

**Template (Intersection of Two Arrays - sorted inputs):**
```java
int i = 0, j = 0;
List<Integer> out = new ArrayList<>();
while (i < a.length && j < b.length) {
    if (a[i] == b[j]) {
        if (out.isEmpty() || out.get(out.size() - 1) != a[i]) out.add(a[i]);
        i++; j++;
    } else if (a[i] < b[j]) i++;
    else j++;
}
```

**Template (Is Subsequence - is `s` a subsequence of `t`?):**
```java
int i = 0, j = 0;
while (i < s.length() && j < t.length()) {
    if (s.charAt(i) == t.charAt(j)) i++;
    j++;
}
return i == s.length();
```

**Variations:**
- **Merge Two Sorted Lists**: same template but with linked-list nodes (see `linked-lists.md`).
- **Intersection of Two Sorted Arrays** vs **Two Arrays II** - one returns set, the other allows duplicates.
- **Longest Word in Dictionary Through Deleting**: variant of "is subsequence".

**Common bugs:**
- For in-place merge: starting from the FRONT - overwrites unprocessed elements. Always merge backwards.
- For intersection: forgetting dedup (if input has dupes and you want unique output).

**Example problems:**
- [Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array/)
- [Intersection of Two Arrays](https://leetcode.com/problems/intersection-of-two-arrays/)
- [Intersection of Two Arrays II](https://leetcode.com/problems/intersection-of-two-arrays-ii/)
- [Is Subsequence](https://leetcode.com/problems/is-subsequence/)
- [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)

---

## 8. Fixed-Window Sliding (Two Pointers at Constant Gap)

**When to use it:** Find best (max/min/avg) over a contiguous subarray of fixed length K.

**Trigger phrases:** "max sum subarray of size K", "K-length window with property", "permutation of T in S".

**Anchor problem:** Maximum Sum Subarray of Size K.

**Template:**
```java
int sum = 0, best = 0;
for (int r = 0; r < n; r++) {
    sum += arr[r];
    if (r >= k) sum -= arr[r - k];                // slide left edge
    if (r >= k - 1) best = Math.max(best, sum);
}
return best;
```

**Template (Find All Anagrams in String - fixed-size window with frequency arrays):**
```java
int[] need = new int[26], have = new int[26];
for (char c : p.toCharArray()) need[c - 'a']++;

List<Integer> out = new ArrayList<>();
int k = p.length();
for (int r = 0; r < s.length(); r++) {
    have[s.charAt(r) - 'a']++;
    if (r >= k) have[s.charAt(r - k) - 'a']--;
    if (r >= k - 1 && Arrays.equals(need, have)) out.add(r - k + 1);
}
return out;
```

For more on sliding window, see arrays.md pattern 3 and strings.md pattern 2.

**Example problems:**
- [Maximum Average Subarray I](https://leetcode.com/problems/maximum-average-subarray-i/)
- [Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/)
- [Permutation in String](https://leetcode.com/problems/permutation-in-string/)
- [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/)
- [Repeated DNA Sequences](https://leetcode.com/problems/repeated-dna-sequences/)

---

## 9. Variable-Window Sliding (Two Pointers, Dynamic Gap)

**When to use it:** Find longest/shortest subarray (or substring) satisfying a constraint that changes dynamically.

**Trigger phrases:** "longest substring with at most K distinct", "min window substring", "longest with no repeating", "shortest with sum >= K".

**Anchor problem:** Longest Substring Without Repeating Characters.

**Template:**
```java
int l = 0, best = 0;
Map<Character, Integer> freq = new HashMap<>();
for (int r = 0; r < s.length(); r++) {
    freq.merge(s.charAt(r), 1, Integer::sum);
    while (isInvalid(freq)) {                     // shrink while window violates constraint
        char left = s.charAt(l++);
        if (freq.merge(left, -1, Integer::sum) == 0) freq.remove(left);
    }
    best = Math.max(best, r - l + 1);
}
```

For deeper coverage, see strings.md pattern 2 and arrays.md pattern 3.

**Example problems:**
- [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)
- [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/)
- [Longest Substring with At Most K Distinct Characters](https://leetcode.com/problems/longest-substring-with-at-most-k-distinct-characters/)
- [Fruit Into Baskets](https://leetcode.com/problems/fruit-into-baskets/)
- [Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/)

---

## 10. 3-Sum / 4-Sum (Outer Loop + Converging)

**When to use it:** Find triplets (or quadruplets) summing to target. After sorting, fix outer index(es) and use two-pointer on the rest.

**Trigger phrases:** "3-sum", "3-sum closest", "4-sum", "3-sum smaller", "valid triangle number".

**Anchor problem:** 3-Sum.

**Template (3-Sum, find all unique triplets summing to 0):**
```java
Arrays.sort(nums);
List<List<Integer>> out = new ArrayList<>();
for (int i = 0; i < nums.length - 2; i++) {
    if (i > 0 && nums[i] == nums[i - 1]) continue;            // skip dup outer
    int l = i + 1, r = nums.length - 1;
    int target = -nums[i];
    while (l < r) {
        int sum = nums[l] + nums[r];
        if (sum == target) {
            out.add(List.of(nums[i], nums[l], nums[r]));
            while (l < r && nums[l] == nums[l + 1]) l++;       // skip dup left
            while (l < r && nums[r] == nums[r - 1]) r--;       // skip dup right
            l++; r--;
        } else if (sum < target) l++;
        else r--;
    }
}
return out;
```

**Template (4-Sum):** same skeleton with two outer loops.

**Variations:**
- **3-Sum Closest**: track best diff, no dup-skip needed.
- **3-Sum Smaller**: count, not enumerate - for each pair (i, l, r), count is `r - l` since all between l and r work too.
- **Valid Triangle Number**: sort, fix largest side, two-pointer on smaller.

**Common bugs:**
- Forgetting duplicate skips at THREE places (outer i, inner l, inner r) - get duplicate triplets in output.
- For 3-Sum closest: not updating best when difference is smaller, even if not zero.

**Example problems:**
- [3Sum](https://leetcode.com/problems/3sum/)
- [3Sum Closest](https://leetcode.com/problems/3sum-closest/)
- [4Sum](https://leetcode.com/problems/4sum/)
- [3Sum Smaller](https://leetcode.com/problems/3sum-smaller/)
- [Valid Triangle Number](https://leetcode.com/problems/valid-triangle-number/)

---

## 11. Reverse / In-place Swap

**When to use it:** Reverse a string, array (or part of it) in place.

**Trigger phrases:** "reverse string", "reverse array in place", "reverse vowels only", "reverse only certain characters".

**Anchor problem:** Reverse String.

**Template (Reverse String - char array):**
```java
public void reverseString(char[] s) {
    int l = 0, r = s.length - 1;
    while (l < r) {
        char t = s[l]; s[l] = s[r]; s[r] = t;
        l++; r--;
    }
}
```

**Template (Reverse Only Vowels - skip non-vowels):**
```java
public String reverseVowels(String s) {
    char[] arr = s.toCharArray();
    int l = 0, r = arr.length - 1;
    String vowels = "aeiouAEIOU";
    while (l < r) {
        while (l < r && vowels.indexOf(arr[l]) < 0) l++;
        while (l < r && vowels.indexOf(arr[r]) < 0) r--;
        char t = arr[l]; arr[l] = arr[r]; arr[r] = t;
        l++; r--;
    }
    return new String(arr);
}
```

**Variations:**
- **Rotate Array**: reverse-three trick - reverse whole, then reverse first K, then reverse rest.
- **Reverse Words in a String**: split, reverse word array, join. OR reverse whole string then reverse each word individually.

**Common bugs:**
- `String` is immutable - can't reverse in place. Convert to `char[]` first.
- For "rotate array by K": forgetting `k %= n` - large K wraps around.

**Example problems:**
- [Reverse String](https://leetcode.com/problems/reverse-string/)
- [Reverse Vowels of a String](https://leetcode.com/problems/reverse-vowels-of-a-string/)
- [Rotate Array](https://leetcode.com/problems/rotate-array/)
- [Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string/)
- [Reverse Words in a String III](https://leetcode.com/problems/reverse-words-in-a-string-iii/)

---

## Suggested revision order if relearning from scratch

1. **Converging (opposite ends)** - most intuitive; one pass O(n) on sorted.
2. **Reverse / In-place Swap** - trivial; warmup.
3. **Same-direction (slow + fast write)** - master the write/read distinction.
4. **Slow/fast for cycle detection (Floyd's)** - elegant; understand WHY phase 2 works.
5. **Slow/fast for find middle** - close cousin; pick first vs second middle deliberately.
6. **Two pointers with gap (Nth from end)** - one pass; use dummy.
7. **Two sequences (merge / intersect)** - parallel walking pattern.
8. **Fixed-window sliding** - special case of two-pointer.
9. **Variable-window sliding** - the workhorse for substring problems.
10. **3-Sum / 4-Sum** - apply converging within a loop. Duplicate handling is the hard part.
11. **Three pointers (Dutch flag)** - niche but elegant.

**Universal traps across every two-pointer pattern:**
- **Inner-loop guards**: when skipping in inner loops (`while (l < r && ...)`), the `l < r` check is mandatory or you walk off the array.
- **For sorted input requirements**: many converging patterns NEED sorted input. Sort first, or the comparison logic doesn't work.
- **Slow vs fast role confusion**: `slow` is usually the WRITE index or the half-speed pointer. Be deliberate; don't swap them.
- **Off-by-one in window size**: fixed window of K needs `r >= k - 1` before recording, not `r >= k`.
- **For Dutch flag**: incrementing mid after swapping with high - skips an unprocessed element.
- **For 3-Sum**: forgetting to skip duplicates in 3 places (outer i, inner l, inner r).
- **For sliding window with NEGATIVES**: the "shrink while sum >= K" pattern breaks because sums no longer grow monotonically with r. Use prefix sum + monotonic deque instead.
- **For linked list slow/fast**: BOTH `fast != null && fast.next != null` are needed before `fast.next.next`.
- **In-place merge** (sorted arrays): always merge from the BACK to avoid overwriting unprocessed elements.
- **`String` is immutable** - convert to `char[]` before in-place reverse / swap.

---

## Cross-reference table (where each pattern is also covered in depth)

| Pattern | Deeper file |
|---|---|
| Converging on sorted array | arrays.md pattern 1 |
| Same-direction (in-place write) | arrays.md pattern 2 |
| Three pointers (Dutch flag) | arrays.md pattern 2 (variation) |
| Slow/fast cycle detection | linked-lists.md pattern 1 |
| Slow/fast find middle | linked-lists.md pattern 1 (variation) |
| Two-pointer with gap | linked-lists.md pattern 2 |
| Two sequences merge | linked-lists.md pattern 5 (linked lists), arrays.md (arrays) |
| Fixed-window sliding | arrays.md pattern 3, strings.md pattern 2 |
| Variable-window sliding | strings.md pattern 2, hashmap-patterns.md pattern 7 |
| 3-Sum / 4-Sum | arrays.md pattern 1 (variation) |
| String palindrome check | strings.md pattern 1 |
| Trap rain water (two-ptr variant) | arrays.md pattern 1 (variation) |

Use THIS file when you want the unified picture; jump to the specific file for deeper coverage of that pattern's variations.
