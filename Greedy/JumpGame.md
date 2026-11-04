# LeetCode 55 — Jump Game



Medium

Greedy



Pattern: Greedy — Track the Farthest Reachable Index

This problem is a very important Greedy problem for your Infosys SP preparation.

The key question is:

Can we reach the last index by making jumps, where each element tells us the maximum jump length from that position?

Let's understand it step by step.

## 1. Understand the Problem

You are given an integer array `nums`.

You start at the first index, and each element represents the maximum jump length you can make from that position.

Return `true` if you can reach the last index. Otherwise, return `false`.

### Example 1

```
Input: nums = [2,3,1,1,4]
Output: true
```

Explanation:

Start at index `0`.

```
Index:  0  1  2  3  4
Nums:  [2, 3, 1, 1, 4]
```

At index `0`, the maximum jump length is `2`.

We can jump from index `0` to index `1`.

At index `1`, the maximum jump length is `3`.

We can jump directly to index `4`.

```
0 → 1 → 4
```

We reached the last index.

Answer: `true`.

### Example 2

```
Input: nums = [3,2,1,0,4]
Output: false
```

Explanation:

```
Index:  0  1  2  3  4
Nums:  [3, 2, 1, 0, 4]
```

We can reach index `3`, but:

```
nums[3] = 0
```

We cannot move any further.

Index `4` is unreachable.

Answer: `false`.

Important: Each value represents the maximum jump distance, not the exact jump distance. You can jump fewer positions if you want.

## 2. Brute-Force Approach — Recursion

### Idea

At every index, try every possible jump from `1` to `nums[index]`.

For example:

```
nums = [2,3,1,1,4]
```

At index `0`, we can jump:

- 1 position → index `1`
- 2 positions → index `2`

We recursively explore both possibilities.

If any path reaches the last index, return `true`.

### Java Code — Brute Force

```
class Solution {
    public boolean canJump(int[] nums) {
        return canReach(nums, 0);
    }

    private boolean canReach(int[] nums, int index) {

        // Reached or passed the last index.
        if (index >= nums.length - 1) {
            return true;
        }

        // Try every possible jump.
        for (int jump = 1; jump <= nums[index]; jump++) {

            // Explore the next reachable index.
            if (canReach(nums, index + jump)) {
                return true;
            }
        }

        // No path can reach the last index.
        return false;
    }
}
```

### Complexity

- Time: Exponential in the worst case because multiple jump paths are explored repeatedly.
- Space: `O(n)` recursion stack in the worst case.

This approach becomes inefficient for large arrays.

## 3. Identify the Bottleneck

Consider:

```
nums = [2,3,1,1,4]
```

From index `0`, we can reach indices `1` and `2`.

From index `1`, we can reach indices `2`, `3`, and `4`.

From index `2`, we can reach index `3`.

Notice something:

We don't actually need to remember every possible path.

We only need to know:

What is the farthest index we can reach so far?

For example:

```
Index 0 → Reach up to index 2
Index 1 → Reach up to index 4
```

Once we know the farthest reachable index is `4`, we know the last index is reachable.

This observation leads to the Greedy approach.

## 4. Recognize the Pattern — Greedy

The key idea is to maintain:

```
int maxReach = 0;
```

`maxReach` represents the farthest index we can reach using all positions we've been able to access so far.

For every index `i`, calculate:

\\[ \text{newReach} = i + nums[i] \\]

Update:

\\[ \text{maxReach} = \max(\text{maxReach}, i + nums[i]) \\]

But there is one important condition:

We can only process index `i` if it is reachable.

```
if (i > maxReach) {
    return false;
}
```

Why?

Because if the current index is beyond our farthest reachable position, we cannot get there.

### Visual example

```
nums = [2,3,1,1,4]
```

### 2

Index 0

### 3

Index 1

### 1

Index 2

### 1

Index 3

### 4

Index 4

At index 0, maxReach becomes 2. At index 1, maxReach becomes 4.

Last index is reachable

## 5. Derive the Optimal Approach

Let's build the logic.

Initialize:

```
int maxReach = 0;
```

Now traverse the array:

```
for (int i = 0; i < nums.length; i++) {
```

### Step 1: Check whether the current index is reachable

```
if (i > maxReach) {
    return false;
}
```

Suppose:

```
nums = [3,2,1,0,4]
```

At index `3`, the farthest reachable index is `3`.

We can reach index `3`.

But after processing it:

```
newReach = 3 + 0 = 3
```

At index `4`:

```
4 > 3
```

We cannot reach index `4`.

So return `false`.

### Step 2: Update the farthest reachable index

```
maxReach = Math.max(maxReach, i + nums[i]);
```

Why use `Math.max`?

Because we want to preserve the farthest position reached so far.

For example:

```
maxReach = 4
i = 2
nums[i] = 1

i + nums[i] = 3
```

The new position is only `3`, but we already know we can reach `4`.

We must not reduce `maxReach`.

### Step 3: Return true if traversal completes

If we finish the loop without encountering an unreachable index, the last index is reachable.

## 6. Optimal Java Solution

O(n) Time

```
class Solution {
    public boolean canJump(int[] nums) {

        // Track the farthest index we can reach.
        int maxReach = 0;

        // Traverse every index.
        for (int i = 0; i < nums.length; i++) {

            // If current index is beyond our reach,
            // we cannot proceed.
            if (i > maxReach) {
                return false;
            }

            // Update the farthest reachable index.
            maxReach = Math.max(maxReach, i + nums[i]);

            // We can already reach the last index.
            if (maxReach >= nums.length - 1) {
                return true;
            }
        }

        return true;
    }
}
```

### Complexity

- Time: `O(n)` — we traverse the array once.
- Space: `O(1)` — only one extra variable is maintained.

## 7. Detailed Dry Run

Let's dry-run both examples because understanding the unreachable case is particularly important.

### Example A: Reachable

```
nums = [2,3,1,1,4]
```

Initially:

```
maxReach = 0
```

Iteration 1 — Index 0

### 2

0

### 3

1

### 1

2

### 1

3

### 4

4

```
i = 0
maxReach = 0
```

Reachability check:

```
0 > 0 → false
```

Update:

```
maxReach = max(0, 0 + 2)
         = 2
```

We can reach indices `0, 1, 2`.

Iteration 2 — Index 1

```
i = 1
maxReach = 2
```

Reachability check:

```
1 > 2 → false
```

Update:

```
maxReach = max(2, 1 + 3)
         = 4
```

We can now reach index `4`.

Return true

### Example B: Unreachable

```
nums = [3,2,1,0,4]
```

Initially:

```
maxReach = 0
```

| Index | `nums[i]` | Previous `maxReach` | New `maxReach` |
| ----- | --------- | ------------------- | -------------- |
| 0     | 3         | 0                   | 3              |
| 1     | 2         | 3                   | 3              |
| 2     | 1         | 3                   | 3              |
| 3     | 0         | 3                   | 3              |
| 4     | 4         | 3                   | —              |

At index `4`:

```
if (i > maxReach)
```

Substitute:

```
4 > 3 → true
```

We cannot reach index `4`.

Return false

### Complete Dry Run Table

| Index | `nums[i]` | `i + nums[i]` | `maxReach` after update |
| ----- | --------- | ------------- | ----------------------- |
| 0     | 2         | 2             | 2                       |
| 1     | 3         | 4             | 4                       |

We can stop as soon as `maxReach` reaches the last index.

## 8. Tricky Parts You Must Understand

### A. Why do we use `i > maxReach`?

This is the most important condition.

Suppose:

```
nums = [3,2,1,0,4]
```

At index `3`, we can reach it because:

```
3 <= maxReach (3)
```

But at index `4`:

```
4 > maxReach (3)
```

There is no possible way to reach that index.

So return `false`.

### B. Why use `Math.max()`?

Suppose:

```
nums = [4,1,1,1,0]
```

At index `0`:

```
maxReach = 4
```

At index `1`:

```
i + nums[i] = 1 + 1 = 2
```

If we simply wrote:

```
maxReach = i + nums[i];
```

We would incorrectly reduce `maxReach` from `4` to `2`.

That's why we use:

```
maxReach = Math.max(maxReach, i + nums[i]);
```

### C. Why don't we need to calculate the exact jump path?

Because the problem only asks whether reaching the last index is possible.

We don't need to know whether the path is:

```
0 → 1 → 4
```

or:

```
0 → 2 → 3 → 4
```

We only need the farthest reachable position.

### D. Is `maxReach >= nums.length - 1` necessary?

No, it is an optimization.

Even without it, the loop can finish and return `true` if the last index is reachable.

However, this early-return condition avoids unnecessary iterations once the destination is reachable.

## GitHub-Ready Notes

```
Problem: LeetCode 55 — Jump Game

Pattern:
Greedy — Farthest Reachable Index

Approach:
1. Initialize maxReach = 0.
2. Traverse the array.
3. If i > maxReach, return false.
4. Update maxReach = max(maxReach, i + nums[i]).
5. If maxReach >= n - 1, return true.
6. If traversal completes, return true.

Time Complexity: O(n)
Space Complexity: O(1)

Key Insight:
Maintain the farthest index reachable so far.
If the current index exceeds maxReach, it is unreachable.

Important:
Use Math.max() to avoid reducing the farthest reach.
The value nums[i] represents maximum jump distance,
not an exact jump distance.
```

## Quick Comparison: LC 55 vs LC 45

| Concept       | LC 55 — Jump Game                     | LC 45 — Jump Game II     |
| ------------- | ------------------------------------- | ------------------------ |
| Goal          | Check whether last index is reachable | Find minimum jumps       |
| Return type   | `boolean`                             | `int`                    |
| Main variable | `maxReach`                            | `currentEnd`, `farthest` |
| Pattern       | Greedy                                | Greedy                   |
| Time          | `O(n)`                                | `O(n)`                   |

LC 45 is a natural follow-up. It uses a similar farthest-reach idea, but you must determine when to count a jump.

## Test Your Understanding

Try these four questions.

1\. nums = [2,3,1,1,4]. What is maxReach after processing index 1?

2

3

4

2\. Why do we use maxReach = Math.max(maxReach, i + nums[i])?

To keep the farthest reachable index

To count the number of jumps

To sort the array

3\. nums = [3,2,1,0,4]. At which index do we discover that the destination is unreachable?

Index 2

Index 3

Index 4

4\. What is the time complexity of the greedy solution?

O(n²)

O(n)

O(log n)

Check answers

Next recommended problem: LC 45 — Jump Game II. It is one of the best follow-ups to understand how greedy farthest-reach tracking can solve a minimum-jumps problem.
