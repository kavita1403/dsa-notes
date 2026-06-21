# Intervals — Cheat Sheet

**Overlap condition:** `a1 <= b2 AND b1 <= a2`
**Decision fork:** merging/scanning in time order → sort by **start**. Greedily choosing a subset → sort by **end**.

## Merge Intervals
Sort by start → sweep → merge into last if `last.end >= cur.start`, else start new group.

## Insert Interval
Three zones over already-sorted list: before (copy) → overlapping (expand newInterval) → after (copy).

## Non-overlapping Intervals
Sort by **end**. Track `lastEnd`. Keep if `start >= lastEnd`, else count as removal.

## Meeting Rooms
Sort by start. Check adjacent pairs only: `intervals[i].start < intervals[i-1].end` → conflict.

## Meeting Rooms II
Events: start=+1, end=-1. Sort by time, **tiebreak: end before start** (if touching ≠ conflict). Sweep, track max running sum.

## Minimum Interval to Include Each Query
Sort intervals by start, sort queries (keep original index). Min-heap by length. Lazy-push intervals that started; permanently discard (never requeue) intervals that ended before current query.
