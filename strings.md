# String Patterns - Master Reference Note

```java
// Standard helpers assumed throughout
String s = ...;
char[] chars = s.toCharArray();        // mutable view; cheap one-time copy
int n = s.length();
s.charAt(i);                            // O(1), preferred over toCharArray() for read-only
new StringBuilder().append(...).toString();   // for mutation; never += in loops
```

---

## Decision Fork (ask yourself this before writing any code)

> Need a **contiguous substring** with some constraint? -> **Sliding Window**. Two pointers `l`, `r` expand/shrink to maintain validity.
>
> Need to check or build a **palindrome**? -> **Two Pointers** (verify) OR **Expand Around Center** (find all/longest) OR **DP** (count + boolean table for variants).
>
> Need to count chars / detect **anagram / permutation**? -> **Frequency Map** (`int[26]` for lowercase, `Map<Character, Integer>` for unicode).
>
> Need to parse **nested structure** or **brackets**? -> **Stack**. Match opens with closes; for `decodeString` push state on `[`, pop on `]`.
>
> Need to find an **exact pattern** in a longer string? -> **KMP / Rabin-Karp** (no library shortcut beats them for O(n+m)).
>
> Need to **transform** string A into string B (insert/delete/replace)? -> **DP** (edit distance / LCS family). 2D table indexed by prefix lengths.
>
> Need to **generate all** valid strings (parentheses, partitions, combinations)? -> **Backtracking**. Recurse + undo.
>
> Need **prefix-based** lookups (autocomplete, word dictionary)? -> **Trie**. Per-character branching.
>
> Need to pick chars to satisfy a global property (no two adjacent same, partition into labels)? -> **Greedy** (often + heap or counting).

Everything else in string-land is a variant or combination of these nine.

---

## Quick Lookup Table (read this first when revisiting after a gap)

| # | Pattern | Trigger phrases | Anchor problem | Core idea |
|---|---|---|---|---|
| 1 | Two Pointers | "palindrome check", "reverse", "valid palindrome", "compare from both ends" | Valid Palindrome | Converge `l` and `r` toward center; one pass O(n) |
| 2 | Sliding Window | "longest/shortest substring with", "at most K distinct", "without repeating chars" | Longest Substring Without Repeating Characters | Expand `r`, shrink `l` while invariant violated; track answer at each step |
| 3 | Frequency Map / Hashing | "anagram", "permutation", "character count", "all same chars" | Valid Anagram | `int[26]` for ASCII / `Map<Character,Integer>` for unicode; compare or maintain delta |
| 4 | Stack-based Parsing | "valid parentheses", "decode string", "simplify path", "evaluate expression" | Valid Parentheses | Push opens / state; pop on close; check match |
| 5 | DP on Strings | "edit distance", "longest common subsequence", "regex match", "wildcard match", "interleaving" | Edit Distance | 2D dp[i][j] over prefix lengths; transition by character match/mismatch |
| 6 | Palindrome (Expand / DP / Manacher) | "longest palindromic substring", "count palindromic substrings", "palindrome partitioning" | Longest Palindromic Substring | Expand around each center (2n-1 centers); for count, same template; for partition, backtrack |
| 7 | Backtracking on Strings | "generate all", "letter combinations", "partition", "restore IP", "word break II" | Generate Parentheses | Build candidate char-by-char, prune invalid, undo last choice on return |
| 8 | Trie / Prefix Tree | "prefix", "autocomplete", "word search II (dictionary on grid)", "starts with" | Implement Trie | Tree keyed by character; mark `isWord` at terminal nodes |
| 9 | Pattern Matching (KMP) | "find pattern in text", "repeated substring", "shortest period" | Implement strStr() | Precompute longest-proper-prefix-which-is-also-suffix (LPS) array; never re-scan text |
| 10 | Greedy on Strings | "reorganize so no two adjacent same", "partition labels", "task scheduler" | Reorganize String | Sort by frequency / use max-heap; pick best choice that keeps remaining feasible |
| 11 (bonus) | Number-String Conversion | "Roman to integer", "atoi", "integer to English words", "count and say" | Roman to Integer | Map symbol values; sweep with lookahead for subtractive cases |

---

## 1. Two Pointers (converge from ends)

**When to use it:** Whenever the answer depends on comparing/inspecting characters from both ends moving inward - palindrome verification, string reversal in-place, splitting based on a pivot character.

**Trigger phrases:** "valid palindrome", "reverse string", "is rotation of", "compare alphanumeric", "reverse vowels".

**Anchor problem:** Valid Palindrome - if you forget everything else, re-derive from this.

**Template:**
```java
int l = 0, r = s.length() - 1;
while (l < r) {
    // skip / filter logic
    while (l < r && !isValid(s.charAt(l))) l++;
    while (l < r && !isValid(s.charAt(r))) r--;

    if (Character.toLowerCase(s.charAt(l)) != Character.toLowerCase(s.charAt(r))) return false;
    l++; r--;
}
return true;
```

**Variations:**
- **In-place reverse** (`char[]` not `String` - String is immutable): swap `l` and `r`, walk inward.
- **Reverse only vowels / specific chars**: skip-on-each-side first, then swap.
- **Compare with offset** (one starts at 0, other at start of second half): same skeleton, different bounds.

**Common bugs:**
- Forgetting `l < r` guard in INNER skip loops -> walks off the array.
- Calling `s.charAt(l)` on a String inside a hot loop is fine (O(1)) but if you mutate, `char[] arr = s.toCharArray()` once and work on that.
- Off-by-one when `n` is odd vs even - both `l < r` AND `l <= r` (rarely) need care; `l < r` is correct for symmetric checks.

---

## 2. Sliding Window (variable + fixed size)

**When to use it:** Find a **contiguous substring** satisfying a constraint (longest, shortest, with property X). The window expands by moving `r`, contracts by moving `l`, and you track the best answer at each `r`.

**Trigger phrases:** "longest substring with at most K distinct chars", "longest substring without repeating", "smallest window containing all chars of T", "permutation in string", "find all anagrams".

**Anchor problem:** Longest Substring Without Repeating Characters.

**Template (variable-size window):**
```java
int l = 0, best = 0;
Map<Character, Integer> count = new HashMap<>();   // or int[26] if lowercase ASCII

for (int r = 0; r < s.length(); r++) {
    char c = s.charAt(r);
    count.merge(c, 1, Integer::sum);

    while (isInvalid(count)) {              // shrink until window is valid again
        char left = s.charAt(l++);
        if (count.merge(left, -1, Integer::sum) == 0) count.remove(left);
    }

    best = Math.max(best, r - l + 1);
}
return best;
```

**Template (fixed-size window, e.g. "permutation in string"):**
```java
int k = pattern.length();
int[] need = new int[26];
int[] have = new int[26];
for (char c : pattern.toCharArray()) need[c - 'a']++;

for (int r = 0; r < s.length(); r++) {
    have[s.charAt(r) - 'a']++;
    if (r >= k) have[s.charAt(r - k) - 'a']--;   // window slides; drop char that just fell off

    if (r >= k - 1 && Arrays.equals(need, have)) return true;
}
return false;
```

**Variations:**
- **Min Window Substring**: track count of chars *matched* (not just deltas); shrink while still matched.
- **Longest Repeating Character Replacement**: `windowLen - maxFreqInWindow <= k`.
- **Fruit Into Baskets**: same as "longest substring with at most 2 distinct".

**Common bugs:**
- Forgetting to **shrink from the left** (turning sliding window into "track every prefix" - O(n^2)).
- Using `count.containsKey(c)` then `count.get(c)` (two map lookups) - use `merge` / `getOrDefault` for one.
- For fixed-size: forgetting the `r >= k` guard - drops chars that never entered.
- For "min window", returning `r - l + 1` from the WRONG l (after shrink past optimum). Capture `bestL`, `bestR` separately when answer improves.

---

## 3. Frequency Map / Hashing

**When to use it:** Compare character distributions - anagram detection, character counting, mapping equivalence (isomorphic), finding all occurrences of a permutation.

**Trigger phrases:** "anagram", "permutation", "rearrangement of", "isomorphic", "same character distribution", "common chars".

**Anchor problem:** Valid Anagram.

**Template:**
```java
// For ASCII lowercase, int[26] is faster + cleaner than HashMap
int[] freq = new int[26];
for (char c : s.toCharArray()) freq[c - 'a']++;
for (char c : t.toCharArray()) freq[c - 'a']--;
for (int f : freq) if (f != 0) return false;
return true;

// For unicode / generic chars
Map<Character, Integer> freq = new HashMap<>();
for (char c : s.toCharArray()) freq.merge(c, 1, Integer::sum);
```

**Variations:**
- **Group Anagrams**: sort each string OR build a frequency signature as the map key.
  ```java
  Map<String, List<String>> groups = new HashMap<>();
  for (String w : words) {
      char[] arr = w.toCharArray();
      Arrays.sort(arr);
      groups.computeIfAbsent(new String(arr), k -> new ArrayList<>()).add(w);
  }
  ```
- **Isomorphic Strings**: two maps `s->t` and `t->s`; reject if either map disagrees.
- **Find All Anagrams in a String**: combine with sliding window (Q2 template).
- **Ransom Note**: subtract one from the other; reject if any goes negative.

**Common bugs:**
- Using `Map` when `int[26]` would do (10x slower).
- Forgetting to handle case (`'A'` vs `'a'`) - normalize early.
- Mutating a key's count then forgetting to `remove()` when it hits 0 - `containsKey` later returns true incorrectly.

---

## 4. Stack-Based Parsing

**When to use it:** Whenever the input has **nested or matched** structure - brackets, expression evaluation, decoding repeated groups, simplifying directory paths.

**Trigger phrases:** "valid parentheses", "decode string `k[encoded]`", "simplify path `..`/`.`", "evaluate basic calculator", "next greater element on a string of digits".

**Anchor problem:** Valid Parentheses.

**Template (matching brackets):**
```java
Deque<Character> stack = new ArrayDeque<>();
Map<Character, Character> close = Map.of(')', '(', ']', '[', '}', '{');

for (char c : s.toCharArray()) {
    if (close.containsValue(c)) {
        stack.push(c);                                       // open
    } else if (close.containsKey(c)) {
        if (stack.isEmpty() || stack.pop() != close.get(c)) return false;
    }
}
return stack.isEmpty();
```

**Template (decode string `3[a2[c]]` -> `accaccacc`):**
```java
Deque<Integer> countStack = new ArrayDeque<>();
Deque<StringBuilder> strStack = new ArrayDeque<>();
StringBuilder cur = new StringBuilder();
int k = 0;

for (char c : s.toCharArray()) {
    if (Character.isDigit(c)) {
        k = k * 10 + (c - '0');
    } else if (c == '[') {
        countStack.push(k);
        strStack.push(cur);
        k = 0;
        cur = new StringBuilder();
    } else if (c == ']') {
        StringBuilder prev = strStack.pop();
        int repeat = countStack.pop();
        for (int i = 0; i < repeat; i++) prev.append(cur);
        cur = prev;
    } else {
        cur.append(c);
    }
}
return cur.toString();
```

**Variations:**
- **Simplify Path**: split by `/`, push directories, pop on `..`, ignore `.` and empty.
- **Basic Calculator**: two stacks (operands + operators) OR sign-tracking for + - only.
- **Remove K Digits**: monotonic stack to keep smallest result.

**Common bugs:**
- Using `Stack` (legacy, synchronized) instead of `ArrayDeque` - slower and inherits from Vector.
- For multi-digit numbers, forgetting to accumulate (`k = k * 10 + (c - '0')`).
- For `decodeString`, mixing up the order: count goes on its stack BEFORE string goes on its stack (LIFO order matters).

---

## 5. Dynamic Programming on Strings (LCS family)

**When to use it:** Transform / compare two strings, where the answer for prefixes (i, j) depends on (i-1, j), (i, j-1), (i-1, j-1). Classic family: LCS, edit distance, regex/wildcard match, distinct subsequences, interleaving.

**Trigger phrases:** "edit distance", "longest common subsequence", "regex match `.` `*`", "wildcard match `?` `*`", "interleaving string", "distinct subsequences".

**Anchor problem:** Edit Distance.

**Template (Edit Distance):**
```java
int m = word1.length(), n = word2.length();
int[][] dp = new int[m + 1][n + 1];

for (int i = 0; i <= m; i++) dp[i][0] = i;     // delete i chars
for (int j = 0; j <= n; j++) dp[0][j] = j;     // insert j chars

for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
            dp[i][j] = dp[i - 1][j - 1];           // free match
        } else {
            dp[i][j] = 1 + Math.min(dp[i - 1][j - 1],          // replace
                            Math.min(dp[i - 1][j], dp[i][j - 1])); // delete / insert
        }
    }
}
return dp[m][n];
```

**Template (LCS - longest common subsequence):**
```java
for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        if (a.charAt(i - 1) == b.charAt(j - 1)) {
            dp[i][j] = dp[i - 1][j - 1] + 1;
        } else {
            dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
        }
    }
}
```

**Template (Regex Match `.` `*`):**
```java
boolean[][] dp = new boolean[m + 1][n + 1];
dp[0][0] = true;
for (int j = 1; j <= n; j++) {
    if (p.charAt(j - 1) == '*') dp[0][j] = dp[0][j - 2];   // pattern can match empty
}
for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        char pc = p.charAt(j - 1);
        if (pc == '.' || pc == s.charAt(i - 1)) {
            dp[i][j] = dp[i - 1][j - 1];
        } else if (pc == '*') {
            dp[i][j] = dp[i][j - 2];                       // zero occurrences
            char prev = p.charAt(j - 2);
            if (prev == '.' || prev == s.charAt(i - 1)) {
                dp[i][j] |= dp[i - 1][j];                  // one+ occurrences
            }
        }
    }
}
return dp[m][n];
```

**Variations:**
- **Distinct Subsequences**: `dp[i][j] =dp[i-1][j-1] + dp[i-1][j]` on match, else `dp[i-1][j]`.
- **Interleaving String**: 2D dp over prefixes of s1 and s2 verifying s3.
- **Min ASCII Delete Sum**: same skeleton as edit distance, weights by ASCII value.

**Common bugs:**
- **Off-by-one**: dp is sized `[m+1][n+1]`, but `charAt(i-1)` indexes into the string. Mixing 1-based dp with 0-based strings is the #1 bug.
- Forgetting base case (`dp[0][0]`, the empty-prefix row/column).
- Trying to reduce to 1D dp on a problem that needs the diagonal (`dp[i-1][j-1]`) - cache the diagonal in a temp variable.

---

## 6. Palindrome Patterns (Expand Around Center / DP / Manacher's)

**When to use it:** Any problem asking about palindromic substrings or subsequences.

**Trigger phrases:** "longest palindromic substring", "count palindromic substrings", "palindrome partitioning", "palindrome pairs", "shortest palindrome".

**Anchor problem:** Longest Palindromic Substring.

**Template (Expand Around Center - O(n^2), easy to write):**
```java
public String longestPalindrome(String s) {
    int start = 0, maxLen = 0;
    for (int i = 0; i < s.length(); i++) {
        int len1 = expand(s, i, i);         // odd-length center
        int len2 = expand(s, i, i + 1);     // even-length center
        int len = Math.max(len1, len2);
        if (len > maxLen) {
            maxLen = len;
            start = i - (len - 1) / 2;
        }
    }
    return s.substring(start, start + maxLen);
}

private int expand(String s, int l, int r) {
    while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) { l--; r++; }
    return r - l - 1;
}
```

**Template (Count Palindromic Substrings - same expand, just count):**
```java
int count = 0;
for (int i = 0; i < s.length(); i++) {
    count += expand(s, i, i) + expand(s, i, i + 1);  // each returns 1 per palindrome found
}
```
(Modify `expand` to return count instead of length: `while (...) { count++; l--; r++; } return count;`)

**Template (Longest Palindromic Subsequence - DP, O(n^2)):**
```java
int n = s.length();
int[][] dp = new int[n][n];
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

**Variations:**
- **Palindrome Partitioning** (return all): backtrack, check palindrome on each substring choice.
- **Palindrome Partitioning II** (min cuts): DP - precompute `isPalin[i][j]`, then `cuts[i] = min over j` where `s[i..j]` is palindrome.
- **Shortest Palindrome** (prepend to make palindrome): KMP on `s + "#" + reverse(s)`, find LPS to know longest palindrome prefix.
- **Manacher's Algorithm** (O(n) longest palindrome): rarely required; mention you know it exists. Insert `#` between every char to unify odd/even cases.

**Common bugs:**
- Mixing length and index: `start = i - (len - 1) / 2` is the correct formula (you can re-derive: center at i, half-length is `(len-1)/2` left of center).
- For LPS-on-subsequence, forgetting `dp[i][i] = 1` base case.
- Manacher's: forgetting to clamp the mirror radius.

---

## 7. Backtracking on Strings

**When to use it:** Generate all valid strings, partitions, combinations, IP addresses, expressions.

**Trigger phrases:** "generate all valid parentheses", "letter combinations of phone number", "restore IP addresses", "word break II (return all sentences)", "palindrome partitioning", "remove invalid parentheses".

**Anchor problem:** Generate Parentheses.

**Template (Generate Parentheses):**
```java
public List<String> generateParenthesis(int n) {
    List<String> out = new ArrayList<>();
    backtrack(out, new StringBuilder(), 0, 0, n);
    return out;
}

private void backtrack(List<String> out, StringBuilder cur, int open, int close, int n) {
    if (cur.length() == 2 * n) {
        out.add(cur.toString());
        return;
    }
    if (open < n) {
        cur.append('(');
        backtrack(out, cur, open + 1, close, n);
        cur.deleteCharAt(cur.length() - 1);            // undo
    }
    if (close < open) {
        cur.append(')');
        backtrack(out, cur, open, close + 1, n);
        cur.deleteCharAt(cur.length() - 1);            // undo
    }
}
```

**Template (Restore IP Addresses):**
```java
private void backtrack(String s, int start, List<String> path, List<String> out) {
    if (path.size() == 4) {
        if (start == s.length()) out.add(String.join(".", path));
        return;
    }
    for (int len = 1; len <= 3 && start + len <= s.length(); len++) {
        String part = s.substring(start, start + len);
        if (isValid(part)) {
            path.add(part);
            backtrack(s, start + len, path, out);
            path.remove(path.size() - 1);
        }
    }
}
private boolean isValid(String p) {
    if (p.length() > 1 && p.charAt(0) == '0') return false;     // no leading zero
    int n = Integer.parseInt(p);
    return n >= 0 && n <= 255;
}
```

**Variations:**
- **Letter Combinations of Phone Number**: map each digit to letters, recurse per digit.
- **Word Break II**: backtrack with memo, or DP first to know feasibility before backtracking.
- **Remove Invalid Parentheses**: BFS (level = number of removals) typically better than backtrack.

**Common bugs:**
- Forgetting to **undo** (the line after the recursive call) - `StringBuilder` state leaks across branches.
- Using `String` concatenation in the recursive call - O(n) copies per call, blows up complexity. Use `StringBuilder` + undo.
- Adding `cur` (the same SB reference) to the output list - everyone shares the same string state. Must `cur.toString()`.

---

## 8. Trie / Prefix Tree (specialized for strings)

**When to use it:** Prefix-based queries - autocomplete, word-search on a grid (with dictionary), longest common prefix among many words.

**Trigger phrases:** "implement Trie", "autocomplete", "word search II (with dictionary)", "starts with prefix", "replace words", "design search autocomplete".

**Anchor problem:** Implement Trie (Prefix Tree).

**Template:**
```java
class TrieNode {
    TrieNode[] children = new TrieNode[26];          // for lowercase a-z
    boolean isWord = false;
}

class Trie {
    private final TrieNode root = new TrieNode();

    public void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.isWord = true;
    }

    public boolean search(String word)   { TrieNode n = walk(word); return n != null && n.isWord; }
    public boolean startsWith(String pre) { return walk(pre) != null; }

    private TrieNode walk(String s) {
        TrieNode node = root;
        for (char c : s.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return null;
            node = node.children[idx];
        }
        return node;
    }
}
```

**Variations:**
- **Word Search II** (find all dictionary words in a grid): build trie of words, DFS the grid with trie node pointer alongside; prune when no child exists.
- **Replace Words**: trie of roots; for each word, walk char-by-char and return on first `isWord = true`.
- **Design Autocomplete**: store top-K sentences at each node, or DFS subtree on query.

**Common bugs:**
- Forgetting to mark `isWord = true` at the end of insert - `search` always returns false even when prefix exists.
- Using `Map<Character, TrieNode>` when `TrieNode[26]` would do (memory + cache hit).
- For unicode, the array approach fails - must use map.

---

## 9. Pattern Matching - KMP / Rabin-Karp

**When to use it:** Find one (or all) occurrence(s) of pattern P in text T. Naive is O(n*m); KMP / Z / Rabin-Karp are O(n+m).

**Trigger phrases:** "implement strStr()", "find pattern", "shortest period of string", "repeated substring pattern", "longest happy prefix" (LPS itself).

**Anchor problem:** Implement strStr() (substring search).

**Template (KMP - build LPS, then match):**
```java
public int strStr(String haystack, String needle) {
    int m = needle.length(), n = haystack.length();
    if (m == 0) return 0;
    int[] lps = buildLps(needle);

    int i = 0, j = 0;
    while (i < n) {
        if (haystack.charAt(i) == needle.charAt(j)) {
            i++; j++;
            if (j == m) return i - m;
        } else if (j > 0) {
            j = lps[j - 1];                    // fall back in pattern, NOT in text
        } else {
            i++;
        }
    }
    return -1;
}

private int[] buildLps(String p) {
    int m = p.length();
    int[] lps = new int[m];
    int len = 0, i = 1;
    while (i < m) {
        if (p.charAt(i) == p.charAt(len)) {
            lps[i++] = ++len;
        } else if (len > 0) {
            len = lps[len - 1];
        } else {
            lps[i++] = 0;
        }
    }
    return lps;
}
```

**Variations:**
- **Rabin-Karp** (rolling hash): compute hash of pattern once, slide hash across text. Faster in practice for multi-pattern search; risk of hash collision (use two hashes for safety).
- **Repeated Substring Pattern** (`abab` is `ab` repeated): trick - `s` is repeated iff `(s + s).substring(1, 2n - 1).contains(s)`.
- **Shortest Palindrome**: combine KMP on `s + "#" + reverse(s)`; LPS of result gives longest palindromic prefix.
- **Longest Happy Prefix**: literally the last value of the LPS array.

**Common bugs:**
- Off-by-one in LPS construction (1-indexed vs 0-indexed).
- Falling back too far on mismatch - must use `lps[j - 1]`, not `lps[j]`.
- For Rabin-Karp: forgetting to handle hash collisions with a verification step.

---

## 10. Greedy on Strings

**When to use it:** Pick characters in a way that satisfies a global property - reorganize to avoid adjacent duplicates, partition into labels of non-overlapping characters, schedule with cooldown.

**Trigger phrases:** "reorganize string so no two adjacent equal", "partition labels", "task scheduler", "remove K digits to make smallest number", "smallest subsequence of distinct chars".

**Anchor problem:** Reorganize String.

**Template (Reorganize String - max-heap by frequency):**
```java
public String reorganizeString(String s) {
    int[] freq = new int[26];
    for (char c : s.toCharArray()) freq[c - 'a']++;

    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> b[1] - a[1]);   // [char, count]
    for (int i = 0; i < 26; i++) if (freq[i] > 0) pq.add(new int[]{i, freq[i]});

    StringBuilder sb = new StringBuilder();
    int[] prev = null;
    while (!pq.isEmpty()) {
        int[] cur = pq.poll();
        sb.append((char) (cur[0] + 'a'));
        cur[1]--;
        if (prev != null && prev[1] > 0) pq.add(prev);
        prev = cur;
    }
    return sb.length() == s.length() ? sb.toString() : "";    // not possible if leftover
}
```

**Template (Partition Labels):**
```java
public List<Integer> partitionLabels(String s) {
    int[] lastIndex = new int[26];
    for (int i = 0; i < s.length(); i++) lastIndex[s.charAt(i) - 'a'] = i;

    List<Integer> out = new ArrayList<>();
    int start = 0, end = 0;
    for (int i = 0; i < s.length(); i++) {
        end = Math.max(end, lastIndex[s.charAt(i) - 'a']);
        if (i == end) {                          // partition closes
            out.add(end - start + 1);
            start = i + 1;
        }
    }
    return out;
}
```

**Variations:**
- **Task Scheduler**: same heap-pick pattern; cooldown adds idle slots.
- **Remove K Digits**: monotonic stack - pop while top > current and removals remain.
- **Smallest Subsequence of Distinct Characters**: stack + remaining-count + already-in-stack set.

**Common bugs:**
- For reorganize: forgetting to put `prev` back into heap AFTER picking next - otherwise you alternate only two letters.
- For partition labels: confusing `end` vs `lastIndex[c]` - you track the FARTHEST end seen so far, not the current char's last.
- Greedy without proof: always sanity-check that local optimum aligns with global. Many string problems look greedy but actually need DP.

---

## 11 (bonus). Number-String Conversion

**When to use it:** Conversion between numeric and textual representations - Roman numerals, integer to/from string (atoi), integer to English words, "count and say".

**Trigger phrases:** "Roman to integer", "integer to Roman", "atoi", "integer to English words", "count and say", "addStrings (big int as string)".

**Anchor problem:** Roman to Integer.

**Template (Roman to Integer - subtractive trick):**
```java
Map<Character, Integer> map = Map.of(
    'I', 1, 'V', 5, 'X', 10, 'L', 50,
    'C', 100, 'D', 500, 'M', 1000
);
int total = 0;
for (int i = 0; i < s.length(); i++) {
    int cur = map.get(s.charAt(i));
    int next = (i + 1 < s.length()) ? map.get(s.charAt(i + 1)) : 0;
    total += (cur < next) ? -cur : cur;           // subtractive case
}
return total;
```

**Template (atoi - state machine simplified):**
```java
public int myAtoi(String s) {
    int i = 0, n = s.length(), sign = 1;
    long result = 0;
    while (i < n && s.charAt(i) == ' ') i++;
    if (i < n && (s.charAt(i) == '+' || s.charAt(i) == '-')) {
        sign = s.charAt(i++) == '-' ? -1 : 1;
    }
    while (i < n && Character.isDigit(s.charAt(i))) {
        result = result * 10 + (s.charAt(i++) - '0');
        if (sign * result > Integer.MAX_VALUE) return Integer.MAX_VALUE;
        if (sign * result < Integer.MIN_VALUE) return Integer.MIN_VALUE;
    }
    return (int) (sign * result);
}
```

**Variations:**
- **Integer to Roman**: greedy - try largest symbols first, including subtractive (`IV`, `IX`, etc.) in the symbol table.
- **Count and Say**: build iteratively - run-length encode previous term.
- **Add Strings / Multiply Strings**: simulate long multiplication, store digits in `char[]` or `int[]`.
- **Integer to English Words**: split into groups of three digits (thousands, millions), use lookup tables.

**Common bugs:**
- **Overflow**: always cast to `long` (or check before multiply) when accumulating digits. `Integer.MAX_VALUE = 2_147_483_647` is 10 digits - one extra digit overflows.
- For atoi: forgetting whitespace stripping, or accepting `"+-5"` (multiple signs).
- For Roman: not handling the subtractive case (treating `IV` as `1 + 5 = 6` instead of `-1 + 5 = 4`).

---

## Suggested revision order if relearning from scratch

1. **Two Pointers** -> trivially extends to many problems; warmup.
2. **Sliding Window** -> the workhorse for "substring with property X". Master both variable and fixed flavors.
3. **Frequency Map** -> often used together with sliding window.
4. **Stack-based Parsing** -> parentheses, decode, simplify - 5 problems, one template.
5. **Palindromes (Expand Around Center)** -> easy once two-pointers is internalized.
6. **DP on Strings** -> heavy lifting (edit distance, LCS, regex). Take your time.
7. **Backtracking** -> mechanical once template is memorized; focus on undo discipline.
8. **Trie** -> niche but interview-popular for autocomplete / word-search-II.
9. **KMP** -> harder; in interviews, "I know KMP exists, can derive LPS" is often enough.
10. **Greedy** -> last because correctness is the hardest part.
11. **Number-String Conversion** -> easy bonus, off-by-one + overflow are the only risks.

**Universal traps across every pattern:**
- Confusing **substring** (contiguous) with **subsequence** (not necessarily contiguous). Read the problem twice.
- Using `s += c` in a loop = O(n^2). Always `StringBuilder` for mutation.
- `String.charAt(i)` vs `s.toCharArray()[i]`: charAt is O(1) and doesn't allocate; toCharArray allocates an n-byte array once. Use charAt for read-only loops; use toCharArray when you need to mutate.
- `String.equals` vs `==`: `==` compares references. Always `.equals` for string content.
- `'a' - 'A' = 32`: case-conversion arithmetic. `Character.toLowerCase()` is clearer and handles unicode.
- For unicode (non-ASCII), `s.length()` returns CODE UNITS, not characters. Surrogate pairs count as 2. Use `s.codePoints()` for true character iteration.
