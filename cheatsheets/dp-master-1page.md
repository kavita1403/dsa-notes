# DP - Cheat Sheet

**4 questions for EVERY DP problem:** (1) STATE - what does `dp[i]` mean in plain English? (2) TRANSITION - how does `dp[i]` come from earlier dp[j]? (3) BASE - what's dp[0]? (4) ORDER - in what order do we fill so dependencies are ready?

**Decision fork:** Count ways -> additive (`dp[i] += dp[j]`). Min/max -> min/max DP. Boolean possible -> OR DP. Pick subset each-once -> 0/1 knapsack (1D capacity REVERSE). Reuse allowed -> unbounded knapsack (1D capacity FORWARD). Subsequence (not contiguous) -> 2D dp by prefix lengths (LCS family). Split into best partition -> dp[i] = min over j of (dp[j] + cost(j+1..i)). State changes over time (stocks) -> state-machine DP. Grid path -> dp[i][j] from neighbors. Largest 1-square -> dp[i][j] = min(3 neighbors) + 1. Burst-balloons / matrix-chain -> interval DP picking split k. Tree -> return tuple per node. Subset bitmask (n <= 20) -> dp[mask].

## Linear DP (Climbing Stairs / House Robber)
```java
// Climbing stairs: dp[i] = dp[i-1] + dp[i-2]
int prev2 = 1, prev1 = 2;
for (int i = 3; i <= n; i++) { int cur = prev1 + prev2; prev2 = prev1; prev1 = cur; }
return prev1;

// House Robber: dp[i] = max(dp[i-1], dp[i-2] + nums[i])
int prev2 = 0, prev1 = 0;
for (int x : nums) { int cur = Math.max(prev1, prev2 + x); prev2 = prev1; prev1 = cur; }
return prev1;
```
House Robber II (circle): try `rob(0..n-2)` AND `rob(1..n-1)`, take max.

## 0/1 Knapsack (Partition Equal Subset Sum)
```java
boolean[] dp = new boolean[target + 1];
dp[0] = true;
for (int x : nums) {
    for (int c = target; c >= x; c--) dp[c] |= dp[c - x];     // REVERSE - each item once
}
return dp[target];
```
Capacity FORWARD in 1D = unbounded knapsack (item reuse). REVERSE = 0/1.

## Unbounded Knapsack - Coin Change (MIN coins)
```java
int[] dp = new int[amount + 1];
Arrays.fill(dp, amount + 1);                                  // sentinel = impossible
dp[0] = 0;
for (int i = 1; i <= amount; i++)
    for (int coin : coins)
        if (i >= coin) dp[i] = Math.min(dp[i], dp[i - coin] + 1);
return dp[amount] > amount ? -1 : dp[amount];
```

## Coin Change II (COUNT ways - combinations)
```java
int[] dp = new int[amount + 1];
dp[0] = 1;
for (int coin : coins)                                        // OUTER: coins
    for (int c = coin; c <= amount; c++)                      // INNER: capacity
        dp[c] += dp[c - coin];
```
Loop order matters! coins-outer counts combinations; target-outer counts permutations.

## LIS - O(n^2) DP
```java
int[] dp = new int[n]; Arrays.fill(dp, 1);
for (int i = 1; i < n; i++)
    for (int j = 0; j < i; j++)
        if (nums[j] < nums[i]) dp[i] = Math.max(dp[i], dp[j] + 1);
return Arrays.stream(dp).max().getAsInt();
```

## LIS - O(n log n) Patience Sort
```java
int[] tails = new int[n]; int size = 0;
for (int x : nums) {
    int lo = 0, hi = size;
    while (lo < hi) {
        int mid = (lo + hi) >>> 1;
        if (tails[mid] < x) lo = mid + 1; else hi = mid;
    }
    tails[lo] = x;
    if (lo == size) size++;
}
return size;
```
`tails[i]` is smallest tail of any LIS of length `i+1`. Use `<=` for non-strict.

## LCS - Longest Common Subsequence
```java
int[][] dp = new int[m + 1][n + 1];
for (int i = 1; i <= m; i++)
    for (int j = 1; j <= n; j++)
        dp[i][j] = a.charAt(i-1) == b.charAt(j-1)
            ? dp[i-1][j-1] + 1
            : Math.max(dp[i-1][j], dp[i][j-1]);
return dp[m][n];
```
Off-by-one warning: dp sized [m+1][n+1], charAt uses i-1. Stick to ONE convention.

## Edit Distance
```java
int[][] dp = new int[m + 1][n + 1];
for (int i = 0; i <= m; i++) dp[i][0] = i;
for (int j = 0; j <= n; j++) dp[0][j] = j;
for (int i = 1; i <= m; i++)
    for (int j = 1; j <= n; j++)
        dp[i][j] = a.charAt(i-1) == b.charAt(j-1)
            ? dp[i-1][j-1]
            : 1 + Math.min(dp[i-1][j-1], Math.min(dp[i-1][j], dp[i][j-1]));
```

## Word Break
```java
boolean[] dp = new boolean[s.length() + 1];
dp[0] = true;
for (int i = 1; i <= s.length(); i++)
    for (int j = 0; j < i; j++)
        if (dp[j] && dict.contains(s.substring(j, i))) { dp[i] = true; break; }
return dp[s.length()];
```

## Decode Ways
```java
int[] dp = new int[n + 1];
dp[0] = 1;
dp[1] = s.charAt(0) == '0' ? 0 : 1;
for (int i = 2; i <= n; i++) {
    int one = s.charAt(i-1) - '0';
    int two = Integer.parseInt(s.substring(i-2, i));
    if (one >= 1) dp[i] += dp[i-1];
    if (two >= 10 && two <= 26) dp[i] += dp[i-2];
}
```
Watch for "0" - can't decode alone, only as part of 10 or 20.

## Stock with Cooldown (state machine)
```java
int hold = -prices[0], sold = 0, rest = 0;
for (int i = 1; i < prices.length; i++) {
    int pHold = hold, pSold = sold, pRest = rest;
    hold = Math.max(pHold, pRest - prices[i]);                // keep or buy from rest
    sold = pHold + prices[i];                                  // sell today
    rest = Math.max(pRest, pSold);                             // rest or recover from sold
}
return Math.max(sold, rest);
```

## Stock with Unlimited Transactions
```java
int profit = 0;
for (int i = 1; i < prices.length; i++)
    if (prices[i] > prices[i-1]) profit += prices[i] - prices[i-1];
```

## Unique Paths (Grid)
```java
int[][] dp = new int[m][n];
for (int i = 0; i < m; i++) dp[i][0] = 1;
for (int j = 0; j < n; j++) dp[0][j] = 1;
for (int i = 1; i < m; i++)
    for (int j = 1; j < n; j++)
        dp[i][j] = dp[i-1][j] + dp[i][j-1];
```

## Min Path Sum
```java
dp[0][0] = grid[0][0];
for (int i = 1; i < m; i++) dp[i][0] = dp[i-1][0] + grid[i][0];
for (int j = 1; j < n; j++) dp[0][j] = dp[0][j-1] + grid[0][j];
for (int i = 1; i < m; i++)
    for (int j = 1; j < n; j++)
        dp[i][j] = grid[i][j] + Math.min(dp[i-1][j], dp[i][j-1]);
```

## Maximal Square (largest 1s square in binary matrix)
```java
int[][] dp = new int[m][n];
int best = 0;
for (int i = 0; i < m; i++)
    for (int j = 0; j < n; j++)
        if (matrix[i][j] == '1') {
            dp[i][j] = (i == 0 || j == 0) ? 1
                : Math.min(dp[i-1][j-1], Math.min(dp[i-1][j], dp[i][j-1])) + 1;
            best = Math.max(best, dp[i][j]);
        }
return best * best;       // AREA, not side length
```
Return `side * side` (area), not just side.

## Maximal Rectangle (reduce to histogram per row)
```java
int[] heights = new int[n];
for (int i = 0; i < m; i++) {
    for (int j = 0; j < n; j++)
        heights[j] = (matrix[i][j] == '1') ? heights[j] + 1 : 0;
    best = Math.max(best, largestRectInHistogram(heights));
}
```

## Longest Palindromic Subsequence
```java
int[][] dp = new int[n][n];
for (int i = n - 1; i >= 0; i--) {            // iterate i DECREASING (depends on i+1)
    dp[i][i] = 1;
    for (int j = i + 1; j < n; j++) {
        if (s.charAt(i) == s.charAt(j)) dp[i][j] = dp[i+1][j-1] + 2;
        else dp[i][j] = Math.max(dp[i+1][j], dp[i][j-1]);
    }
}
return dp[0][n-1];
```
Equivalent: `LCS(s, reverse(s))`.

## Interval DP - Burst Balloons
```java
int[] arr = new int[n + 2]; arr[0] = arr[n+1] = 1;
for (int i = 0; i < n; i++) arr[i+1] = nums[i];
int[][] dp = new int[n+2][n+2];
for (int len = 2; len < n+2; len++) {
    for (int i = 0; i + len < n+2; i++) {
        int j = i + len;
        for (int k = i + 1; k < j; k++)                          // burst k LAST
            dp[i][j] = Math.max(dp[i][j],
                arr[i] * arr[k] * arr[j] + dp[i][k] + dp[k][j]);
    }
}
return dp[0][n+1];
```
Think "which is burst LAST in this range" - its neighbors are arr[i] and arr[j].

## Tree DP - House Robber III
```java
int[] dfs(TreeNode node) {                  // returns {rob, skip}
    if (node == null) return new int[2];
    int[] L = dfs(node.left), R = dfs(node.right);
    int rob = node.val + L[1] + R[1];
    int skip = Math.max(L[0], L[1]) + Math.max(R[0], R[1]);
    return new int[]{rob, skip};
}
```
Return TUPLE of states. Parent picks best combo.

## Tree DP - Max Path Sum
```java
int best = Integer.MIN_VALUE;
int dfs(TreeNode n) {
    if (n == null) return 0;
    int L = Math.max(0, dfs(n.left)), R = Math.max(0, dfs(n.right));
    best = Math.max(best, L + R + n.val);   // path THROUGH n
    return Math.max(L, R) + n.val;           // path going UP (one side only)
}
```

## Bitmask DP - Partition K Equal Subsets
```java
// Sort desc; try to fill k buckets to target each. Memo on bucket-state mask.
Boolean[] memo = new Boolean[1 << n];
boolean dfs(int idx, int curBucket, int k, int target, int mask, ...) {
    if (k == 0) return true;
    if (curBucket == target) return dfs(0, 0, k - 1, target, mask, ...);
    for (int i = 0; i < n; i++)
        if ((mask & (1 << i)) == 0 && curBucket + nums[i] <= target)
            if (dfs(i + 1, curBucket + nums[i], k, target, mask | (1 << i), ...)) return true;
    return false;
}
```

## Bitmask ops
```java
mask | (1 << i)            // set bit i
mask & ~(1 << i)           // clear bit i
mask ^ (1 << i)            // toggle bit i
(mask & (1 << i)) != 0     // test bit i
Integer.bitCount(mask)
```

## Universal traps
- **State must be definable in plain English** ("dp[i] = the number of ways to..."). If you can't, you don't understand the problem.
- **Off-by-one with 1-indexed dp vs 0-indexed input** - the #1 DP bug. Pick a convention, stick to it.
- **Iteration order** - for 0/1 knapsack 1D, iterate capacity REVERSE; for unbounded, FORWARD.
- **Wrong base case** - dp[0] sets the tone. Counting -> dp[0] = 1. Min -> dp[0] = 0. Boolean -> dp[0] = true.
- **`Integer.MAX_VALUE + 1` overflow** in min-DPs - use a sentinel like `n + 1` or `amount + 1`.
- **Integer overflow in counting DPs** - use `long` or `% 1_000_000_007`.
- **Pure recursion without memo** - exponential. ALWAYS cache (memo) or tabulate.
- **Space optimization (2D -> 1D)** - safe only if current row depends on previous row only. Watch iteration direction.
- **Coin change combinations vs permutations** - coins-outer = combinations; target-outer = permutations.
- **For LIS O(n log n)** - `tails` is NOT the LIS, just its length.
- **For Maximal Square** - return AREA (side * side), not side.
- **For Burst Balloons** - think "burst LAST", not "burst first".
