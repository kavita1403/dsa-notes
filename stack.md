# Stack Patterns — Master Reference Note

```java
// Java's inbuilt stack support — pick ONE of these, not java.util.Stack, in new code
Deque<Integer> stack = new ArrayDeque<>(); // preferred: push(), pop(), peek(), isEmpty()

// Legacy option — still shows up in old code / interviews may ask "what's wrong with this?"
Stack<Integer> legacyStack = new Stack<>(); // extends Vector, synchronized, avoid in new code
```

---

# Part 1 — Foundations (read this before Part 2)

## Why does Stack exist as a data structure?

A stack models exactly one idea: **the last thing you did is the first thing you undo.**
LIFO — Last In, First Out. Picture a stack of plates: you can only take the top one off,
and you can only add a new one on top. You cannot pull a plate from the middle without
disturbing everything above it — and that restriction is the entire point. It forces a
strict, reversible order of operations, which turns out to be exactly the shape a huge
class of real problems need.

**The core insight:** any time a problem needs you to "remember what came before, in
reverse order, so you can undo/close/resolve it later" — that's a stack, even if the
problem statement never uses the word.

## Real-world examples (where this already runs, right now, without you thinking about it)

1. **The function call stack itself.** Every function call your program makes pushes a
   stack frame (return address, local variables); returning from the function pops it.
   Recursion **is** a stack under the hood — this is exactly why infinite/too-deep
   recursion throws `StackOverflowError` in Java, not `OutOfMemoryError` on the heap.
2. **Browser back button.** Each page visited is pushed; hitting "back" pops.
3. **Undo/Redo (Ctrl+Z) in editors.** Every edit action is pushed onto an undo stack;
   undo pops and reverses it, and can push onto a redo stack.
4. **Bracket/syntax matching in IDEs.** Your editor highlighting a matching `{`/`}` when
   you place your cursor next to one is a live bracket-matching stack.
5. **Expression evaluation in compilers and calculators.** Parsing `(3 + (4 * 5))` or
   converting infix → postfix relies on a stack to track operator precedence and nesting.
6. **Backtracking algorithms** (maze solving, N-Queens, Sudoku). Push a choice; if it
   dead-ends, pop it and try the next option. Recursive backtracking uses the implicit
   call stack; iterative backtracking uses an explicit one.
7. **Depth-First Search (DFS)** on trees/graphs — either the implicit recursive call
   stack, or an explicit `Deque` when you convert DFS to an iterative form (see graphs
   notes for the recursion → iteration conversion in general).

## Java's inbuilt stack-related tools

### 1. `java.util.Stack<T>` — legacy, know it but don't reach for it

- Extends `Vector` (array-based, resizable, **synchronized**).
- Methods: `push(e)`, `pop()`, `peek()`, `empty()`, `search(e)`.
- **Why it's discouraged:** synchronization overhead you almost never need in
  single-threaded algorithm code, and extending `Vector` is a well-known Java API
  design mistake — `Vector` itself is a legacy class. The official Java docs literally
  recommend `Deque` instead.
- Interviewers sometimes ask "what's wrong with `java.util.Stack`?" — this is the answer.

### 2. `Deque<T>` via `ArrayDeque<T>` — the modern default choice

- "Double-Ended Queue," but using only one end makes it a perfect, faster stack.
- Stack-shaped methods: `push(e)` (adds to head), `pop()` (removes from head),
  `peek()` (look at head without removing), `isEmpty()`, `size()`.
- **Why preferred:** not synchronized (no overhead you don't need), no `Vector`
  baggage, resizable circular-array backing with better cache locality/performance than
  a linked structure.
- Bonus: the same object can act as a queue too (`offer()`/`poll()`), so `ArrayDeque` is
  genuinely the most versatile default collection to reach for.

### 3. `LinkedList<T>` (also implements `Deque`)

- Works as a stack too (`push()`/`pop()`), but `ArrayDeque` is generally faster because
  it avoids the per-node object allocation overhead a linked list pays on every insert.
- Only reach for `LinkedList` if you also need list-splicing behavior elsewhere in the
  same object — not for plain stack use.

### Quick comparison table

| Class | push | pop | peek | Thread-safe? | Recommended? |
|---|---|---|---|---|---|
| `Stack` | `push()` | `pop()` | `peek()` | Yes (synchronized) | No — legacy |
| `ArrayDeque` | `push()` | `pop()` | `peek()` | No | **Yes — default choice** |
| `LinkedList` (as Deque) | `push()` | `pop()` | `peek()` | No | Only if you need list ops too |

### Gotcha worth memorizing: exception types differ on empty pop/peek

```java
Stack<Integer> s = new Stack<>();
s.pop(); // throws java.util.EmptyStackException

Deque<Integer> d = new ArrayDeque<>();
d.pop(); // throws java.util.NoSuchElementException — DIFFERENT exception type!
```
Always guard with `isEmpty()` before `pop()`/`peek()` in loops — it's the single most
common stack-pattern bug (see "Common pitfalls" in every pattern below).

---

# Part 2 — Stack Patterns (Master Reference)

## Quick Lookup Table (read this first when revisiting after a gap)

| # | Pattern | Trigger phrases | Anchor problem | Core idea |
|---|---|---|---|---|
| 1 | Valid Parentheses / Bracket Matching | "valid parentheses", "balanced brackets" | Valid Parentheses | Push openers, pop-and-match on closers |
| 2 | Monotonic Stack (Next Greater/Smaller) | "next greater element", "daily temperatures" | Next Greater Element I | Maintain an increasing/decreasing stack, pop while the invariant is violated |
| 3 | Min Stack / Max Stack | "design a stack with getMin in O(1)" | Min Stack | Auxiliary stack tracks the running min alongside the main stack |
| 4 | Expression Evaluation (Postfix/Infix/Calculator) | "evaluate expression", "reverse polish notation", "basic calculator" | Evaluate Reverse Polish Notation | Stack holds operands (and sometimes operators), respect precedence/parens |
| 5 | Largest Rectangle in Histogram | "largest rectangle in histogram", "maximal rectangle" | Largest Rectangle in Histogram | Monotonic increasing stack of **indices** |
| 6 | Stock Span / Previous Greater Element | "stock span problem" | Online Stock Span | Monotonic decreasing stack of (value, count) or indices |
| 7 | Iterative Tree/Graph DFS | "iterative preorder/inorder/postorder", "avoid recursion" | Binary Tree Inorder Traversal (Iterative) | Explicit stack simulates the call stack |
| 8 | Queue via Two Stacks | "implement queue using stacks" | Implement Queue using Stacks | Input stack + output stack, amortized O(1) |
| 9 | Iterative Backtracking with Explicit Stack | "iterative backtracking", "avoid stack overflow in DFS" | N-Queens (iterative) | Push state before a choice; pop to undo/backtrack |
| 10 | Trapping Rain Water (stack variant) | "trapping rain water" | Trapping Rain Water | Monotonic decreasing stack of indices tracks bounded water |
| 11 | Sliding Window Maximum (Monotonic Deque) | "max in every window of size k" | Sliding Window Maximum | Deque holds decreasing values; drop stale front indices |
| 12 | Nested Structure Parsing (Decode String) | "decode string", "flatten nested list" | Decode String | Stack of (count, partial-string) frames, one per nesting level |
| 13 | Simplify Path / Undo-Style Stack | "simplify unix path" | Simplify Path | Push valid segments, pop on `".."` |

---

## 1. Valid Parentheses / Bracket Matching

**When to use it:** Any problem that needs to verify nesting/closing order of paired
symbols — brackets, tags, quotes.

**Trigger phrases:** "valid parentheses", "balanced brackets", "every open has a matching close".

**Anchor problem:** Valid Parentheses.

**Template:**
```java
boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> pairs = Map.of(')', '(', ']', '[', '}', '{');

    for (char c : s.toCharArray()) {
        if (pairs.containsValue(c)) {
            stack.push(c); // opener
        } else if (pairs.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != pairs.get(c)) return false; // closer mismatch
        }
    }
    return stack.isEmpty(); // no leftover unmatched openers
}
```

**Why this shape:** The most recently opened bracket must be the next one closed — that
*is* the LIFO property, stated directly as the problem's rule.

**Edge cases:**
- Empty string → valid (`true`).
- String of only openers (`"((("`) → stack never empties → `false`.
- A closer with nothing on the stack (`")"`) → must check `isEmpty()` *before* popping.

**Common pitfalls:**
- Popping without checking `isEmpty()` first → exception instead of a clean `false`.
- Forgetting the final `stack.isEmpty()` check — a string like `"((()"` passes every
  individual match but leaves an unmatched opener.

**Example problems:** Valid Parentheses, Remove Invalid Parentheses, Minimum Add to Make Parentheses Valid, Check if word is valid after substitutions.

---

## 2. Monotonic Stack (Next Greater / Next Smaller Element)

**When to use it:** "For each element, find the next element to its [left/right] that is
[greater/smaller]" — any variant of that sentence.

**Trigger phrases:** "next greater element", "next smaller element", "daily temperatures", "next warmer day".

**Anchor problem:** Next Greater Element I / Daily Temperatures.

**Template (next greater element, scanning left→right, stack holds indices):**
```java
int[] nextGreaterElement(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>(); // holds INDICES, values decreasing bottom→top

    for (int i = 0; i < n; i++) {
        // current element is bigger than what's on top → it's the "next greater" for those
        while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
            result[stack.pop()] = nums[i];
        }
        stack.push(i);
    }
    return result;
}
```

**Why this shape:** The stack only ever holds elements that are still "waiting" for
their next-greater. When a bigger element shows up, it resolves every smaller element
still waiting on the stack in one pass — this is exactly why it's O(n), not O(n²): every
index is pushed once and popped at most once.

**Edge cases:**
- No greater element exists for the top elements → they stay on the stack and keep
  their default (`-1`).
- Circular array variant (e.g. Next Greater Element II) → iterate `2*n` times using `i % n`.

**Common pitfalls:**
- Storing values instead of indices when you need to write back into a `result` array
  by position.
- Getting the comparison direction backwards (next *greater* vs next *smaller* flips
  `<` to `>` in the `while` condition).

**Example problems:** Next Greater Element I/II, Daily Temperatures, Next Smaller Element, Sum of Subarray Minimums.

---

## 3. Min Stack / Max Stack (O(1) getMin/getMax)

**When to use it:** "Design a stack that supports push, pop, top, and retrieving the
minimum element in constant time."

**Trigger phrases:** "getMin in O(1)", "design a stack with min/max tracking".

**Anchor problem:** Min Stack.

**Template:**
```java
class MinStack {
    private final Deque<Integer> stack = new ArrayDeque<>();
    private final Deque<Integer> minStack = new ArrayDeque<>(); // parallel stack of running mins

    public void push(int val) {
        stack.push(val);
        minStack.push(minStack.isEmpty() ? val : Math.min(val, minStack.peek()));
    }

    public void pop() {
        stack.pop();
        minStack.pop(); // pop in lockstep — keeps the two stacks aligned
    }

    public int top() { return stack.peek(); }
    public int getMin() { return minStack.peek(); }
}
```

**Why this shape:** A single running "current min" variable breaks the moment you pop
the element that *was* the min — you'd have no way to know the previous min. A parallel
stack fixes this by remembering the min *as of every push*, so popping automatically
"restores" the correct previous minimum.

**Edge cases:**
- Pushing a value equal to the current min — still push it onto `minStack` (don't skip
  it), otherwise a later pop desyncs the two stacks.
- Popping down to empty — both stacks should empty together.

**Common pitfalls:**
- Keeping only one min value instead of a full parallel stack — works until you pop the min.
- Letting `stack` and `minStack` sizes drift out of sync (always push/pop both together).

**Example problems:** Min Stack, Max Stack, Design a stack with increment operation.

---

## 4. Expression Evaluation (Postfix / Infix / Basic Calculator)

**When to use it:** Evaluating or converting arithmetic expressions — calculators,
compilers, RPN calculators.

**Trigger phrases:** "evaluate expression", "reverse polish notation", "basic calculator", "infix to postfix".

**Anchor problem:** Evaluate Reverse Polish Notation.

**Template (postfix/RPN evaluation — the simplest form to anchor on):**
```java
int evalRPN(String[] tokens) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (String token : tokens) {
        if (isOperator(token)) {
            int b = stack.pop(); // second operand comes off first
            int a = stack.pop();
            stack.push(apply(a, b, token));
        } else {
            stack.push(Integer.parseInt(token));
        }
    }
    return stack.pop();
}

boolean isOperator(String t) { return t.equals("+") || t.equals("-") || t.equals("*") || t.equals("/"); }

int apply(int a, int b, String op) {
    return switch (op) {
        case "+" -> a + b;
        case "-" -> a - b;
        case "*" -> a * b;
        case "/" -> a / b;
        default -> throw new IllegalArgumentException(op);
    };
}
```

**Why this shape:** Postfix notation removes the need for precedence rules entirely —
operators apply to whatever's directly beneath them on the stack, in the order written.
For infix expressions (`Basic Calculator I/II`), the stack instead holds partial results
and pending signs/operators, pushed/popped as parentheses open and close.

**Edge cases:**
- Operand order for non-commutative ops (`-`, `/`): pop `b` first, then `a`, compute `a op b`.
- Division truncation direction (Java integer division truncates toward zero — matches
  most RPN problem specs, but double-check).
- Multi-digit numbers and negative numbers when parsing infix expressions character by character.

**Common pitfalls:**
- Swapping operand order on subtraction/division (`b - a` instead of `a - b`).
- Off-by-one errors when parsing multi-character numbers out of a raw infix string.

**Example problems:** Evaluate Reverse Polish Notation, Basic Calculator I/II/III, Decode Ways-adjacent parsing problems.

---

## 5. Largest Rectangle in Histogram

**When to use it:** "Find the largest rectangular area" in a set of bars/heights.

**Trigger phrases:** "largest rectangle in histogram", "maximal rectangle in a binary matrix".

**Anchor problem:** Largest Rectangle in Histogram.

**Template:**
```java
int largestRectangleArea(int[] heights) {
    Deque<Integer> stack = new ArrayDeque<>(); // indices, heights increasing bottom→top
    int maxArea = 0;
    int n = heights.length;

    for (int i = 0; i <= n; i++) {
        int h = (i == n) ? 0 : heights[i]; // sentinel 0 forces a final flush
        while (!stack.isEmpty() && heights[stack.peek()] > h) {
            int height = heights[stack.pop()];
            int width = stack.isEmpty() ? i : i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }
        stack.push(i);
    }
    return maxArea;
}
```

**Why this shape:** For every bar, its maximal rectangle extends left/right until it
hits a shorter bar. The monotonic increasing stack of indices lets you compute, in O(n)
total, exactly how far each bar could stretch — the moment a shorter bar appears, every
taller bar on the stack has its rectangle's right boundary finalized.

**Edge cases:**
- All bars the same height → the whole array is one rectangle.
- The sentinel `0` appended at the end is essential — without it, bars still on the
  stack at the end never get their rectangle computed.

**Common pitfalls:**
- Forgetting the sentinel pass (`i <= n`), silently dropping the last rectangle(s).
- Computing `width` incorrectly when the stack becomes empty mid-pop (use `i` directly, not `i - (-1) - 1`… handle the empty case explicitly).

**Example problems:** Largest Rectangle in Histogram, Maximal Rectangle (2D, run this per row).

---

## 6. Stock Span / Previous Greater Element

**When to use it:** "For each day, count how many consecutive previous days had a
value ≤ today's value" — a mirror image of pattern #2, looking backward instead of forward.

**Trigger phrases:** "stock span problem", "previous greater element".

**Anchor problem:** Online Stock Span.

**Template:**
```java
class StockSpanner {
    private final Deque<int[]> stack = new ArrayDeque<>(); // {price, span}

    public int next(int price) {
        int span = 1;
        while (!stack.isEmpty() && stack.peek()[0] <= price) {
            span += stack.pop()[1]; // absorb the span of everything we're bigger than
        }
        stack.push(new int[]{price, span});
        return span;
    }
}
```

**Why this shape:** Instead of re-scanning backward every time (O(n) per query), each
resolved element's span is "absorbed" into the new top — so total work across all calls
stays O(n) amortized, same accounting argument as the monotonic-stack pattern above.

**Edge cases:**
- Strictly increasing price sequence → span keeps compounding into a single stack entry.
- First price ever seen → span is always `1`.

**Common pitfalls:**
- Using `<` instead of `<=` in the absorb condition — span problems are usually
  defined as "less than or equal," so ties must be absorbed too.

**Example problems:** Online Stock Span, Previous Greater Element, Sum of Subarray Ranges (uses both next/previous greater and smaller).

---

## 7. Iterative Tree/Graph DFS (Simulating the Call Stack)

**When to use it:** Whenever recursion depth risks a `StackOverflowError`, or a problem
explicitly asks for an iterative traversal.

**Trigger phrases:** "iterative preorder/inorder/postorder", "without recursion", "avoid stack overflow".

**Anchor problem:** Binary Tree Inorder Traversal (Iterative).

**Template (iterative inorder — the trickiest of the three to get right):**
```java
List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode curr = root;

    while (curr != null || !stack.isEmpty()) {
        while (curr != null) {      // go as far left as possible, pushing along the way
            stack.push(curr);
            curr = curr.left;
        }
        curr = stack.pop();          // no more left — visit this node
        result.add(curr.val);
        curr = curr.right;           // then explore its right subtree
    }
    return result;
}
```

**Why this shape:** Recursion's implicit call stack does exactly this bookkeeping for
you — "remember where I was, go deeper, come back." An explicit `Deque` replaces that
implicit mechanism one-for-one, which is why this pattern is the direct answer to
"how do I convert my recursive solution to iterative?"

**Edge cases:**
- Empty tree (`root == null`) → loop condition never true, returns empty list immediately.
- Skewed tree (all-left or all-right) → stack depth equals tree height, same as recursion would use.

**Common pitfalls:**
- Preorder/postorder need a *different* push order than inorder — don't copy-paste
  inorder's shape and expect it to work for the other two without adjustment.
- Forgetting `curr = curr.right` after popping — silently skips entire right subtrees.

**Example problems:** Binary Tree Preorder/Inorder/Postorder Traversal (Iterative), Iterative DFS on a general graph, Flatten Binary Tree to Linked List.

---

## 8. Queue via Two Stacks

**When to use it:** "Implement a FIFO queue using only stack operations" — a classic
"invert the data structure" interview question.

**Trigger phrases:** "implement queue using stacks".

**Anchor problem:** Implement Queue using Stacks.

**Template:**
```java
class MyQueue {
    private final Deque<Integer> inStack = new ArrayDeque<>();
    private final Deque<Integer> outStack = new ArrayDeque<>();

    public void push(int x) { inStack.push(x); }

    public int pop() {
        moveIfNeeded();
        return outStack.pop();
    }

    public int peek() {
        moveIfNeeded();
        return outStack.peek();
    }

    public boolean empty() { return inStack.isEmpty() && outStack.isEmpty(); }

    private void moveIfNeeded() {
        if (outStack.isEmpty()) {
            while (!inStack.isEmpty()) outStack.push(inStack.pop()); // reverses order once
        }
    }
}
```

**Why this shape:** Reversing a stack's order once (by dumping it into a second stack)
turns LIFO into FIFO for whatever's already been moved. As long as you only refill
`outStack` when it's empty, every element gets reversed exactly once across its
lifetime — giving amortized O(1) per operation even though a single `pop()` can
occasionally do O(n) work.

**Edge cases:**
- `peek()`/`pop()` called while `outStack` still has leftover elements from a prior
  batch — must NOT move `inStack` again yet (that would reorder things incorrectly).
- `empty()` must check both stacks, not just one.

**Common pitfalls:**
- Moving elements from `inStack` to `outStack` on every call instead of only "if
  needed" — breaks the amortized O(1) guarantee and can reorder elements.

**Example problems:** Implement Queue using Stacks, Implement Stack using Queues (the mirror-image problem).

---

## 9. Iterative Backtracking with an Explicit Stack

**When to use it:** Backtracking problems where recursion depth is a genuine risk, or
when a problem explicitly forbids/discourages recursion.

**Trigger phrases:** "iterative backtracking", "convert this recursive backtracking to iterative".

**Anchor problem:** N-Queens (iterative variant).

**Template (generic shape — push a "state," pop to try the next branch or backtrack):**
```java
void iterativeBacktrack(State initial) {
    Deque<State> stack = new ArrayDeque<>();
    stack.push(initial);

    while (!stack.isEmpty()) {
        State state = stack.pop();

        if (isGoal(state)) {
            recordSolution(state);
            continue; // don't expand further from a completed state
        }
        if (!isValid(state)) continue; // prune — the equivalent of an early "return" in recursion

        for (State next : generateChoices(state)) {
            stack.push(next); // each pushed state is an independent "branch" to try later
        }
    }
}
```

**Why this shape:** Recursive backtracking relies on the call stack to hold "what to
try next if this branch fails." An explicit stack of *complete, independent state
snapshots* replaces that — the key difference from other iterative-conversion patterns
(#7) is that here each stack entry must carry a full, self-contained copy of the state,
since there's no automatic "unwind" the way a return statement gives you in recursion.

**Edge cases:**
- State objects must be immutable or deep-copied before pushing — sharing/mutating one
  state object across pushed branches is the iterative version of the classic recursive
  backtracking mutation bug.
- Very large branching factors can make the explicit stack's memory footprint larger
  than recursion's, since you may have many siblings queued at once instead of one
  active call frame.

**Common pitfalls:**
- Pushing a *reference* to a mutable state and then continuing to mutate it in the loop
  — every pushed "snapshot" ends up pointing at the same final mutated object.

**Example problems:** N-Queens (iterative), Iterative Subsets/Permutations generation, Iterative Sudoku Solver.

---

## 10. Trapping Rain Water (Stack Variant)

**When to use it:** "How much water can be trapped between bars of given heights" —
same setup as Largest Rectangle, different question.

**Trigger phrases:** "trapping rain water".

**Anchor problem:** Trapping Rain Water.

**Template:**
```java
int trap(int[] height) {
    Deque<Integer> stack = new ArrayDeque<>(); // indices, heights decreasing bottom→top
    int water = 0;

    for (int i = 0; i < height.length; i++) {
        while (!stack.isEmpty() && height[stack.peek()] < height[i]) {
            int bottom = stack.pop(); // the "floor" of the trapped water
            if (stack.isEmpty()) break; // no left wall — nothing trapped here
            int left = stack.peek();
            int boundedHeight = Math.min(height[left], height[i]) - height[bottom];
            int width = i - left - 1;
            water += boundedHeight * width;
        }
        stack.push(i);
    }
    return water;
}
```

**Why this shape:** Water at any position is bounded above by the *shorter* of its
nearest taller left and right walls. A monotonic decreasing stack naturally exposes
"the nearest taller bar to the left" the moment a taller bar to the right appears —
exactly the two walls needed to compute one horizontal layer of trapped water at a time.

**Edge cases:**
- Strictly increasing or strictly decreasing heights → traps zero water (no `while`
  body ever executes, or every element found lacks a left wall).
- Flat sections at the same height — handled naturally since `<` (strict) leaves equal
  heights on the stack rather than popping them.

**Common pitfalls:**
- Forgetting the `if (stack.isEmpty()) break;` check after popping the bottom — there's
  no left wall to bound the water in that case, and reading `stack.peek()` would throw.
- This problem also has a simpler two-pointer O(1)-space solution — know both, since
  interviewers may ask you to compare trade-offs (stack: intuitive but O(n) space vs.
  two-pointer: O(1) space, less obvious why it works).

**Example problems:** Trapping Rain Water, Trapping Rain Water II (2D variant, needs a heap instead).

---

## 11. Sliding Window Maximum (Monotonic Deque)

**When to use it:** "Find the max (or min) in every contiguous window of size k" — a
deque-flavored sibling of the monotonic stack patterns above.

**Trigger phrases:** "sliding window maximum", "max of every subarray of size k".

**Anchor problem:** Sliding Window Maximum.

**Template:**
```java
int[] maxSlidingWindow(int[] nums, int k) {
    Deque<Integer> deque = new ArrayDeque<>(); // indices, values DECREASING front→back
    int[] result = new int[nums.length - k + 1];

    for (int i = 0; i < nums.length; i++) {
        if (!deque.isEmpty() && deque.peekFirst() <= i - k) {
            deque.pollFirst(); // front index has fallen out of the window
        }
        while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i]) {
            deque.pollLast(); // anything smaller than nums[i] can never be the answer again
        }
        deque.offerLast(i);

        if (i >= k - 1) result[i - k + 1] = nums[deque.peekFirst()];
    }
    return result;
}
```

**Why this shape:** This is the "both ends" generalization of a monotonic stack — you
need to evict from the *back* (values that can never win, same reasoning as pattern #2)
**and** from the *front* (values that are still the max but have aged out of the
window). Only a `Deque`, not a plain stack, can do both in O(1).

**Edge cases:**
- `k == 1` → every element is its own window's max, result equals input.
- `k == nums.length` → a single output value, the global max.

**Common pitfalls:**
- Using a plain `Deque` as only a stack (one-ended) here won't work — this pattern
  genuinely needs both `pollFirst()`/`peekFirst()` and `pollLast()`/`offerLast()`.
- Checking window expiry with `<=` vs `<` incorrectly — the correct check is
  `deque.peekFirst() <= i - k` (index has fully left the window).

**Example problems:** Sliding Window Maximum, Shortest Subarray with Sum at Least K (monotonic deque on prefix sums).

---

## 12. Nested Structure Parsing (Decode String)

**When to use it:** Parsing strings/lists with nested, bracket-delimited repetition or structure.

**Trigger phrases:** "decode string", "flatten nested list weight sum", "nested brackets with counts".

**Anchor problem:** Decode String (`"3[a2[c]]"` → `"accaccacc"`).

**Template:**
```java
String decodeString(String s) {
    Deque<Integer> countStack = new ArrayDeque<>();
    Deque<StringBuilder> stringStack = new ArrayDeque<>();
    StringBuilder current = new StringBuilder();
    int count = 0;

    for (char c : s.toCharArray()) {
        if (Character.isDigit(c)) {
            count = count * 10 + (c - '0'); // support multi-digit counts
        } else if (c == '[') {
            countStack.push(count);
            stringStack.push(current);
            count = 0;
            current = new StringBuilder();      // start a fresh frame for this nesting level
        } else if (c == ']') {
            int repeat = countStack.pop();
            StringBuilder prev = stringStack.pop();
            current = prev.append(current.toString().repeat(repeat)); // fold this level into its parent
        } else {
            current.append(c);
        }
    }
    return current.toString();
}
```

**Why this shape:** Each `[` opens a new nesting level whose partial result must be
kept separate until its matching `]` tells you how many times to repeat it and fold it
back into the *parent* level's partial string — a second stack (`stringStack`) holding
"the parent's progress so far" is what makes that fold-back possible.

**Edge cases:**
- Nested brackets several levels deep (`"2[3[a]]"`) — each level gets its own stack frame.
- Multi-digit counts (`"12[a]"`) — must accumulate digits, not just take one character.

**Common pitfalls:**
- Only tracking a single count/string instead of a *pair* of parallel stacks — loses
  the parent's partial progress the moment you enter a nested bracket.
- Forgetting to reset `count` to `0` after consuming it at a `[`.

**Example problems:** Decode String, Nested List Weight Sum, Flatten Nested List Iterator, Basic Calculator III (combines this with pattern #4).

---

## 13. Simplify Path / Undo-Style Stack

**When to use it:** Any "process tokens left to right, where certain tokens mean undo
the previous one" problem — filesystem paths are the canonical example.

**Trigger phrases:** "simplify unix-style path", "canonical path".

**Anchor problem:** Simplify Path.

**Template:**
```java
String simplifyPath(String path) {
    Deque<String> stack = new ArrayDeque<>();
    for (String part : path.split("/")) {
        if (part.isEmpty() || part.equals(".")) {
            continue;                 // no-ops
        } else if (part.equals("..")) {
            if (!stack.isEmpty()) stack.pop(); // "undo" — go up one directory if possible
        } else {
            stack.push(part);
        }
    }
    // stack is bottom-to-top == root-to-leaf, so reverse for the final path
    Deque<String> reversed = new ArrayDeque<>();
    while (!stack.isEmpty()) reversed.push(stack.pop());
    return "/" + String.join("/", reversed);
}
```

**Why this shape:** `".."` is *literally* an undo operation — "remove the last thing I
committed to." That is exactly what `pop()` does, which is why this whole class of
"undo the previous step" problems maps directly onto a stack even when the problem
statement never says the word "stack."

**Edge cases:**
- `".."` at the root (`stack` already empty) — must be a no-op, not an error (can't go
  above root).
- Multiple consecutive slashes (`"//"`) — `split("/")` naturally produces empty tokens,
  which the `isEmpty()` check skips.

**Common pitfalls:**
- Popping unconditionally on `".."` without checking `isEmpty()` first.
- Forgetting that a `Deque` used with `push()`/`pop()` builds up in *reverse* order
  relative to how you'll want to print it — needs an explicit reversal step (or use an
  `ArrayList` instead of a second stack if you don't need stack semantics for the output).

**Example problems:** Simplify Path, Basic Calculator (undo-style sign tracking), Backspace String Compare.

---

## My Mistakes (fill in as you go)

_Add your own here when you debug something for more than 10 minutes — write down why
you got it wrong, not just the fix._
