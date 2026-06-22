# Two Pointers - Cheat Sheet

**Two-pointers is CROSS-CUTTING** - shows up in arrays, strings, linked lists, sliding windows. This is the synthesis sheet; jump to `arrays.md` / `strings.md` / `linked-lists.md` for deeper coverage.

**Decision fork:** Sorted, pair/triplet with target -> CONVERGING (opposite ends). In-place modify (remove / move / partition) -> SAME-DIRECTION (slow + fast write). Linked list cycle / middle / Nth from end -> SLOW/FAST. Palindrome check -> CONVERGING. Merge / intersect two sorted -> TWO SEQUENCES (one ptr per). Sort 0/1/2 -> THREE POINTERS (Dutch flag). Substring with property -> sliding window (special case of two-pointer). 3-Sum / 4-Sum -> outer loop + CONVERGING on rest.

## The 4 skeletons that cover everything

```java
// 1. CONVERGING (opposite ends, sorted)
int l = 0, r = n - 1;
while (l < r) {
    if (good(arr[l], arr[r])) return ...;
    else if (need_bigger) l++;
    else r--;
}

// 2. SAME-DIRECTION (slow writes, fast reads)
int slow = 0;
for (int fast = 0; fast < n; fast++) {
    if (keep(arr[fast])) arr[slow++] = arr[fast];
}

// 3. SLOW/FAST (different speeds for linked list)
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
}

// 4. TWO SEQUENCES (one pointer per array/list)
int i = 0, j = 0;
while (i < a.length && j < b.length) {
    if (a[i] == b[j]) { ... i++; j++; }
    else if (a[i] < b[j]) i++;
    else j++;
}
```

## Two Sum II (Sorted) - Converging
```java
int l = 0, r = n - 1;
while (l < r) {
    int sum = arr[l] + arr[r];
    if (sum == target) return new int[]{l, r};
    else if (sum < target) l++; else r--;
}
```

## Container With Most Water
```java
int l = 0, r = n - 1, best = 0;
while (l < r) {
    best = Math.max(best, Math.min(height[l], height[r]) * (r - l));
    if (height[l] < height[r]) l++; else r--;       // move the SHORTER side
}
```
Moving the taller side can only decrease area; moving shorter MAY improve.

## Trap Rain Water (two-pointer variant)
```java
int l = 0, r = n - 1, leftMax = 0, rightMax = 0, water = 0;
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
```
The SHORTER side determines current cell's water level.

## Valid Palindrome (skip non-alphanumeric)
```java
int l = 0, r = s.length() - 1;
while (l < r) {
    while (l < r && !Character.isLetterOrDigit(s.charAt(l))) l++;
    while (l < r && !Character.isLetterOrDigit(s.charAt(r))) r--;
    if (Character.toLowerCase(s.charAt(l)) != Character.toLowerCase(s.charAt(r))) return false;
    l++; r--;
}
```
Inner skip loops MUST have `l < r` guard.

## Remove Duplicates from Sorted Array (Same-direction)
```java
int slow = 0;
for (int fast = 0; fast < n; fast++) {
    if (slow == 0 || arr[fast] != arr[slow - 1]) arr[slow++] = arr[fast];
}
return slow;
```

## Move Zeros (Same-direction)
```java
int slow = 0;
for (int fast = 0; fast < n; fast++)
    if (arr[fast] != 0) arr[slow++] = arr[fast];
while (slow < n) arr[slow++] = 0;
```

## Sort Colors / Dutch Flag (Three Pointers)
```java
int low = 0, mid = 0, high = n - 1;
while (mid <= high) {
    if (arr[mid] == 0) swap(arr, low++, mid++);
    else if (arr[mid] == 1) mid++;
    else swap(arr, mid, high--);     // do NOT increment mid - swapped-in unprocessed
}
```

## Floyd's Cycle Detection (Linked List)
```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow == fast) return true;
}
return false;
```
Need BOTH guards (`fast != null && fast.next != null`) to avoid NPE on `fast.next.next`.

## Floyd's Find Cycle Start (Phase 2)
```java
// after slow == fast in detection:
slow = head;
while (slow != fast) { slow = slow.next; fast = fast.next; }
return slow;     // cycle start
```

## Find Middle of Linked List
```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next; fast = fast.next.next;
}
return slow;     // even count: SECOND middle. For first-middle: fast = head.next.
```

## Remove Nth from End (gap = n + 1)
```java
ListNode dummy = new ListNode(0, head), first = dummy, second = dummy;
for (int i = 0; i < n + 1; i++) first = first.next;
while (first != null) { first = first.next; second = second.next; }
second.next = second.next.next;
return dummy.next;
```
Gap is n+1 (NOT n) - lands `second` at node BEFORE the one to remove.

## Merge Sorted Array (in-place, nums1 has capacity)
```java
int i = m - 1, j = n - 1, write = m + n - 1;
while (i >= 0 && j >= 0) {
    nums1[write--] = (nums1[i] >= nums2[j]) ? nums1[i--] : nums2[j--];
}
while (j >= 0) nums1[write--] = nums2[j--];     // i's leftovers already in place
```
Always merge from BACK to avoid overwriting unprocessed elements.

## Is Subsequence
```java
int i = 0, j = 0;
while (i < s.length() && j < t.length()) {
    if (s.charAt(i) == t.charAt(j)) i++;
    j++;
}
return i == s.length();
```

## 3-Sum (sort + converging on rest)
```java
Arrays.sort(nums);
for (int i = 0; i < nums.length - 2; i++) {
    if (i > 0 && nums[i] == nums[i-1]) continue;            // skip dup i
    int l = i + 1, r = nums.length - 1;
    while (l < r) {
        int sum = nums[i] + nums[l] + nums[r];
        if (sum == 0) {
            out.add(List.of(nums[i], nums[l], nums[r]));
            while (l < r && nums[l] == nums[l+1]) l++;       // skip dup l
            while (l < r && nums[r] == nums[r-1]) r--;       // skip dup r
            l++; r--;
        } else if (sum < 0) l++; else r--;
    }
}
```
Three dup-skips required (i, l, r) to avoid duplicate triplets in output.

## Reverse String (in place - char[])
```java
int l = 0, r = s.length - 1;
while (l < r) { char t = s[l]; s[l] = s[r]; s[r] = t; l++; r--; }
```
String is immutable in Java - must use char[].

## Reverse Vowels (skip non-vowels)
```java
char[] arr = s.toCharArray();
int l = 0, r = arr.length - 1;
String vowels = "aeiouAEIOU";
while (l < r) {
    while (l < r && vowels.indexOf(arr[l]) < 0) l++;
    while (l < r && vowels.indexOf(arr[r]) < 0) r--;
    char t = arr[l]; arr[l] = arr[r]; arr[r] = t;
    l++; r--;
}
```

## Variable-Window Sliding (Longest Substring Without Repeating)
```java
int l = 0, best = 0;
Map<Character, Integer> freq = new HashMap<>();
for (int r = 0; r < s.length(); r++) {
    freq.merge(s.charAt(r), 1, Integer::sum);
    while (freq.size() < r - l + 1 || hasDuplicate(freq)) {
        if (freq.merge(s.charAt(l), -1, Integer::sum) == 0) freq.remove(s.charAt(l));
        l++;
    }
    best = Math.max(best, r - l + 1);
}
```

## Cross-Reference Map
- Converging / palindrome / 3-Sum / trap rain water -> arrays.md / strings.md
- Same-direction / Dutch flag -> arrays.md
- Slow/fast cycle + middle + gap-N -> linked-lists.md
- Sliding window patterns -> arrays.md, strings.md, hashmap-patterns.md

## Universal traps
- **Inner-loop guards**: `while (l < r && ...)` - the `l < r` is MANDATORY or you walk off the array.
- **Sorted input required** for converging patterns - sort first or comparison logic breaks.
- **Slow vs fast role confusion**: slow = WRITE index (or half-speed pointer). Don't swap them.
- **Off-by-one in window size**: fixed window K needs `r >= k - 1` before recording, not `r >= k`.
- **Dutch flag**: after swapping with high, do NOT increment mid - swapped-in element is unprocessed.
- **3-Sum dup-skip**: needed in THREE places (i, l, r). Skipping just one gives duplicate triplets.
- **Sliding window with negatives**: "shrink while sum >= K" breaks - sums no longer monotonic in r.
- **Linked list slow/fast**: both `fast != null && fast.next != null` needed before `fast.next.next`.
- **In-place merge**: always from the BACK to avoid overwriting unprocessed elements.
- **`String` is immutable** - convert to `char[]` for in-place reverse / swap.
