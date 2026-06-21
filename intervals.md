# Interval Patterns — Reference Note

## Quick Lookup Table

| Pattern | Key signal | Sort by | Core idea |
|---|---|---|---|
| Merge / combine overlapping | "merge", "insert into sorted intervals" | start | Sweep once, merge into last if overlapping |
| Greedy interval selection | "max non-overlapping", "min removals" | end | Keep whichever interval ends earliest |
| Counting concurrent intervals | "min rooms", "max overlap at once" | start (+ events/heap) | Sweep-line counter or heap of end-times |
| Heap-based query covering | "smallest interval containing each query" | both intervals & queries | Lazy push + permanent discard on expiry |

**Fast decision rule:** are you *merging/scanning* in time order (sort by start), or *greedily choosing a subset* (sort by end)? Picking the wrong key for the situation is the most common interval-problem bug.

---

## The foundational overlap condition

Two intervals `a = [a1, a2]` and `b = [b1, b2]` overlap if:
```
a1 <= b2  AND  b1 <= a2
```
Derive this by negating "they don't overlap" (`a2 < b1` or `b2 < a1`), rather than memorizing it directly.

**Always check the boundary case explicitly** — does `[1,2]` and `[2,3]` count as overlapping in *this* problem? This single `<` vs `<=` decision is the most common off-by-one bug in the whole topic.

---

## 1. Merge Intervals

**When to use it:** "Merge all overlapping intervals" with a list given (not necessarily sorted).

**Template:**
```java
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0])); // sort by start

    List<int[]> result = new ArrayList<>();
    for (int[] interval : intervals) {
        if (result.isEmpty() || result.get(result.size() - 1)[1] < interval[0]) {
            result.add(interval); // no overlap with last merged interval -> new group
        } else {
            result.get(result.size() - 1)[1] =
                Math.max(result.get(result.size() - 1)[1], interval[1]); // extend last group
        }
    }
    return result.toArray(new int[result.size()][]);
}
```

**Why this shape:** Once sorted by start, any interval that overlaps the current merged group must appear immediately after it in the sorted order — so a single left-to-right sweep, comparing only against the *last* merged group, is enough. No need to compare against every earlier interval.

**Edge cases:** empty input array; a single interval; all intervals overlapping into one giant interval.

**Common pitfalls:** forgetting to sort first; comparing the new interval against the *original* last interval instead of the *updated/extended* end value.

**Example problems:** Merge Intervals.

---

## 2. Insert Interval

**When to use it:** Input is already sorted and non-overlapping; you're inserting exactly one new interval and need the result still sorted/non-overlapping.

**Template (three-zone sweep):**
```java
public int[][] insert(int[][] intervals, int[] newInterval) {
    List<int[]> result = new ArrayList<>();
    int i = 0, n = intervals.length;

    // Zone 1: entirely before -> no overlap possible yet
    while (i < n && intervals[i][1] < newInterval[0]) {
        result.add(intervals[i]);
        i++;
    }

    // Zone 2: overlapping -> keep expanding newInterval itself
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
        newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
        i++;
    }
    result.add(newInterval);

    // Zone 3: entirely after -> no overlap, copy rest as-is
    while (i < n) {
        result.add(intervals[i]);
        i++;
    }
    return result.toArray(new int[result.size()][]);
}
```

**Why this shape:** Because the input is already sorted and non-overlapping, you never need to re-check anything once you've left a zone — zone 1 ends exactly where overlap starts, zone 2 ends exactly where overlap stops, zone 3 is just "copy the rest." This makes it O(n) instead of re-running a full Merge Intervals pass.

**Edge cases:** new interval overlaps nothing (falls cleanly into zone 1 or zone 3); new interval swallows multiple existing intervals entirely; empty input list.

**Common pitfalls:** trying to treat this as "just re-run Merge Intervals on everything" — works, but loses the O(n) zone-splitting benefit; off-by-one between `<` and `<=` at zone boundaries.

**Example problems:** Insert Interval.

---

## 3. Non-overlapping Intervals (Greedy by end time)

**When to use it:** "Minimum number of intervals to remove so the rest don't overlap" — equivalent to "maximum number of non-overlapping intervals you can keep."

**Template:**
```java
public int eraseOverlapIntervals(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1])); // sort by END time

    int removals = 0;
    int lastEnd = Integer.MIN_VALUE;

    for (int[] interval : intervals) {
        if (interval[0] >= lastEnd) {
            lastEnd = interval[1]; // keep this interval
        } else {
            removals++; // conflicts with the one we kept -> remove this one instead
        }
    }
    return removals;
}
```

**Why this shape:** This is the classic "activity selection" greedy proof — picking whichever interval *ends earliest* among conflicting options always leaves the most room for future intervals to also fit. Sorting by start time does **not** give this guarantee, which is why this is the one pattern in this note that deliberately sorts by end instead of start.

**Edge cases:** all intervals identical; all intervals mutually overlapping (answer = n - 1); no overlaps at all (answer = 0).

**Common pitfalls:** sorting by start instead of end — this is the single most common mistake on this exact problem, since most other interval problems train you to default to sort-by-start.

**Example problems:** Non-overlapping Intervals, Maximum Length of Pair Chain.

---

## 4. Meeting Rooms (boolean — can one person attend all meetings?)

**When to use it:** "Can a single person attend every meeting" — just a yes/no overlap check, simplest version of this whole topic.

**Template:**
```java
public boolean canAttendMeetings(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0])); // sort by start

    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] < intervals[i - 1][1]) {
            return false; // overlap found
        }
    }
    return true;
}
```

**Why this shape:** After sorting by start, any overlap *must* show up between consecutive intervals — so checking only adjacent pairs is sufficient.

**Edge cases:** 0 or 1 meetings (trivially true); back-to-back meetings where one starts exactly when another ends — decide if `<` or `<=` matches the problem's definition of "conflict."

**Common pitfalls:** using `<=` instead of `<` (or vice versa) at the touching-endpoint boundary, silently flipping the answer for back-to-back meetings.

**Example problems:** Meeting Rooms.

---

## 5. Meeting Rooms II (sweep line / heap — minimum rooms needed)

**When to use it:** "Minimum number of rooms/resources to handle all meetings" — counting *maximum concurrent* intervals, not counting overlapping pairs.

**Key distinction to hold onto:** this is **not** "how many pairs of meetings conflict." It's "at any single instant, what's the peak number of meetings happening simultaneously."

**Template (events / sweep-line):**
```java
public int minMeetingRooms(int[][] intervals) {
    int n = intervals.length;
    int[][] events = new int[2 * n][2]; // {time, +1 or -1}

    for (int i = 0; i < n; i++) {
        events[2 * i] = new int[]{intervals[i][0], 1};   // start event
        events[2 * i + 1] = new int[]{intervals[i][1], -1}; // end event
    }

    Arrays.sort(events, (a, b) -> {
        if (a[0] != b[0]) return Integer.compare(a[0], b[0]);
        return Integer.compare(a[1], b[1]); // tie: end (-1) before start (+1)
    });

    int rooms = 0, maxRooms = 0;
    for (int[] event : events) {
        rooms += event[1];
        maxRooms = Math.max(maxRooms, rooms);
    }
    return maxRooms;
}
```

**Why this shape:** Every meeting contributes one start (+1) and one end (-1) event. Sorting all events by time and sweeping with a running counter gives, at every instant, exactly how many meetings are concurrently active — the max value that counter ever hits is the minimum rooms needed.

**The tiebreak is not optional — it encodes the problem's own definition of "conflict."** If a meeting ends at the same time another starts (`(0,8)` and `(8,10)`), and the problem says that's *not* a conflict, the end event must be processed before the start event at that tied time — otherwise the counter briefly (and incorrectly) reads 2. Flip the tiebreak if a problem variant *does* consider touching endpoints a conflict.

**Edge cases:** meetings with identical start and end times; meetings touching exactly at the boundary (the tiebreak above).

**Common pitfalls:**
- Counting overlapping *pairs* instead of *peak concurrency* — these give different (wrong) answers, e.g. `[0,30],[5,10],[15,20]` has 2 pairwise overlaps with the same meeting but never needs more than 2 rooms.
- Missing the tie-break entirely in the comparator, which silently breaks back-to-back meetings.

**Example problems:** Meeting Rooms II.

---

## 6. Minimum Interval to Include Each Query (heap + lazy push/permanent discard)

**When to use it:** Given a set of intervals and a set of query points, for each query find the *smallest* interval that contains it.

**Template:**
```java
public int[] minInterval(int[][] intervals, int[] queries) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0])); // sort intervals by start

    int n = queries.length;
    Integer[] queryIndices = new Integer[n];
    for (int i = 0; i < n; i++) queryIndices[i] = i;
    Arrays.sort(queryIndices, (a, b) -> Integer.compare(queries[a], queries[b])); // sort queries, keep original index

    // min-heap keyed by interval length: {length, end}
    PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> Integer.compare(a[0], b[0]));
    int[] result = new int[n];
    int ptr = 0;

    for (int idx : queryIndices) {
        int q = queries[idx];

        // lazy push: add every interval that has started by this query value
        while (ptr < intervals.length && intervals[ptr][0] <= q) {
            heap.offer(new int[]{intervals[ptr][1] - intervals[ptr][0] + 1, intervals[ptr][1]});
            ptr++;
        }

        // permanent discard: remove intervals that have already ended before this query
        while (!heap.isEmpty() && heap.peek()[1] < q) {
            heap.poll();
        }

        result[idx] = heap.isEmpty() ? -1 : heap.peek()[0];
    }
    return result;
}
```

**Why this shape:** Each interval needs to be pushed into the heap **exactly once** (when it starts) and popped-and-discarded **at most once** (when it permanently expires) — across *all* queries combined, not per query. That's what gives the real `O((n + q) log n)` bound instead of accidentally re-checking/re-queueing the same interval repeatedly.

**The critical distinction:** when popping an invalid interval from the heap top, ask whether it's invalid for a *temporary* reason (not started yet — handled by the lazy-push pointer, so it never even enters the heap early) or a *permanent* reason (already ended — must be discarded for good, never pushed back). Conflating these two is the core bug in naive attempts at this problem.

**Edge cases:** a query with no covering interval at all (`-1`); multiple intervals of identical length tied for smallest; query value lower than every interval's start.

**Common pitfalls:** pushing an expired interval back into the heap "to be safe" — this re-introduces the exact wasted, repeated work the heap-discard step is designed to eliminate; forgetting to track each query's original index before sorting, so answers get written to the wrong output position.

**Example problems:** Minimum Interval to Include Each Query.
