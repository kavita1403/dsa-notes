# Strings - Cheat Sheet

**Decision fork:** Contiguous substring with property -> sliding window. Palindrome -> two pointers (verify) / expand around center (find). Anagram / char count -> frequency map (`int[26]`). Nested brackets / parsing -> stack. Transform A -> B -> DP. Generate all valid -> backtracking. Prefix lookup -> trie. Find pattern in text -> KMP. Pick chars under global constraint -> greedy + heap.

## Two Pointers (palindrome verify, reverse)
```java
int l = 0, r = s.length() - 1;
while (l < r) {
    while (l < r && !isValid(s.charAt(l))) l++;     // skip non-alphanumeric, etc.
    while (l < r && !isValid(s.charAt(r))) r--;
    if (lower(s.charAt(l)) != lower(s.charAt(r))) return false;
    l++; r--;
}
```
Don't forget `l < r` guard in INNER skip loops.

## Sliding Window (variable size)
```java
int l = 0, best = 0;
Map<Character, Integer> count = new HashMap<>();
for (int r = 0; r < s.length(); r++) {
    count.merge(s.charAt(r), 1, Integer::sum);
    while (isInvalid(count)) {
        if (count.merge(s.charAt(l++), -1, Integer::sum) == 0) count.remove(s.charAt(l - 1));
    }
    best = Math.max(best, r - l + 1);
}
```
Sliding window MUST shrink from the left or it degrades to O(n^2). For min-window, capture `bestL, bestR` separately because final l may be past the optimum.

## Sliding Window (fixed size - permutation in string)
```java
int[] need = new int[26], have = new int[26];
for (char c : p.toCharArray()) need[c - 'a']++;
for (int r = 0; r < s.length(); r++) {
    have[s.charAt(r) - 'a']++;
    if (r >= k) have[s.charAt(r - k) - 'a']--;        // drop char that fell off
    if (r >= k - 1 && Arrays.equals(need, have)) return true;
}
```

## Frequency Map (anagram, char distribution)
```java
int[] freq = new int[26];                              // lowercase ASCII -> array, NOT map
for (char c : s.toCharArray()) freq[c - 'a']++;
for (char c : t.toCharArray()) if (--freq[c - 'a'] < 0) return false;
return true;
```
Use `int[26]` for lowercase a-z (10x faster than HashMap). Use `Map<Character,Integer>` only for unicode.

## Stack-Based Parsing (parentheses, decode string)
```java
Deque<Character> stack = new ArrayDeque<>();           // ArrayDeque, NOT legacy Stack
for (char c : s.toCharArray()) {
    if (isOpen(c)) stack.push(c);
    else if (stack.isEmpty() || stack.pop() != matching(c)) return false;
}
return stack.isEmpty();
```
For `decodeString` (`3[a2[c]]`): two stacks (countStack + strStack), push BOTH on `[`, pop BOTH on `]`, repeat current into prev.

## DP on Strings (edit distance / LCS template)
```java
int[][] dp = new int[m + 1][n + 1];
for (int i = 0; i <= m; i++) dp[i][0] = i;             // delete all of a
for (int j = 0; j <= n; j++) dp[0][j] = j;             // insert all of b
for (int i = 1; i <= m; i++)
    for (int j = 1; j <= n; j++)
        dp[i][j] = a.charAt(i-1) == b.charAt(j-1)
            ? dp[i-1][j-1]
            : 1 + Math.min(dp[i-1][j-1], Math.min(dp[i-1][j], dp[i][j-1]));
```
DP sized `[m+1][n+1]` but `charAt(i-1)` indexes the string. Mixing 1-based dp with 0-based strings is the #1 bug.

## Palindrome - Expand Around Center
```java
int expand(String s, int l, int r) {
    while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) { l--; r++; }
    return r - l - 1;
}
// Call with both (i, i) odd-length AND (i, i+1) even-length centers. 2n-1 total centers.
// Start of longest = i - (len - 1) / 2;
```

## Backtracking (generate parentheses)
```java
void backtrack(StringBuilder cur, int open, int close, int n, List<String> out) {
    if (cur.length() == 2 * n) { out.add(cur.toString()); return; }
    if (open  < n)    { cur.append('('); backtrack(cur, open+1, close, n, out); cur.deleteCharAt(cur.length()-1); }
    if (close < open) { cur.append(')'); backtrack(cur, open, close+1, n, out); cur.deleteCharAt(cur.length()-1); }
}
```
ALWAYS undo (the line after recursive call). Adding `cur` itself to output (without `.toString()`) shares mutating state across results.

## Trie (prefix tree)
```java
class TrieNode { TrieNode[] children = new TrieNode[26]; boolean isWord; }

void insert(String word) {
    TrieNode n = root;
    for (char c : word.toCharArray()) {
        int i = c - 'a';
        if (n.children[i] == null) n.children[i] = new TrieNode();
        n = n.children[i];
    }
    n.isWord = true;                                  // MUST mark, or search returns false
}
```

## KMP (find pattern in text - O(n+m))
```java
int[] lps = buildLps(pattern);
int i = 0, j = 0;
while (i < n) {
    if (text.charAt(i) == pattern.charAt(j)) {
        i++; j++;
        if (j == m) return i - m;                     // match found at i - m
    } else if (j > 0) {
        j = lps[j - 1];                               // fall back in PATTERN, not text
    } else { i++; }
}
```
On mismatch: jump using `lps[j-1]`, NOT `lps[j]`. Build LPS by comparing `p[i]` to `p[len]`.

## Greedy (reorganize string - max-heap by freq)
```java
PriorityQueue<int[]> pq = new PriorityQueue<>((a,b) -> b[1] - a[1]);   // {char, count}
int[] prev = null;
while (!pq.isEmpty()) {
    int[] cur = pq.poll();
    sb.append((char)(cur[0] + 'a'));
    if (--cur[1] >= 0 && prev != null && prev[1] > 0) pq.add(prev);
    prev = cur;
}
```
Pattern: pick best, defer re-adding previous by one step so you never use the same char twice in a row.

## Universal traps
- **Substring vs subsequence**: substring is contiguous, subsequence is not. Read problem twice.
- **`s += c` in a loop is O(n^2)**: ALWAYS `StringBuilder`.
- **`==` on Strings**: compares references. Use `.equals()`.
- **String is immutable**: any "mutation" allocates. Use `char[]` or `StringBuilder` in hot loops.
- **`'a' - 'A' = 32`**: case-conversion arithmetic. Prefer `Character.toLowerCase()` for unicode safety.
- **`s.length()` returns CODE UNITS, not characters**: surrogate pairs count as 2. For unicode iteration, use `s.codePoints()`.
