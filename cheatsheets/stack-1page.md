# Stack — Cheat Sheet

**Default choice:** `Deque<Integer> stack = new ArrayDeque<>();` — not `java.util.Stack` (legacy, extends `Vector`, synchronized). `push()`/`pop()`/`peek()`/`isEmpty()`.

**Why stack exists:** LIFO — the last thing done is the first thing undone. Function call stack, undo/redo, browser back, backtracking, DFS, expression parsing all reduce to this one shape.

**Gotcha:** `Stack.pop()` on empty throws `EmptyStackException`; `ArrayDeque.pop()` throws `NoSuchElementException`. Always `isEmpty()` check first.

## Valid Parentheses
```java
if (isOpener(c)) stack.push(c);
else if (stack.isEmpty() || stack.pop() != matchFor(c)) return false;
// end: return stack.isEmpty();
```

## Monotonic Stack — Next Greater Element
```java
while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) result[stack.pop()] = nums[i];
stack.push(i);
```
Flip `<` to `>` for Next Smaller. Store indices when you need to write back positions.

## Min Stack
```java
minStack.push(minStack.isEmpty() ? val : Math.min(val, minStack.peek()));
// pop BOTH stacks together, always in lockstep
```

## Expression Eval (RPN)
```java
if (isOperator(tok)) { int b = stack.pop(), a = stack.pop(); stack.push(apply(a, b, tok)); }
else stack.push(Integer.parseInt(tok));
```
Watch operand order on `-`/`/`: pop `b` first, then `a`, compute `a op b`.

## Largest Rectangle in Histogram
```java
while (!stack.isEmpty() && heights[stack.peek()] > h) {
    int height = heights[stack.pop()];
    int width = stack.isEmpty() ? i : i - stack.peek() - 1;
    maxArea = Math.max(maxArea, height * width);
}
stack.push(i);
```
Append a sentinel `0` height at the end to flush everything still on the stack.

## Stock Span / Previous Greater
```java
while (!stack.isEmpty() && stack.peek()[0] <= price) span += stack.pop()[1];
stack.push(new int[]{price, span});
```

## Iterative Inorder Traversal
```java
while (curr != null) { stack.push(curr); curr = curr.left; }
curr = stack.pop(); visit(curr); curr = curr.right;
```

## Queue via Two Stacks
```java
if (outStack.isEmpty()) while (!inStack.isEmpty()) outStack.push(inStack.pop());
return outStack.pop(); // amortized O(1) — only refill outStack when it's empty
```

## Trapping Rain Water
```java
while (!stack.isEmpty() && height[stack.peek()] < height[i]) {
    int bottom = stack.pop();
    if (stack.isEmpty()) break; // no left wall
    int bounded = Math.min(height[stack.peek()], height[i]) - height[bottom];
    water += bounded * (i - stack.peek() - 1);
}
stack.push(i);
```

## Sliding Window Maximum (Monotonic Deque — both ends!)
```java
if (!deque.isEmpty() && deque.peekFirst() <= i - k) deque.pollFirst(); // evict expired front
while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i]) deque.pollLast(); // evict useless back
deque.offerLast(i);
```

## Decode String (nested parsing)
```java
if (c == '[') { countStack.push(count); stringStack.push(current); count = 0; current = new StringBuilder(); }
else if (c == ']') { int r = countStack.pop(); current = stringStack.pop().append(current.toString().repeat(r)); }
```
Two parallel stacks — one for counts, one for the parent's partial string per nesting level.

## Simplify Path (undo-style)
```java
if (part.equals("..")) { if (!stack.isEmpty()) stack.pop(); }
else if (!part.isEmpty() && !part.equals(".")) stack.push(part);
```
`".."` = undo = pop. Guard empty before popping.
