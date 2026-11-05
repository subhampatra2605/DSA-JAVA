# LeetCode 56 — Merge Intervals



Medium

Greedy + Sorting



The main idea: Sort intervals by their starting points. Then compare each interval with the last merged interval. If they overlap, merge them; otherwise, add a new interval.

## 1. Understand the problem

You are given an array of intervals, where each interval is represented as:

`[start, end]`

Merge all overlapping intervals and return the non-overlapping intervals that cover all the original intervals.

Example 1

```
Input:  [[1,3], [2,6], [8,10], [15,18]]

Output: [[1,6], [8,10], [15,18]]
```

Why?

- `[1,3]` and `[2,6]` overlap because `2 <= 3`.
- Their merged interval becomes `[1,6]`.
- `[8,10]` does not overlap with `[1,6]`.
- `[15,18]` does not overlap with `[8,10]`.

Example 2

```
Input:  [[1,4], [4,5]]

Output: [[1,5]]
```

Notice that intervals touching at an endpoint also overlap. We use `nextStart <= currentEnd`.

## 2. Brute-force approach

Before optimizing, think about what we need to do:

1. Pick an interval.
2. Find every interval that overlaps with it.
3. Merge them.
4. Repeat until no more intervals can be merged.

The difficulty is that merging one interval may make it overlap with another interval that did not overlap initially. We may need repeated passes.

A straightforward brute-force implementation can repeatedly scan and merge pairs:

```
import java.util.*;

class Solution {
    public int[][] merge(int[][] intervals) {

        // Keep merging until a complete pass makes no changes.
        List<int[]> list = new ArrayList<>();

        for (int[] interval : intervals) {
            list.add(interval);
        }

        boolean merged = true;

        while (merged) {
            merged = false;

            for (int i = 0; i < list.size(); i++) {
                for (int j = i + 1; j < list.size(); j++) {

                    int[] a = list.get(i);
                    int[] b = list.get(j);

                    // Check whether the two intervals overlap.
                    if (a[0] <= b[1] && b[0] <= a[1]) {

                        // Merge them into one interval.
                        a[0] = Math.min(a[0], b[0]);
                        a[1] = Math.max(a[1], b[1]);

                        // Remove the second interval.
                        list.remove(j);

                        merged = true;
                        break;
                    }
                }

                if (merged) {
                    break;
                }
            }
        }

        return list.toArray(new int[list.size()][]);
    }
}
```

Complexity: This repeated-scanning approach can take \\(O(n^3)\\) time in the worst case due to repeated pair scans and removals. Space is \\(O(n)\\).

This is useful for understanding the merging operation, but we can do much better.

## 3. Identify the bottleneck

The brute-force approach repeatedly checks intervals in arbitrary order.

Consider:

```
[[8,10], [1,3], [2,6], [15,18]]
```

The intervals are not arranged by their starting points.

What if we sort them first?

```
[[1,3], [2,6], [8,10], [15,18]]
```

Now we only need to compare the current interval with the last interval in our result.

That removes the need to repeatedly scan all pairs.

## 4. Recognize the pattern

Sorting + Greedy

Why greedy?

At every step, we make a locally optimal decision:

- If the current interval overlaps with the last merged interval, merge immediately.
- Otherwise, add it as a new interval.

Sorting makes this safe because all intervals are processed in increasing order of their starting points.

## 5. Derive the optimal approach

After sorting by start time, maintain a list called `merged`.

For every interval, compare it with the last interval in `merged`.

There are only two possibilities:

Case 1: They overlap

Condition:

```
current[0] <= last[1]
```

Merge them by extending the end point:

```
last[1] = Math.max(last[1], current[1]);
```

Why `Math.max`? Because the new interval might end earlier or later than the last merged interval.

Case 2: They do not overlap

Condition:

```
current[0] > last[1]
```

Add the current interval as a new interval:

```
merged.add(current);
```

Why do we only compare with the last interval?

Because after sorting, all earlier intervals start at or before the current interval. If the current interval does not overlap with the last merged interval, it cannot overlap with any earlier merged interval either.

## 6. Optimal Java solution

```
import java.util.*;

class Solution {
    public int[][] merge(int[][] intervals) {

        // Step 1: Sort intervals by their starting point.
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);

        // Stores the final merged intervals.
        List<int[]> merged = new ArrayList<>();

        // Step 2: Process each interval.
        for (int[] current : intervals) {

            // If the list is empty, add the first interval.
            if (merged.isEmpty()) {
                merged.add(current);
            }

            else {
                // Get the last interval in the merged list.
                int[] last = merged.get(merged.size() - 1);

                // Case 1: Current interval overlaps with last.
                if (current[0] <= last[1]) {

                    // Extend the end if current ends later.
                    last[1] = Math.max(last[1], current[1]);
                }

                // Case 2: No overlap.
                else {
                    // Add current as a separate interval.
                    merged.add(current);
                }
            }
        }

        // Convert List<int[]> into int[][].
        return merged.toArray(new int[merged.size()][]);
    }
}
```

Complexity

- Time: \\(O(n\log n)\\) — sorting dominates; merging takes \\(O(n)\\).
- Space: \\(O(n)\\) for the output list. Sorting may also use implementation-dependent auxiliary space.

A small improvement: instead of subtracting in the comparator, you can write:

```
Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
```

This avoids possible integer overflow.

## 7. Detailed dry run

Let's trace the most important example:

```
intervals = [[1,3], [2,6], [8,10], [15,18]]
```

After sorting:

```
[[1,3], [2,6], [8,10], [15,18]]
```

Initially:

```
merged = []
```

Iteration 1 — current = [1,3]

`merged` is empty, so add the interval.

Result

[[1,3]]

Iteration 2 — current = [2,6]

Last interval = `[1,3]`

Check overlap:

```
current[0] <= last[1]
     2     <=    3       → true
```

They overlap. Now calculate:

```
last[1] = max(3, 6)
         = 6
```

Result

[[1,6]]

Iteration 3 — current = [8,10]

Last interval = `[1,6]`

Check overlap:

```
8 <= 6 → false
```

No overlap. Add `[8,10]`.

Result

[[1,6], [8,10]]

Iteration 4 — current = [15,18]

Last interval = `[8,10]`

Check overlap:

```
15 <= 10 → false
```

No overlap. Add `[15,18]`.

Result

[[1,6], [8,10], [15,18]]

Final answer:

```
[[1,6], [8,10], [15,18]]
```

### Important dry run: Why do we use `Math.max()`?

Consider:

```
[[1,10], [2,5], [6,12]]
```

After sorting, the order remains the same.

| Current interval | Comparison     | Merged result |
| ---------------- | -------------- | ------------- |
| `[1,10]`         | First interval | `[[1,10]]`    |
| `[2,5]`          | `2 <= 10`      | `[[1,10]]`    |
| `[6,12]`         | `6 <= 10`      | `[[1,12]]`    |

When `[2,5]` is processed, the end must remain `10`, not become `5`.

That's why we write:

```
last[1] = Math.max(last[1], current[1]);
```

We never shrink the already-covered range.

## 8. Tricky parts and interview insights

### A. Why sort by starting point?

Without sorting, an interval that should merge with an earlier interval might appear later, forcing repeated comparisons.

Sorting guarantees that we process intervals from left to right.

### B. Why check only `current[0] <= last[1]`?

After sorting, the current interval's start is already greater than or equal to the starts of all previously processed intervals.

Therefore, if it starts after the last merged interval ends, it cannot overlap with any earlier interval.

### C. Why `<=` and not `<`?

Because touching intervals should merge.

```
[1,4] and [4,5]
```

Since `4 <= 4`, they merge into `[1,5]`.

### D. Why use `merged.get(merged.size() - 1)`?

We only need to inspect the most recently created or updated merged interval. Earlier intervals are already finalized.

### E. What if an interval is completely inside another?

```
[1,10] and [3,7]
```

They overlap, but `Math.max(10,7)` remains `10`.

The result is `[1,10]`.

## GitHub-ready notes — LC 56

Problem: Merge all overlapping intervals and return non-overlapping intervals.

Approach:

1. Sort intervals by their starting point.
2. Create a result list.
3. For every interval, compare it with the last merged interval.
4. If `current[0] <= last[1]`, update the last interval's end using `Math.max()`.
5. Otherwise, add the current interval.
6. Convert the result list into a 2D array.

Pattern: Sorting + Greedy

Time Complexity: \\(O(n\log n)\\)

Space Complexity: \\(O(n)\\) for output.

Key Insights:

- Sorting makes overlapping intervals adjacent.
- Only the last merged interval needs to be checked.
- Use `<=` because touching endpoints count as overlap.
- Use `Math.max()` to ensure the merged range never shrinks.
- The order of intervals in the input does not matter after sorting.

## Test your understanding

Try these without looking at the solution.

Q1. After sorting, merged = [[1,5]]. Current interval = [2,3]. What happens?

Add [2,3] separately

Merge, result stays [[1,5]]

Replace [1,5] with [2,3]

Q2. merged = [[1,4]]. Current interval = [4,8]. What happens?

No overlap

Merge into [1,8]

Keep both intervals

Q3. Why do we sort intervals by their starting points?

To make every interval have the same length

To ensure overlapping intervals are processed next to each other

To eliminate duplicate intervals

Q4. merged = [[1,6], [8,10]]. Current = [9,12]. Which interval do we compare against?

[1,6]

[8,10]

Both

Check answers

Recommended next problem: LC 57 — Insert Interval. It builds directly on LC 56 and tests whether you truly understand interval overlap and merging.
