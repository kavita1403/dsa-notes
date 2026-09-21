# Dynamic Programming - Master Reference Note

```java
// The four questions to answer for EVERY DP problem:
// 1. STATE      - what does dp[i] (or dp[i][j]) represent? Define in plain English.
// 2. TRANSITION - how is dp[i] derived from earlier states? Write the recurrence.
// 3. BASE CASE  - what's dp[0] (or dp[0][0])? What's the trivially-known answer?
// 4. ORDER      - in what order do we compute states so dependencies are ready?

// Two implementation flavors:
int dpMemo(int i, int[] memo) {                    // TOP-DOWN (recursion + memo)
    if (memo[i] != -1) return memo[i];
    return memo[i] = base(i) ? value : transition(...);
}
int dpTab(int n) {                                  // BOTTOM-UP (tabulation)
    int[] dp = new int[n + 1];
    dp[0] = base;
    for (int i = 1; i <= n; i++) dp[i] = transition(...);
    return dp[n];
}
```

---

## Decision Fork (ask yourself this before writing any code)

> "Count the number of WAYS to do X" -> **Additive DP** (transitions SUM). e.g., climbing stairs, coin change II.
>
> "MIN / MAX value of X" -> **Min/Max DP** (transitions take min/max). e.g., min path sum, max subarray.
>
> "Can we achieve X?" (yes/no) -> **Boolean DP** (transitions use OR). e.g., partition equal subset sum, word break.
>
> "Pick subset of items with constraint" -> **0/1 Knapsack** (each item once, 2D dp by items + capacity).
>
> "Pick items, each can be reused" -> **Unbounded Knapsack** (1D dp by capacity, iterate items).
>
> "Longest / shortest SUBSEQUENCE" (not contiguous) -> **2D DP on prefix lengths** (LCS family).
>
> "Longest / shortest SUBSTRING" (contiguous) -> **Sliding window OR 1D DP**.
>
> "Split string / array into best partition" -> **1D DP with j < i loop** (decision at each split point).
>
> "State changes over time with limits" (buy/sell, games) -> **State machine DP** (one dp[i] per state).
>
> "Grid path - count or min cost" -> **2D matrix DP** (dp[i][j] from top + left neighbors).
>
> "Largest rectangle / square of all 1s" -> **Region DP** (dp[i][j] = min of three neighbors + 1).
>
> "Burst balloons / matrix chain / merge stones" -> **Interval DP** (dp[i][j] over range, pick split point k).
>
> "Tree problem with optimal substructure" -> **Tree DP** (return tuple per node).
>
> "Subset with bitmask (n <= 20)" -> **Bitmask DP** (dp[mask] = best for that subset).

Everything else in DP-land is a variant or combination of these.

---

## Quick Lookup Table (read this first when revisiting after a gap)

| # | Pattern | Trigger phrases | Anchor problem | Core idea |
|---|---|---|---|---|
| 1 | Linear DP / Fibonacci | "climbing stairs", "ways to reach N", "house robber", "min cost climbing stairs" | Climbing Stairs | dp[i] from dp[i-1], dp[i-2] (or further back) |
| 2 | 0/1 Knapsack | "pick subset for max value within capacity", "partition equal subset sum", "target sum" | Partition Equal Subset Sum | 2D dp[i][cap]: include or skip item i. Iterate capacity in REVERSE for 1D. |
| 3 | Unbounded Knapsack | "coin change (min coins)", "coin change 2 (count ways)", "perfect squares" | Coin Change | 1D dp[amount]: try each coin. Iterate capacity FORWARD. |
| 4 | LIS (Longest Increasing Subseq) | "longest increasing subsequence", "longest bitonic", "russian doll envelopes" | Longest Increasing Subsequence | O(n^2) dp[i] = max(dp[j]) + 1 for j < i. O(n log n) patience sort. |
| 5 | LCS (Longest Common Subseq) | "longest common subsequence", "delete to make equal", "shortest common supersequence" | Longest Common Subsequence | 2D dp[i][j] = dp[i-1][j-1] + 1 on match, else max(dp[i-1][j], dp[i][j-1]) |
| 6 | Edit Distance Family | "edit distance", "regex match", "wildcard match", "distinct subsequences" | Edit Distance | 2D dp[i][j] with 3 transitions: replace, insert, delete |
| 7 | String DP (Word Break / Decode) | "word break", "decode ways", "concatenated words" | Word Break | dp[i] = OR over j<i of (dp[j] AND valid(s[j..i])) |
| 8 | Palindrome DP | "longest palindromic subsequence", "palindrome partitioning II", "count palindromic substrings" | Palindrome Partitioning II | dp[i][j] = palindrome status; or partition cuts dp |
| 9 | Stock Buy/Sell (State Machine) | "best time to buy and sell stock IV", "with cooldown", "with fee" | Best Time to Buy and Sell Stock with Cooldown | State per (day, holdings, transactions); transition by buy/sell/hold |
| 10 | Grid Path DP | "unique paths", "min path sum", "dungeon game", "longest increasing path" | Unique Paths | dp[i][j] from dp[i-1][j] + dp[i][j-1] (or min/max) |
| 11 | Region DP (Histogram / Square) | "maximal rectangle (in 0/1 matrix)", "maximal square", "largest square of 1s" | Maximal Square | dp[i][j] = min(top, left, top-left) + 1 if grid[i][j] == '1' |
| 12 | Interval DP | "burst balloons", "matrix chain multiplication", "remove boxes", "merge stones" | Burst Balloons | dp[i][j] = max over split point k of dp[i][k-1] + dp[k+1][j] + cost(i,k,j) |
| 13 | Tree DP | "house robber III", "binary tree max path sum", "diameter of binary tree" | House Robber III | Return tuple (rob, notRob) per node; combine bottom-up |
| 14 | Bitmask DP | "traveling salesman", "partition to K equal sum subsets", "shortest path visiting all nodes" | Partition to K Equal Sum Subsets | dp[mask] = best for subset; iterate masks 0..(1<<n)-1 |
| 15 | Digit DP | "count numbers with digit constraints", "count integers with no consecutive equal" | (specialized) | dp[pos][tight][state] - count completions of partial number |

---

## 1. Linear DP / Fibonacci Family (1D, dp[i] from earlier indices)

**When to use it:** Problems where the answer at position `i` depends on a small number of earlier positions. The 1D DP foundation.

**Trigger phrases:** "climbing stairs", "min cost climbing stairs", "house robber", "house robber II", "decode ways", "Fibonacci", "tribonacci".

**Anchor problem:** Climbing Stairs.

**Template (Climbing Stairs - count ways to reach N taking 1 or 2 steps):**
```java
public int climbStairs(int n) {
    if (n <= 2) return n;
    int prev2 = 1, prev1 = 2;
    for (int i = 3; i <= n; i++) {
        int cur = prev1 + prev2;
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```
**State:** `dp[i]` = number of ways to reach step `i`.
**Transition:** `dp[i] = dp[i-1] + dp[i-2]` (you came from one below or two below).
**Base:** `dp[1] = 1`, `dp[2] = 2`.
**Space-optimized:** since you only need the last two values, two variables suffice (O(1) space).

**Template (House Robber - max loot, no two adjacent):**
```java
public int rob(int[] nums) {
    int prev2 = 0, prev1 = 0;
    for (int x : nums) {
        int cur = Math.max(prev1, prev2 + x);          // skip OR take current
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```
**State:** `dp[i]` = max loot up to (and considering) house `i`.
**Transition:** `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` - skip or rob current.

**Template (House Robber II - houses in a circle):**
```java
public int rob(int[] nums) {
    if (nums.length == 1) return nums[0];
    return Math.max(robRange(nums, 0, nums.length - 2),    // skip last
                    robRange(nums, 1, nums.length - 1));    // skip first
}
```
Trick: handle the cycle by trying both "skip first" and "skip last".

**Variations:**
- **Min Cost Climbing Stairs**: `dp[i] = cost[i] + min(dp[i-1], dp[i-2])`.
- **Decode Ways**: `dp[i] = (s[i-1] != '0' ? dp[i-1] : 0) + (validTwoDigit(s, i-2, i-1) ? dp[i-2] : 0)`.
- **Tribonacci**: `dp[i] = dp[i-1] + dp[i-2] + dp[i-3]`.
- **Paint House** (1D with K states per i): `dp[i][color]` over color choices.

**Common bugs:**
- Off-by-one between problem's 1-indexed steps and 0-indexed array.
- Not handling `n = 0` or `n = 1` base cases.
- For House Robber II: forgetting that single-element arrays don't fit the "skip first or last" logic.

**Example problems:**
- [Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)
- [House Robber](https://leetcode.com/problems/house-robber/)
- [House Robber II](https://leetcode.com/problems/house-robber-ii/)
- [Min Cost Climbing Stairs](https://leetcode.com/problems/min-cost-climbing-stairs/)
- [Decode Ways](https://leetcode.com/problems/decode-ways/)

---

## 2. 0/1 Knapsack

**When to use it:** Pick a subset of items (each at most once) to maximize value (or hit a target) within a constraint. The defining DP problem.

**Trigger phrases:** "0/1 knapsack", "pick subset for max value within capacity", "partition equal subset sum", "target sum (assign + / -)", "last stone weight II".

**Anchor problem:** Partition Equal Subset Sum.

**Template (2D - explicit items + capacity):**
```java
// dp[i][c] = can we make sum c using first i items?
boolean[][] dp = new boolean[n + 1][target + 1];
for (int i = 0; i <= n; i++) dp[i][0] = true;       // sum 0 always achievable (pick nothing)

for (int i = 1; i <= n; i++) {
    int w = nums[i - 1];
    for (int c = 1; c <= target; c++) {
        dp[i][c] = dp[i - 1][c];                     // skip item
        if (c >= w) dp[i][c] |= dp[i - 1][c - w];    // OR take item
    }
}
return dp[n][target];
```

**Template (1D space-optimized - iterate capacity REVERSE):**
```java
boolean[] dp = new boolean[target + 1];
dp[0] = true;
for (int x : nums) {
    for (int c = target; c >= x; c--) {              // REVERSE - so dp[c-x] is from PREVIOUS iteration
        dp[c] |= dp[c - x];
    }
}
return dp[target];
```
**Why reverse?** If you iterate forward, `dp[c - x]` might already have been updated in this iteration -> you'd be picking the same item twice (which would be unbounded knapsack).

**Variations:**
- **Partition Equal Subset Sum**: target = `totalSum / 2`. If odd sum, return false.
- **Target Sum** (assign + / - to each number to hit target): equivalent to subset-sum where subset = positives, target = `(sum + target) / 2`.
- **Last Stone Weight II**: minimize `|sumA - sumB|` over partitions; equivalent to subset-sum closest to half.
- **0/1 Knapsack Max Value** (items have value + weight): change boolean to int, transition `dp[c] = max(dp[c], dp[c-w] + v)`.

**Common bugs:**
- Iterating capacity FORWARD with 1D dp - turns 0/1 into unbounded.
- For partition: forgetting to check `totalSum % 2 == 0` first.
- Conflating "exact target" vs "max value not exceeding capacity" - different transitions.

**Example problems:**
- [Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/)
- [Target Sum](https://leetcode.com/problems/target-sum/)
- [Last Stone Weight II](https://leetcode.com/problems/last-stone-weight-ii/)
- [Ones and Zeroes](https://leetcode.com/problems/ones-and-zeroes/)
- [Profitable Schemes](https://leetcode.com/problems/profitable-schemes/)

---

## 3. Unbounded Knapsack

**When to use it:** Same as 0/1 knapsack, but each item can be picked any number of times (coins, lengths to cut, etc.).

**Trigger phrases:** "coin change (min coins)", "coin change 2 (count ways)", "perfect squares", "minimum cost to cut rod", "combination sum IV".

**Anchor problem:** Coin Change (min number of coins to make amount).

**Template (Coin Change - MIN coins):**
```java
// dp[i] = min coins to make amount i (Integer.MAX_VALUE = impossible)
int[] dp = new int[amount + 1];
Arrays.fill(dp, amount + 1);                         // sentinel meaning "impossible"
dp[0] = 0;
for (int i = 1; i <= amount; i++) {
    for (int coin : coins) {
        if (i >= coin) dp[i] = Math.min(dp[i], dp[i - coin] + 1);
    }
}
return dp[amount] > amount ? -1 : dp[amount];
```

**Template (Coin Change 2 - COUNT ways):**
```java
// dp[c] = number of ways to make c using available coins
int[] dp = new int[amount + 1];
dp[0] = 1;                                           // one way to make 0: pick nothing
for (int coin : coins) {                              // OUTER: coins
    for (int c = coin; c <= amount; c++) {           // INNER: capacity forward
        dp[c] += dp[c - coin];
    }
}
return dp[amount];
```
**Why this loop order matters:** With coins-outer / capacity-inner, you count COMBINATIONS (each coin used in order). Swapping the loops counts PERMUTATIONS (ordered sequences) - different problem!

**Template (Combination Sum IV - ordered sequences, sums to target):**
```java
int[] dp = new int[target + 1];
dp[0] = 1;
for (int t = 1; t <= target; t++) {                  // OUTER: target
    for (int num : nums) {                            // INNER: numbers
        if (t >= num) dp[t] += dp[t - num];
    }
}
return dp[target];
```

**Variations:**
- **Perfect Squares**: like Coin Change with `coins = [1, 4, 9, ...]`.
- **Minimum Number of Refueling Stops**: variant where you pick at most some count - heap-based usually better.

**Common bugs:**
- Mixing combination (coins-outer) and permutation (target-outer) loop orders.
- Forgetting to initialize dp[0] = 0 for min, or dp[0] = 1 for count.
- Sentinel mistake: using `Integer.MAX_VALUE` directly causes overflow when you add 1. Use `amount + 1` as "impossible" sentinel.

**Example problems:**
- [Coin Change](https://leetcode.com/problems/coin-change/)
- [Coin Change II](https://leetcode.com/problems/coin-change-ii/)
- [Perfect Squares](https://leetcode.com/problems/perfect-squares/)
- [Minimum Cost For Tickets](https://leetcode.com/problems/minimum-cost-for-tickets/)
- [Combination Sum IV](https://leetcode.com/problems/combination-sum-iv/)

---

## 4. LIS - Longest Increasing Subsequence

**When to use it:** Find length (or actual sequence) of the longest STRICTLY INCREASING subsequence (not contiguous). Has both O(n^2) DP and elegant O(n log n) patience-sort variant.

**Trigger phrases:** "longest increasing subsequence", "longest bitonic subsequence", "russian doll envelopes", "number of LIS", "longest chain of pairs".

**Anchor problem:** Longest Increasing Subsequence.

**Template (O(n^2) DP):**
```java
int n = nums.length;
int[] dp = new int[n];               // dp[i] = LIS ending at index i
Arrays.fill(dp, 1);
for (int i = 1; i < n; i++) {
    for (int j = 0; j < i; j++) {
        if (nums[j] < nums[i]) dp[i] = Math.max(dp[i], dp[j] + 1);
    }
}
int best = 0;
for (int v : dp) best = Math.max(best, v);
return best;
```

**Template (O(n log n) patience sort - binary search):**
```java
int[] tails = new int[nums.length];  // tails[i] = smallest tail of an increasing subseq of length i+1
int size = 0;
for (int x : nums) {
    int lo = 0, hi = size;
    while (lo < hi) {                 // binary search for first tails[mid] >= x
        int mid = (lo + hi) >>> 1;
        if (tails[mid] < x) lo = mid + 1;
        else hi = mid;
    }
    tails[lo] = x;
    if (lo == size) size++;
}
return size;
```
**Intuition:** `tails[i]` is the smallest possible tail value among ALL increasing subsequences of length `i+1`. For each new `x`, find the leftmost tail that's `>=` x and replace it (or append if x is larger than all tails). The final `size` is the LIS length. Note: `tails` is NOT the actual LIS, just its length.

**For strictly vs non-strictly:** use `<=` instead of `<` in the binary search for non-strict (allows duplicates).

**Variations:**
- **Russian Doll Envelopes**: sort by width asc, then height desc (so same width doesn't chain). LIS on heights.
- **Longest Bitonic Subsequence**: LIS from left + LIS from right; answer = max(lisLeft[i] + lisRight[i] - 1).
- **Number of LIS**: track both `dp[i]` (length) and `count[i]` (number of LIS ending at i).
- **Longest Chain of Pairs**: sort by second element, greedy pick.

**Common bugs:**
- Returning `dp[n - 1]` instead of `max(dp)` - LIS may end at any index, not necessarily last.
- For O(n log n): using wrong comparison in binary search (`<` vs `<=`) - changes strict vs non-strict.
- For Russian Doll: not sorting by height DESC for tied widths - allows pairs of same width to chain incorrectly.

**Example problems:**
- [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)
- [Russian Doll Envelopes](https://leetcode.com/problems/russian-doll-envelopes/)
- [Number of Longest Increasing Subsequence](https://leetcode.com/problems/number-of-longest-increasing-subsequence/)
- [Maximum Length of Pair Chain](https://leetcode.com/problems/maximum-length-of-pair-chain/)
- [Longest String Chain](https://leetcode.com/problems/longest-string-chain/)

---

## 5. LCS - Longest Common Subsequence

**When to use it:** Compare two sequences and find the longest subsequence (not contiguous) present in both. Foundation for edit distance, shortest common supersequence, and many string DP problems.

**Trigger phrases:** "longest common subsequence", "delete operation for two strings", "shortest common supersequence", "minimum ASCII delete sum".

**Anchor problem:** Longest Common Subsequence.

**Template:**
```java
int m = a.length(), n = b.length();
int[][] dp = new int[m + 1][n + 1];                  // dp[i][j] = LCS of a[0..i-1] and b[0..j-1]
for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        if (a.charAt(i - 1) == b.charAt(j - 1)) {
            dp[i][j] = dp[i - 1][j - 1] + 1;         // match: extend prior LCS
        } else {
            dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);   // skip one char
        }
    }
}
return dp[m][n];
```

**Template (space-optimized - 1D rolling):**
```java
int[] prev = new int[n + 1];
int[] cur = new int[n + 1];
for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        if (a.charAt(i - 1) == b.charAt(j - 1)) cur[j] = prev[j - 1] + 1;
        else cur[j] = Math.max(prev[j], cur[j - 1]);
    }
    int[] tmp = prev; prev = cur; cur = tmp;
}
return prev[n];
```

**Template (reconstruct the actual LCS string):**
```java
// After building dp[][], walk back from dp[m][n]:
StringBuilder sb = new StringBuilder();
int i = m, j = n;
while (i > 0 && j > 0) {
    if (a.charAt(i - 1) == b.charAt(j - 1)) {
        sb.append(a.charAt(i - 1));
        i--; j--;
    } else if (dp[i - 1][j] > dp[i][j - 1]) i--;
    else j--;
}
return sb.reverse().toString();
```

**Variations:**
- **Delete Operation for Two Strings** (min deletions to make equal): `m + n - 2 * LCS(a, b)`.
- **Shortest Common Supersequence (SCS)**: length = `m + n - LCS`. Reconstruct via backtracking.
- **Min ASCII Delete Sum**: same skeleton, weighted by ASCII values.
- **Longest Palindromic Subsequence**: LCS of string vs its reverse.

**Common bugs:**
- Off-by-one: dp is `[m+1][n+1]`, but `charAt(i-1)` indexes the string. Mixing 1-based dp with 0-based string is the #1 LCS bug.
- For 1D rolling: forgetting that `prev[j-1]` (top-left in 2D) is overwritten - need a temp variable if using a single 1D array.
- For reconstruct: walking backwards in wrong order (need to reverse the StringBuilder).

**Example problems:**
- [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/)
- [Delete Operation for Two Strings](https://leetcode.com/problems/delete-operation-for-two-strings/)
- [Shortest Common Supersequence](https://leetcode.com/problems/shortest-common-supersequence/)
- [Minimum ASCII Delete Sum for Two Strings](https://leetcode.com/problems/minimum-ascii-delete-sum-for-two-strings/)
- [Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/) (LCS of string and its reverse)

---

## 6. Edit Distance Family

**When to use it:** Transform string A into B with min operations (insert / delete / replace). Or count distinct subsequences, or match with regex / wildcard.

**Trigger phrases:** "edit distance", "minimum operations to convert", "regex match `.` `*`", "wildcard match `?` `*`", "distinct subsequences", "interleaving string", "one edit distance".

**Anchor problem:** Edit Distance.

**Template (Edit Distance):**
```java
int m = word1.length(), n = word2.length();
int[][] dp = new int[m + 1][n + 1];
for (int i = 0; i <= m; i++) dp[i][0] = i;           // delete all of word1
for (int j = 0; j <= n; j++) dp[0][j] = j;           // insert all of word2

for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
            dp[i][j] = dp[i - 1][j - 1];             // free match
        } else {
            dp[i][j] = 1 + Math.min(dp[i - 1][j - 1],          // replace
                            Math.min(dp[i - 1][j], dp[i][j - 1])); // delete / insert
        }
    }
}
return dp[m][n];
```

**Template (Distinct Subsequences - count ways s can equal t as subsequence):**
```java
// dp[i][j] = number of ways s[0..i-1] subsequences equal t[0..j-1]
int[][] dp = new int[m + 1][n + 1];
for (int i = 0; i <= m; i++) dp[i][0] = 1;           // empty target: one way (take nothing)
for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        dp[i][j] = dp[i - 1][j];                     // skip s[i-1]
        if (s.charAt(i - 1) == t.charAt(j - 1)) {
            dp[i][j] += dp[i - 1][j - 1];            // OR match it
        }
    }
}
return dp[m][n];
```

**Template (Regex Match `.` and `*`):**
```java
boolean[][] dp = new boolean[m + 1][n + 1];
dp[0][0] = true;
for (int j = 1; j <= n; j++) {
    if (p.charAt(j - 1) == '*') dp[0][j] = dp[0][j - 2];   // x* matches empty
}
for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        char pc = p.charAt(j - 1);
        if (pc == '.' || pc == s.charAt(i - 1)) {
            dp[i][j] = dp[i - 1][j - 1];
        } else if (pc == '*') {
            dp[i][j] = dp[i][j - 2];                  // zero occurrences of prev char
            char prev = p.charAt(j - 2);
            if (prev == '.' || prev == s.charAt(i - 1)) {
                dp[i][j] |= dp[i - 1][j];             // one+ occurrences
            }
        }
    }
}
return dp[m][n];
```

**Variations:**
- **One Edit Distance**: O(n) two-pointer, not full DP - just verify exactly one edit.
- **Interleaving String** (s3 is interleaving of s1 and s2): 2D dp over prefix lengths of s1, s2.
- **Min ASCII Delete Sum**: weighted edit distance.

**Common bugs:**
- Forgetting base cases for dp[0][j] and dp[i][0] - the "delete everything" / "insert everything" rows.
- For regex `*`: pattern char `*` requires looking at `p[j-2]`, not `p[j-1]`. Off-by-one is common.
- For distinct subsequences: dp[i][0] = 1 (not 0) - one way to match empty target.

**Example problems:**
- [Edit Distance](https://leetcode.com/problems/edit-distance/)
- [Regular Expression Matching](https://leetcode.com/problems/regular-expression-matching/)
- [Wildcard Matching](https://leetcode.com/problems/wildcard-matching/)
- [Distinct Subsequences](https://leetcode.com/problems/distinct-subsequences/)
- [Interleaving String](https://leetcode.com/problems/interleaving-string/)

---

## 7. String DP (Word Break / Decode Ways)

**When to use it:** Strings where each position has a "is the prefix valid?" question, and the answer composes.

**Trigger phrases:** "word break", "word break II (return all)", "decode ways", "concatenated words", "minimum ASCII delete (substring variant)".

**Anchor problem:** Word Break.

**Template (Word Break - boolean):**
```java
Set<String> dict = new HashSet<>(wordDict);
boolean[] dp = new boolean[s.length() + 1];
dp[0] = true;                                        // empty prefix is "breakable"
for (int i = 1; i <= s.length(); i++) {
    for (int j = 0; j < i; j++) {
        if (dp[j] && dict.contains(s.substring(j, i))) {
            dp[i] = true;
            break;                                    // one valid split is enough
        }
    }
}
return dp[s.length()];
```

**Template (Decode Ways - count):**
```java
int n = s.length();
int[] dp = new int[n + 1];
dp[0] = 1;
dp[1] = s.charAt(0) == '0' ? 0 : 1;
for (int i = 2; i <= n; i++) {
    int one = s.charAt(i - 1) - '0';
    int two = Integer.parseInt(s.substring(i - 2, i));
    if (one >= 1) dp[i] += dp[i - 1];                // single digit decode
    if (two >= 10 && two <= 26) dp[i] += dp[i - 2];  // two-digit decode
}
return dp[n];
```

**Variations:**
- **Word Break II** (return all valid sentences): DP for feasibility, then backtrack for enumeration.
- **Concatenated Words**: word break on each word using the rest of the dictionary.
- **Number of Distinct Decode Ways**: variant of decode-ways with characters.

**Common bugs:**
- For Decode Ways: forgetting "0" can't decode alone (only as part of 10 or 20).
- For Word Break: using `substring` in a tight loop - O(n) cost; consider Trie for big dictionaries.
- For Word Break II: pure backtracking without memo -> exponential blow-up. Always DP for feasibility first.

**Example problems:**
- [Word Break](https://leetcode.com/problems/word-break/)
- [Word Break II](https://leetcode.com/problems/word-break-ii/)
- [Decode Ways](https://leetcode.com/problems/decode-ways/)
- [Concatenated Words](https://leetcode.com/problems/concatenated-words/)
- [Decode Ways II](https://leetcode.com/problems/decode-ways-ii/)

---

## 8. Palindrome DP

**When to use it:** Problems about palindromic structure: longest palindromic subsequence, palindrome partitioning, counting palindromic substrings.

**Trigger phrases:** "longest palindromic subsequence", "palindrome partitioning II (min cuts)", "count palindromic substrings", "longest palindromic substring (DP variant)".

**Anchor problem:** Longest Palindromic Subsequence.

**Template (Longest Palindromic Subsequence):**
```java
int n = s.length();
int[][] dp = new int[n][n];                          // dp[i][j] = LPS in s[i..j]
for (int i = n - 1; i >= 0; i--) {
    dp[i][i] = 1;
    for (int j = i + 1; j < n; j++) {
        if (s.charAt(i) == s.charAt(j)) {
            dp[i][j] = dp[i + 1][j - 1] + 2;
        } else {
            dp[i][j] = Math.max(dp[i + 1][j], dp[i][j - 1]);
        }
    }
}
return dp[0][n - 1];
```
**Trick:** equivalent to `LCS(s, reverse(s))`.

**Template (Palindrome Partitioning II - min cuts to partition into palindromes):**
```java
int n = s.length();
boolean[][] isPalin = new boolean[n][n];
for (int i = n - 1; i >= 0; i--) {
    for (int j = i; j < n; j++) {
        if (s.charAt(i) == s.charAt(j) && (j - i < 2 || isPalin[i + 1][j - 1])) {
            isPalin[i][j] = true;
        }
    }
}
int[] cuts = new int[n];
for (int i = 0; i < n; i++) {
    if (isPalin[0][i]) { cuts[i] = 0; continue; }
    cuts[i] = i;
    for (int j = 1; j <= i; j++) {
        if (isPalin[j][i]) cuts[i] = Math.min(cuts[i], cuts[j - 1] + 1);
    }
}
return cuts[n - 1];
```

**Variations:**
- **Count Palindromic Substrings**: precompute `isPalin[i][j]`, then count all true entries.
- **Longest Palindromic Substring**: use expand-around-center (covered in strings.md) - simpler and faster than DP.
- **Palindrome Partitioning** (return all partitions): backtracking + precomputed `isPalin`.

**Common bugs:**
- Iterating in wrong order - `dp[i][j]` depends on `dp[i+1][...]` or `dp[...][j-1]`, so iterate `i` DECREASING.
- For palindrome check: `j - i < 2` is the special case (length 1 or 2 only need char match).

**Example problems:**
- [Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/)
- [Palindrome Partitioning II](https://leetcode.com/problems/palindrome-partitioning-ii/)
- [Palindromic Substrings](https://leetcode.com/problems/palindromic-substrings/)
- [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/)
- [Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/)

---

## 9. Stock Buy/Sell (State Machine DP)

**When to use it:** Sequence of prices where you make buy/sell decisions over time, often with constraints (max K transactions, cooldown, fees).

**Trigger phrases:** "best time to buy and sell stock I/II/III/IV", "with cooldown", "with transaction fee".

**Anchor problem:** Best Time to Buy and Sell Stock with Cooldown.

**Template (Stock I - one transaction):**
```java
int minPrice = Integer.MAX_VALUE, maxProfit = 0;
for (int p : prices) {
    minPrice = Math.min(minPrice, p);
    maxProfit = Math.max(maxProfit, p - minPrice);
}
return maxProfit;
```

**Template (Stock II - unlimited transactions):**
```java
int profit = 0;
for (int i = 1; i < prices.length; i++) {
    if (prices[i] > prices[i - 1]) profit += prices[i] - prices[i - 1];
}
return profit;
```
(Equivalent: collect every upswing.)

**Template (Stock with Cooldown - state machine):**
```java
// Three states per day: hold (own stock), sold (just sold today), rest (no stock, not in cooldown)
int hold = -prices[0], sold = 0, rest = 0;
for (int i = 1; i < prices.length; i++) {
    int prevHold = hold, prevSold = sold, prevRest = rest;
    hold = Math.max(prevHold, prevRest - prices[i]);     // keep holding OR buy from rest
    sold = prevHold + prices[i];                          // sell today
    rest = Math.max(prevRest, prevSold);                  // stay rest OR move from sold
}
return Math.max(sold, rest);
```

**Template (Stock IV - at most K transactions):**
```java
// dp[k][i] = max profit using at most k transactions by day i
int n = prices.length;
if (k >= n / 2) return stockII(prices);              // unlimited transactions
int[][] dp = new int[k + 1][n];
for (int t = 1; t <= k; t++) {
    int maxDiff = -prices[0];                         // best dp[t-1][j] - prices[j] for j < i
    for (int i = 1; i < n; i++) {
        dp[t][i] = Math.max(dp[t][i - 1], prices[i] + maxDiff);
        maxDiff = Math.max(maxDiff, dp[t - 1][i] - prices[i]);
    }
}
return dp[k][n - 1];
```

**Variations:**
- **Stock with Fee**: subtract fee on every sell.
- **Stock III**: at most 2 transactions; use Stock IV with K=2.

**Common bugs:**
- For Stock IV, naive O(K * N^2) TLEs - the `maxDiff` running optimization is required.
- For state machine: confusing prev/cur values - always capture prevs first, then update.

**Example problems:**
- [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)
- [Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)
- [Best Time to Buy and Sell Stock III](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/)
- [Best Time to Buy and Sell Stock IV](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)
- [Best Time to Buy and Sell Stock with Cooldown](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

---

## 10. Grid Path DP (2D matrix DP)

**When to use it:** Walk a grid from top-left to bottom-right, accumulating something (count of paths, min cost, max gold collected).

**Trigger phrases:** "unique paths", "unique paths II (with obstacles)", "min path sum", "dungeon game", "longest increasing path in matrix".

**Anchor problem:** Unique Paths.

**Template (Unique Paths - count paths in obstacle-free grid):**
```java
int[][] dp = new int[m][n];
for (int i = 0; i < m; i++) dp[i][0] = 1;
for (int j = 0; j < n; j++) dp[0][j] = 1;
for (int i = 1; i < m; i++) {
    for (int j = 1; j < n; j++) {
        dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
    }
}
return dp[m - 1][n - 1];
```

**Template (Min Path Sum):**
```java
int[][] dp = new int[m][n];
dp[0][0] = grid[0][0];
for (int i = 1; i < m; i++) dp[i][0] = dp[i - 1][0] + grid[i][0];
for (int j = 1; j < n; j++) dp[0][j] = dp[0][j - 1] + grid[0][j];
for (int i = 1; i < m; i++) {
    for (int j = 1; j < n; j++) {
        dp[i][j] = grid[i][j] + Math.min(dp[i - 1][j], dp[i][j - 1]);
    }
}
return dp[m - 1][n - 1];
```

**Template (Longest Increasing Path in Matrix - DFS + memo):**
```java
int[][] memo;
int[] dx = {-1, 1, 0, 0}, dy = {0, 0, -1, 1};

public int longestIncreasingPath(int[][] matrix) {
    memo = new int[matrix.length][matrix[0].length];
    int best = 0;
    for (int i = 0; i < matrix.length; i++)
        for (int j = 0; j < matrix[0].length; j++)
            best = Math.max(best, dfs(matrix, i, j));
    return best;
}

int dfs(int[][] m, int i, int j) {
    if (memo[i][j] != 0) return memo[i][j];
    int best = 1;
    for (int d = 0; d < 4; d++) {
        int ni = i + dx[d], nj = j + dy[d];
        if (ni >= 0 && ni < m.length && nj >= 0 && nj < m[0].length && m[ni][nj] > m[i][j]) {
            best = Math.max(best, 1 + dfs(m, ni, nj));
        }
    }
    return memo[i][j] = best;
}
```

**Variations:**
- **Unique Paths II** (with obstacles): `dp[i][j] = 0` if obstacle, else same transition.
- **Dungeon Game** (compute min initial health): DP from bottom-right BACKWARDS - reverse direction.
- **Cherry Pickup** (round trip): 3D state - two pickers moving simultaneously.

**Common bugs:**
- Off-by-one on grid dimensions.
- For Dungeon Game: doing DP forward instead of backward gives wrong answer (the health constraint changes the optimality).
- For longest increasing path: forgetting memoization makes it exponential.

**Example problems:**
- [Unique Paths](https://leetcode.com/problems/unique-paths/)
- [Unique Paths II](https://leetcode.com/problems/unique-paths-ii/)
- [Minimum Path Sum](https://leetcode.com/problems/minimum-path-sum/)
- [Dungeon Game](https://leetcode.com/problems/dungeon-game/)
- [Longest Increasing Path in a Matrix](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/)

---

## 11. Region DP (Histogram / Maximal Square)

**When to use it:** Find largest rectangle (or square) of 1s in a binary matrix. The square version has a beautifully short DP. The rectangle version uses histogram + monotonic stack.

**Trigger phrases:** "maximal square (largest all-1s square)", "maximal rectangle (in 0/1 matrix)", "count square submatrices", "largest sub-grid".

**Anchor problem:** Maximal Square.

**Template (Maximal Square):**
```java
// dp[i][j] = side length of largest square ending at (i, j) as bottom-right corner
int m = matrix.length, n = matrix[0].length;
int[][] dp = new int[m][n];
int best = 0;
for (int i = 0; i < m; i++) {
    for (int j = 0; j < n; j++) {
        if (matrix[i][j] == '1') {
            if (i == 0 || j == 0) dp[i][j] = 1;
            else dp[i][j] = Math.min(dp[i - 1][j - 1],
                            Math.min(dp[i - 1][j], dp[i][j - 1])) + 1;
            best = Math.max(best, dp[i][j]);
        }
    }
}
return best * best;
```
**Intuition:** A square of side `s` can end at `(i,j)` only if the three neighbors (top, left, top-left) all support a square of side `s-1`. So take min of three + 1.

**Template (Maximal Rectangle - reduce to histogram per row):**
```java
int m = matrix.length, n = matrix[0].length;
int[] heights = new int[n];
int best = 0;
for (int i = 0; i < m; i++) {
    for (int j = 0; j < n; j++) {
        heights[j] = (matrix[i][j] == '1') ? heights[j] + 1 : 0;
    }
    best = Math.max(best, largestRectangleInHistogram(heights));
}
return best;
```
Use the monotonic-stack histogram algorithm (covered in arrays.md / pattern 9 there).

**Variations:**
- **Count Square Submatrices**: same as Maximal Square but sum of `dp[i][j]` instead of max.
- **Largest 1-bordered Square**: 2 prefix sums (horizontal and vertical), then check borders.

**Common bugs:**
- For Maximal Square: returning the side length instead of `side * side` (the AREA).
- For Maximal Rectangle: incorrect height reset when matrix[i][j] == 0.

**Example problems:**
- [Maximal Square](https://leetcode.com/problems/maximal-square/)
- [Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/)
- [Count Square Submatrices with All Ones](https://leetcode.com/problems/count-square-submatrices-with-all-ones/)
- [Largest Plus Sign](https://leetcode.com/problems/largest-plus-sign/)
- [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) (cross-ref, stack-based sibling)

---

## 12. Interval DP

**When to use it:** Problems where the answer for range `[i, j]` depends on choosing a "split point" `k` in the middle, with cost depending on the split.

**Trigger phrases:** "burst balloons", "matrix chain multiplication", "remove boxes", "minimum cost to merge stones", "stone game", "guess number higher or lower II".

**Anchor problem:** Burst Balloons.

**Template (Burst Balloons - max coins from bursting in optimal order):**
```java
int n = nums.length;
int[] arr = new int[n + 2];
arr[0] = arr[n + 1] = 1;                             // virtual boundary balloons
for (int i = 0; i < n; i++) arr[i + 1] = nums[i];

int[][] dp = new int[n + 2][n + 2];                  // dp[i][j] = max coins from bursting (i, j) exclusive
for (int len = 2; len < n + 2; len++) {
    for (int i = 0; i + len < n + 2; i++) {
        int j = i + len;
        for (int k = i + 1; k < j; k++) {            // pick which balloon to burst LAST
            dp[i][j] = Math.max(dp[i][j],
                arr[i] * arr[k] * arr[j] + dp[i][k] + dp[k][j]);
        }
    }
}
return dp[0][n + 1];
```
**Key insight:** decide which balloon is burst LAST in the range. When it's burst, its neighbors are `arr[i]` and `arr[j]` (the boundaries).

**Template (Matrix Chain Multiplication):**
```java
// p[i] x p[i+1] for matrix i; dp[i][j] = min multiplications for p[i..j]
int n = p.length;
int[][] dp = new int[n][n];
for (int len = 2; len < n; len++) {
    for (int i = 0; i + len < n; i++) {
        int j = i + len;
        dp[i][j] = Integer.MAX_VALUE;
        for (int k = i + 1; k < j; k++) {            // split point
            dp[i][j] = Math.min(dp[i][j], dp[i][k] + dp[k][j] + p[i] * p[k] * p[j]);
        }
    }
}
return dp[0][n - 1];
```

**Variations:**
- **Min Cost to Merge Stones**: like matrix chain but constraint on group size.
- **Stone Game I/II/...**: similar interval-DP, with alternating players.
- **Guess Number Higher or Lower II**: dp[i][j] = min cost to guarantee finding any number in [i..j].

**Common bugs:**
- Iterating in wrong order: `len` must increase (smaller intervals computed first), then `i`.
- For Burst Balloons: thinking "first to burst" instead of "last to burst" - wrong recurrence.
- Forgetting boundary balloons (virtual 1s) in Burst Balloons.

**Example problems:**
- [Burst Balloons](https://leetcode.com/problems/burst-balloons/)
- [Minimum Cost to Merge Stones](https://leetcode.com/problems/minimum-cost-to-merge-stones/)
- [Remove Boxes](https://leetcode.com/problems/remove-boxes/)
- [Minimum Score Triangulation of Polygon](https://leetcode.com/problems/minimum-score-triangulation-of-polygon/)
- [Strange Printer](https://leetcode.com/problems/strange-printer/)

---

## 13. Tree DP

**When to use it:** Tree problems where each node's answer combines its children's answers, and you may need multiple states per node.

**Trigger phrases:** "house robber III (tree)", "binary tree max path sum", "diameter of binary tree", "longest univalue path".

**Anchor problem:** House Robber III.

**Template (House Robber III):**
```java
public int rob(TreeNode root) {
    int[] result = robSub(root);
    return Math.max(result[0], result[1]);
}

private int[] robSub(TreeNode node) {                // returns {rob_this, skip_this}
    if (node == null) return new int[2];
    int[] left = robSub(node.left);
    int[] right = robSub(node.right);

    int rob  = node.val + left[1] + right[1];        // rob current: children must skip
    int skip = Math.max(left[0], left[1]) + Math.max(right[0], right[1]); // skip: best of each child
    return new int[]{rob, skip};
}
```
**Pattern:** return a TUPLE of states per node, let the parent decide.

**Template (Max Path Sum in Binary Tree):**
```java
int best;
public int maxPathSum(TreeNode root) {
    best = Integer.MIN_VALUE;
    dfs(root);
    return best;
}
int dfs(TreeNode node) {
    if (node == null) return 0;
    int left = Math.max(0, dfs(node.left));          // ignore negative children
    int right = Math.max(0, dfs(node.right));
    best = Math.max(best, left + right + node.val);  // path THROUGH node
    return Math.max(left, right) + node.val;          // path going UP from node (one side only)
}
```
**Pattern:** track GLOBAL best across all "through-node paths", but return only "extendable upward" value.

**Variations:**
- **Diameter of Binary Tree**: same skeleton; track left+right as global best, return max(left, right) + 1.
- **Longest Univalue Path**: only count if child val equals current val.
- **Binary Tree Cameras** (min cameras to cover all nodes): 3 states per node (covered by self / covered by child / not covered).

**Common bugs:**
- Returning the GLOBAL best instead of the upward-extendable value - parent can't continue the path.
- Not handling null nodes (return 0 or a sentinel).
- For diameter: using node count vs edge count - clarify upfront.

**Example problems:**
- [House Robber III](https://leetcode.com/problems/house-robber-iii/)
- [Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/)
- [Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/)
- [Longest Univalue Path](https://leetcode.com/problems/longest-univalue-path/)
- [Binary Tree Cameras](https://leetcode.com/problems/binary-tree-cameras/)

---

## 14. Bitmask DP

**When to use it:** Subset problems where `n` is small (typically <= 20). Each state is a bitmask representing which items are "used" or "visited".

**Trigger phrases:** "traveling salesman", "partition to K equal sum subsets", "shortest path visiting all nodes", "smallest sufficient team".

**Anchor problem:** Partition to K Equal Sum Subsets.

**Template (Partition to K Equal Sum Subsets):**
```java
public boolean canPartitionKSubsets(int[] nums, int k) {
    int total = 0;
    for (int x : nums) total += x;
    if (total % k != 0) return false;
    int target = total / k;

    Arrays.sort(nums);
    if (nums[nums.length - 1] > target) return false;

    int n = nums.length;
    Boolean[] memo = new Boolean[1 << n];
    return dfs(nums, k, target, 0, 0, (1 << n) - 1, memo);
}

private boolean dfs(int[] nums, int k, int target, int curSum, int mask, int initialMask, Boolean[] memo) {
    if (k == 1) return true;
    if (curSum == target) {
        Boolean cached = memo[mask];
        if (cached != null) return cached;
        return memo[mask] = dfs(nums, k - 1, target, 0, mask, initialMask, memo);
    }
    for (int i = 0; i < nums.length; i++) {
        if ((mask & (1 << i)) != 0) continue;        // already used
        if (curSum + nums[i] > target) continue;
        if (dfs(nums, k, target, curSum + nums[i], mask | (1 << i), initialMask, memo)) return true;
    }
    return false;
}
```

**Template (Traveling Salesman - shortest tour):**
```java
int n = adj.length;
int[][] dp = new int[1 << n][n];                     // dp[mask][i] = min cost ending at city i, visited mask
for (int[] row : dp) Arrays.fill(row, Integer.MAX_VALUE / 2);
dp[1][0] = 0;                                         // start at city 0

for (int mask = 1; mask < (1 << n); mask++) {
    for (int u = 0; u < n; u++) {
        if ((mask & (1 << u)) == 0) continue;
        for (int v = 0; v < n; v++) {
            if ((mask & (1 << v)) != 0) continue;    // skip if already visited
            int newMask = mask | (1 << v);
            dp[newMask][v] = Math.min(dp[newMask][v], dp[mask][u] + adj[u][v]);
        }
    }
}
int best = Integer.MAX_VALUE;
for (int u = 1; u < n; u++) best = Math.min(best, dp[(1 << n) - 1][u] + adj[u][0]);
return best;
```

**Useful bitmask operations:**
```java
mask | (1 << i)        // set bit i
mask & ~(1 << i)       // clear bit i
mask ^ (1 << i)        // toggle bit i
(mask & (1 << i)) != 0 // test bit i
Integer.bitCount(mask) // count set bits
```

**Variations:**
- **Shortest Path Visiting All Nodes**: BFS with state (mask, currentNode).
- **Smallest Sufficient Team**: bitmask DP over skills mask.

**Common bugs:**
- Confusing "set" / "clear" / "test" - keep the operations table handy.
- For TSP: forgetting that the path must return to start - add `adj[u][0]` in final answer.
- `n` too large - bitmask DP is exponential in `n` (2^n states).

**Example problems:**
- [Partition to K Equal Sum Subsets](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/)
- [Shortest Path Visiting All Nodes](https://leetcode.com/problems/shortest-path-visiting-all-nodes/)
- [Smallest Sufficient Team](https://leetcode.com/problems/smallest-sufficient-team/)
- [Maximum Students Taking Exam](https://leetcode.com/problems/maximum-students-taking-exam/)
- [Minimum Number of Work Sessions to Finish the Tasks](https://leetcode.com/problems/minimum-number-of-work-sessions-to-finish-the-tasks/)

---

## 15. Digit DP

**When to use it:** Count numbers in `[0, N]` (or `[L, R]`) with digit-level constraints (no consecutive equal, sum of digits divisible by X, etc.).

**Trigger phrases:** "count numbers with no consecutive equal digits", "count integers with at most K repeating digits", "numbers without N as digit".

**Template (Count Numbers <= N with no consecutive equal digits):**
```java
String s = String.valueOf(n);
Integer[][][] memo = new Integer[s.length()][11][2];     // pos, lastDigit, tight

private int dp(int pos, int lastDigit, int tight, String s) {
    if (pos == s.length()) return 1;
    Integer cached = memo[pos][lastDigit][tight];
    if (cached != null) return cached;

    int limit = (tight == 1) ? s.charAt(pos) - '0' : 9;
    int count = 0;
    for (int d = 0; d <= limit; d++) {
        if (d == lastDigit) continue;                     // constraint
        int newTight = (tight == 1 && d == limit) ? 1 : 0;
        count += dp(pos + 1, d, newTight, s);
    }
    return memo[pos][lastDigit][tight] = count;
}
```

**Pattern:** `dp[pos][state][tight]` - position in number, extra state (last digit / sum / count), whether we're still tight to N's digits.

**Example problems:**
- [Numbers With Repeated Digits](https://leetcode.com/problems/numbers-with-repeated-digits/)
- [Count Numbers with Unique Digits](https://leetcode.com/problems/count-numbers-with-unique-digits/)
- [Non-negative Integers without Consecutive Ones](https://leetcode.com/problems/non-negative-integers-without-consecutive-ones/)
- [Number of Digit One](https://leetcode.com/problems/number-of-digit-one/)
- [Count of Integers](https://leetcode.com/problems/count-of-integers/)

---

## Suggested revision order if relearning from scratch

1. **Linear DP / Fibonacci (Climbing Stairs, House Robber)** - foundational; internalize "state, transition, base, order".
2. **0/1 Knapsack (Partition Equal Subset Sum)** - the canonical DP; learn the reverse-iteration trick for 1D.
3. **Unbounded Knapsack (Coin Change + Coin Change 2)** - learn coins-outer vs coins-inner loop order.
4. **LIS (O(n^2) + O(n log n) patience sort)** - elegant; comes up often.
5. **LCS** - foundation for many string DPs. Master it first.
6. **Edit Distance Family** - LCS + 3 transitions. Trickier base cases.
7. **String DP (Word Break, Decode Ways)** - 1D DP with substring check.
8. **Grid Path DP** - simple 2D; build intuition for matrix DPs.
9. **Region DP (Maximal Square)** - beautiful 1-line transition.
10. **Stock State Machine** - state-machine framing.
11. **Palindrome DP** - cousins of LCS.
12. **Tree DP** - tuple return per node.
13. **Interval DP (Burst Balloons)** - "split point" thinking.
14. **Bitmask DP** - small n (<=20), exponential states.
15. **Digit DP** - specialty pattern.

**Universal traps across every DP problem:**
- **Define the state in plain English** - if you can't write "dp[i] represents...", you don't understand the problem yet.
- **Off-by-one (1-indexed DP vs 0-indexed input)** - the #1 DP bug. Pick a convention and stick to it.
- **Wrong base case** - usually dp[0] (or dp[0][0]). For boolean DPs, dp[0] = true. For min DPs, dp[0] = 0 or appropriate sentinel.
- **Iteration order** - dependencies must be computed first. For 1D knapsack, REVERSE iteration is the difference between 0/1 and unbounded.
- **Reconstructing the solution** vs **just the value** - if you need the actual subsequence/partition, you need backtracking from the DP table.
- **Integer overflow** in counting DPs - use `long` or `mod 1e9+7`.
- **`Integer.MAX_VALUE + 1` overflow** in min-DPs - use a safe sentinel like `n + 1`.
- **Memoization with HashMap<List<...>, V>** - slow; prefer multi-dim array if dimensions are bounded.
- **Pure recursion without memo** - exponential blow-up. Always cache (top-down) or tabulate (bottom-up).
- **Space optimization (2D -> 1D)** - only safe when current row depends ONLY on previous row (not on `dp[i-2]`); be careful with iteration direction.
