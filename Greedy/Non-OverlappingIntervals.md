# LeetCode 435 — Non-overlapping Intervals



Medium

Greedy + Sorting



This problem is closely related to LC 56 — Merge Intervals, but the objective is different.

- LC 56: Merge overlapping intervals.
- LC 435: Remove the minimum number of intervals so that no intervals overlap.

The key insight is:

When two intervals overlap, keep the interval that ends earlier.

Why? Because it leaves more room for the intervals that come after it.

## 1. Understand the problem

You are given an array of intervals. Return the minimum number of intervals you need to remove so that the remaining intervals are non-overlapping.

Example 1

```
Input:  [[1,2], [2,3], [3,4], [1,3]]

Output: 1
```

Explanation:

Remove `[1,3]`.

Remaining intervals:

```
[1,2], [2,3], [3,4]
```

These do not overlap. Notice that intervals touching at endpoints are allowed here: `[1,2]` and `[2,3]` are non-overlapping.

Example 2

```
Input:  [[1,2], [1,2], [1,2]]

Output: 2
```

We can keep only one interval.

Example 3

```
Input:  [[1,2], [2,3]]

Output: 0
```

They are already non-overlapping.

## 2. Brute-force approach

A brute-force way to think about this problem is to explore all possible subsets of intervals:

- Choose to keep the current interval if it doesn't overlap with the previous kept interval.
- Or remove it.
- Find the maximum number of intervals we can keep.

Then:

\\[ \text{Minimum removals} = n - \text{Maximum intervals kept} \\]

A recursive implementation:

```
class Solution {

    public int eraseOverlapIntervals(int[][] intervals) {
        return intervals.length - findMax(
            intervals, 0, Integer.MIN_VALUE
        );
    }

    private int findMax(int[][] intervals, int index, int lastEnd) {

        // All intervals have been considered.
        if (index == intervals.length) {
            return 0;
        }

        // Option 1: Remove the current interval.
        int remove = findMax(intervals, index + 1, lastEnd);

        // Option 2: Keep it, if it does not overlap.
        int keep = 0;

        if (intervals[index][0] >= lastEnd) {
            keep = 1 + findMax(
                intervals, index + 1, intervals[index][1]
            );
        }

        // Return the maximum number of intervals we can keep.
        return Math.max(remove, keep);
    }
}
```

This recursion assumes intervals are already sorted by start time. Without sorting, it cannot safely make the keep/remove decision in this order.

- Time: \\(O(2^n)\\) after sorting, in the worst case.
- Space: \\(O(n)\\) recursion stack.

The problem is that we explore too many possible combinations.

## 3. Identify the bottleneck

The brute-force solution explores many combinations of keeping and removing intervals.

But we don't actually need to explore every combination.

Consider:

```
[[1,3], [2,4], [3,5]]
```

The first two intervals overlap, and the second and third intervals also overlap.

Which interval should we keep?

- `[1,3]` ends at 3.
- `[2,4]` ends at 4.
- `[3,5]` ends at 5.

Keeping the interval that ends earliest gives us the greatest opportunity to fit more intervals afterward.

## 4. Recognize the pattern

Greedy + Sorting

There are two common ways to solve this problem:

1. Sort by starting point and compare each interval with the previously kept interval.
2. Sort by ending point and greedily keep intervals that do not overlap.

We'll use the second approach because its logic is especially simple.

## 5. Derive the optimal approach

### Step 1: Sort by ending time

```
Arrays.sort(intervals, (a, b) ->
    Integer.compare(a[1], b[1])
);
```

Why ending time?

We want to keep intervals that finish as early as possible, leaving maximum available space for future intervals.

### Step 2: Keep the first interval

The first interval has the earliest ending time, so we keep it.

Maintain:

```
int lastEnd = intervals[0][1];
int removals = 0;
```

### Step 3: Process the remaining intervals

For every interval, compare its starting point with `lastEnd`.

Case A: No overlap

```
if (intervals[i][0] >= lastEnd)
```

Keep the interval and update:

```
lastEnd = intervals[i][1];
```

Case B: Overlap

```
else
```

Remove the current interval:

```
removals++;
```

Why remove the current interval? Because we sorted by ending time, so the previously kept interval ends earlier than or at the same time as the current one. Keeping it is the greedy choice.

Notice: we do not update `lastEnd` when removing an interval.

## 6. Optimal Java solution

```
import java.util.Arrays;

class Solution {
    public int eraseOverlapIntervals(int[][] intervals) {

        // Edge case: zero or one interval needs no removal.
        if (intervals.length <= 1) {
            return 0;
        }

        // Step 1: Sort intervals by their ending points.
        Arrays.sort(intervals, (a, b) ->
            Integer.compare(a[1], b[1])
        );

        // Step 2: Keep the first interval.
        int lastEnd = intervals[0][1];

        // Count intervals that must be removed.
        int removals = 0;

        // Step 3: Process remaining intervals.
        for (int i = 1; i < intervals.length; i++) {

            // Case 1: No overlap, so keep this interval.
            if (intervals[i][0] >= lastEnd) {
                lastEnd = intervals[i][1];
            }

            // Case 2: Overlap, so remove this interval.
            else {
                removals++;
            }
        }

        return removals;
    }
}
```

Complexity

- Time: \\(O(n\log n)\\) — sorting dominates; traversal takes \\(O(n)\\).
- Space: \\(O(1)\\) auxiliary space apart from sorting implementation overhead.

## 7. Detailed dry run

Input:

```
[[1,2], [2,3], [3,4], [1,3]]
```

### Step 1: Sort by ending time

| Interval | Ending time |
| -------- | ----------- |
| `[1,2]`  | 2           |
| `[2,3]`  | 3           |
| `[1,3]`  | 3           |
| `[3,4]`  | 4           |

Sorted array:

```
[[1,2], [2,3], [1,3], [3,4]]
```

### Step 2: Initialize

```
lastEnd = 2;
removals = 0;
```

We keep `[1,2]`.

### Step 3: Iterate

Iteration 1 — current = [2,3]

Check:

```
current start >= lastEnd
     2       >=   2       → true
```

No overlap. Keep `[2,3]`.

```
lastEnd = 3
removals = 0
```

Keep

Iteration 2 — current = [1,3]

Check:

```
current start >= lastEnd
     1       >=   3       → false
```

Overlap! Remove `[1,3]`.

```
lastEnd = 3
removals = 1
```

Remove

Important: `lastEnd` remains 3. We do not update it when removing an interval.

Iteration 3 — current = [3,4]

Check:

```
current start >= lastEnd
     3       >=   3       → true
```

No overlap. Keep `[3,4]`.

```
lastEnd = 4
removals = 1
```

Keep

Final answer:

```
1
```

We removed only `[1,3]`.

### Another important example: Why keep the earlier-ending interval?

Input:

```
[[1,10], [2,3], [4,5]]
```

Sorted by ending time:

```
[[2,3], [4,5], [1,10]]
```

| Current interval | Decision | Reason         |
| ---------------- | -------- | -------------- |
| `[2,3]`          | Keep     | First interval |
| `[4,5]`          | Keep     | `4 >= 3`       |
| `[1,10]`         | Remove   | `1 < 5`        |

Answer: `1`

If we had kept `[1,10]`, we would have lost the opportunity to keep both `[2,3]` and `[4,5]`.

This is the heart of the greedy strategy: finish early, leave room for more intervals.

## 8. Tricky parts and interview insights

### A. Why is the condition `>=` and not `>`?

This is one of the most important differences between LC 56 and LC 435.

| Problem                            | Endpoint rule                              |
| ---------------------------------- | ------------------------------------------ |
| LC 56 — Merge Intervals            | `currentStart <= lastEnd` means overlap    |
| LC 435 — Non-overlapping Intervals | `currentStart >= lastEnd` means no overlap |

For LC 435:

```
[1,2] and [2,3]
```

These intervals are allowed together because one ends exactly when the other begins.

### B. Why don't we update `lastEnd` when removing?

Suppose:

```
last kept interval = [2,5]
current interval   = [3,8]
```

They overlap.

We remove `[3,8]` and retain `[2,5]`.

So:

```
removals++;
```

But `lastEnd` stays `5`.

### C. Could we sort by starting time instead?

Yes! Another valid greedy solution sorts by starting time.

When intervals overlap, keep the one with the smaller ending point:

```
if (intervals[i][0] < lastEnd) {
    removals++;
    lastEnd = Math.min(lastEnd, intervals[i][1]);
} else {
    lastEnd = intervals[i][1];
}
```

This is also \\(O(n\log n)\\).

The ending-time approach is simpler because once an interval is selected, an overlapping later interval can simply be removed.

### D. Correction to the brute-force snippet

The recursive brute-force code in Step 2 needs sorting before calling the helper. Add this to its public method:

```
Arrays.sort(intervals, (a, b) ->
    Integer.compare(a[0], b[0])
);
```

That ensures the recursive keep/remove decisions process intervals in start-time order.

## GitHub-ready notes — LC 435

Problem: Find the minimum number of intervals to remove so that the remaining intervals do not overlap.

Approach:

1. Sort intervals by ending time.
2. Keep the first interval.
3. Track its ending point using `lastEnd`.
4. For each next interval:
   - If `start >= lastEnd`, keep it and update `lastEnd`.
   - Otherwise, remove it and increment `removals`.
5. Return `removals`.

Pattern: Greedy + Sorting (Interval Scheduling)

Time Complexity: \\(O(n\log n)\\)

Space Complexity: \\(O(1)\\) auxiliary, excluding sorting overhead.

Key Insight: To maximize the number of non-overlapping intervals, always prioritize the interval that finishes earliest. Minimum removals = total intervals − maximum intervals kept.

## Test your understanding

Q1. Intervals: [1,5], [2,3]. After sorting by ending time, which interval is kept first?

[1,5]

[2,3]



Q2. lastEnd = 5, current interval = [5,8]. What should happen?

Remove it

Keep it



Q3. lastEnd = 5, current interval = [3,8]. What happens to lastEnd?

Update it to 8

Keep it as 5

Update it to 3



Q4. What is the greedy objective?

Keep the interval that starts earliest

Keep the interval that ends earliest

Keep the interval that is longest



Check answers

Recommended next: LC 452 — Minimum Number of Arrows to Burst Balloons. It uses almost the same greedy idea of sorting by ending time, so it's an excellent follow-up.
