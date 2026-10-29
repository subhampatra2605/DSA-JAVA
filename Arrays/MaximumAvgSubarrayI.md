# LeetCode 643: Maximum Average Subarray I

We'll solve this using your usual **8-step DSA framework**, with commented Java code for both brute force and optimal solutions.

This problem is particularly useful for your Infosys SP preparation because it teaches a fundamental pattern: **Fixed-Size Sliding Window**.

## Step 1: Understand the Problem

You are given an integer array `nums` and an integer `k`.

Find a contiguous subarray of length exactly `k` that has the maximum average, and return that average.

### Example

```text
Input:
nums = [1, 12, -5, -6, 50, 3]
k = 4

Output:
12.75
```

We must examine contiguous subarrays containing exactly four elements.

Possible windows include:

```text
Window 1: [1, 12, -5, -6]
Sum = 2
Average = 2 / 4 = 0.5
```

```text
Window 2: [12, -5, -6, 50]
Sum = 51
Average = 51 / 4 = 12.75
```

```text
Window 3: [-5, -6, 50, 3]
Sum = 42
Average = 42 / 4 = 10.5
```

The maximum average is **12.75**.

### The important observation

Every subarray must have exactly `k` elements.

Because `k` is fixed, every candidate average is:

\[
\text{Average}=\frac{\text{Sum of the window}}{k}
\]

Since `k` remains constant, the window with the **maximum sum** also has the maximum average.

Therefore, we can solve the problem by finding the maximum sum of any contiguous subarray of length `k`, then dividing that sum by `k`.

---

## Step 2: Brute-Force Approach

### Idea

Try every possible subarray of length `k`.

For each starting index:

1. Add the next `k` elements.
2. Calculate their average.
3. Update the maximum average found so far.

### Java Code — Brute Force

```java
class Solution {
    public double findMaxAverage(int[] nums, int k) {
        int n = nums.length;
        double maxAverage = Double.NEGATIVE_INFINITY;

        // Try every possible starting position for a window of size k
        for (int i = 0; i <= n - k; i++) {

            int sum = 0;

            // Calculate the sum of the k elements in this window
            for (int j = i; j < i + k; j++) {
                sum += nums[j];
            }

            // Calculate the average of the current window
            double average = (double) sum / k;

            // Update the maximum average
            maxAverage = Math.max(maxAverage, average);
        }

        return maxAverage;
    }
}
```

### Complexity

- **Time:** `O(n × k)` — we calculate the sum of `k` elements for every possible starting position.
- **Auxiliary space:** `O(1)` — we use only a few variables.

Why is this inefficient?

Consider an array with 1,000 elements and `k = 100`.

When calculating the sum of the first window, we add 100 elements.

When calculating the sum of the next window, we add 100 elements again—even though 99 of those elements were already included in the previous window.

**Can we reuse the sum we have already calculated?**

Yes. That is the key to the sliding-window solution.

---

## Step 3: Identify the Bottleneck

Consider these two consecutive windows:

```text
nums = [1, 12, -5, -6, 50, 3]
k = 4
```

First window:

```text
[1, 12, -5, -6]
Sum = 2
```

Second window:

```text
[12, -5, -6, 50]
Sum = 51
```

Compare the windows:

```text
Previous: [1,  12, -5, -6]
Current:      [12, -5, -6, 50]
```

What changed?

- We removed `1` from the left.
- We added `50` on the right.

The other three elements remained unchanged.

Therefore, instead of recalculating the entire sum:

\[
\text{New sum}=\text{Old sum}-\text{Outgoing element}+\text{Incoming element}
\]

For the example:

\[
51=2-1+50
\]

This is the optimization we need.

---

## Step 4: Recognize the Pattern

### Pattern: Fixed-Size Sliding Window

Use a fixed-size sliding window when a problem asks you to examine **contiguous subarrays of a specified length `k`** and calculate something for each window.

The general approach is:

1. Calculate the result for the first window.
2. Slide the window one position at a time.
3. Remove the contribution of the outgoing element.
4. Add the contribution of the incoming element.
5. Update the best answer.

For this problem, the window's size always remains `k`.



### Connection to problems you already know

| Problem | Pattern | Key idea |
|---|---|---|
| LC 643 — Maximum Average Subarray I | Fixed-size sliding window | Maintain the sum of `k` elements |
| LC 53 — Maximum Subarray | Kadane's algorithm | Decide whether to extend or restart a subarray |
| LC 238 — Product of Array Except Self | Prefix/suffix | Reuse information from other positions |

**Important distinction:** LC 643 requires a subarray of exactly `k` elements, so the window size is fixed. LC 53 allows the length to vary, so Kadane's algorithm is appropriate there.

---

## Step 5: Derive the Optimal Approach

Let's derive the solution rather than simply memorizing it.

### Step 1: Calculate the first window's sum

For:

```text
nums = [1, 12, -5, -6, 50, 3]
k = 4
```

The first four elements are:

```text
[1, 12, -5, -6]
```

Their sum is:

```text
1 + 12 - 5 - 6 = 2
```

We store this in `windowSum`.

```java
int windowSum = 0;

for (int i = 0; i < k; i++) {
    windowSum += nums[i];
}
```

### Step 2: Initialize the maximum sum

The first window is a valid candidate, so initially:

```java
int maxSum = windowSum;
```

Here:

```text
windowSum = 2
maxSum = 2
```

### Step 3: Slide the window

The next window removes the element at index `0` and adds the element at index `4`.

```text
Previous window: [1, 12, -5, -6]
Next window:        [12, -5, -6, 50]
```

The outgoing element is:

```java
nums[i - k]
```

The incoming element is:

```java
nums[i]
```

Therefore, the update is:

```java
windowSum = windowSum - nums[i - k] + nums[i];
```

We can also write:

```java
windowSum += nums[i] - nums[i - k];
```

Both expressions do exactly the same thing.

**Why does the outgoing index equal `i - k`?**

We use `i` as the index of the incoming element. The element leaving the window is exactly `k` positions behind it.

For example, when `i = 4` and `k = 4`:

```text
Incoming index = 4
Outgoing index = 4 - 4 = 0
```

So we remove `nums[0]` and add `nums[4]`.

The new sum becomes:

```text
windowSum = 2 - 1 + 50
          = 51
```

### Step 4: Update the maximum sum

```java
maxSum = Math.max(maxSum, windowSum);
```

Since `51 > 2`, the new maximum becomes `51`.

Repeat this process until every window has been examined.

### Step 5: Calculate the average

Once we know the maximum sum, divide it by `k`:

```java
return (double) maxSum / k;
```

The cast to `double` is important. Without it, integer division could discard the fractional part.

For example:

```java
51 / 4        // 12
(double) 51 / 4  // 12.75
```

---

## Step 6: Optimal Java Solution — With Comments

```java
class Solution {
    public double findMaxAverage(int[] nums, int k) {

        int n = nums.length;

        // Calculate the sum of the first window of size k
        int windowSum = 0;

        for (int i = 0; i < k; i++) {
            windowSum += nums[i];
        }

        // Initially, the first window has the maximum sum
        int maxSum = windowSum;

        // Slide the window one position at a time
        for (int i = k; i < n; i++) {

            // Remove the outgoing element and add the incoming element
            windowSum = windowSum - nums[i - k] + nums[i];

            // Update the maximum sum found so far
            maxSum = Math.max(maxSum, windowSum);
        }

        // Convert the maximum sum to an average
        return (double) maxSum / k;
    }
}
```

### Complexity Analysis

| Complexity | Explanation |
|---|---|
| **Time: `O(n)`** | Calculate the first window in `O(k)`, then examine each remaining element once. Overall, `O(k + n - k) = O(n)`. |
| **Auxiliary space: `O(1)`** | Only a constant number of variables are used. |

We've improved the time complexity from **`O(n × k)` to `O(n)`**.

---

## Step 7: Dry Run the Optimal Solution

Consider:

```text
nums = [1, 12, -5, -6, 50, 3]
k = 4
```

### Phase 1: Initialize the first window

We add the first four elements:

```text
1 + 12 + (-5) + (-6) = 2
```

Initial values:

```text
windowSum = 2
maxSum = 2
```

### Phase 2: Slide the window

The loop starts at `i = k`, which means `i = 4`.

| Step | Incoming element | Outgoing element | Calculation | Window sum | Maximum sum |
|---|---:|---:|---|---:|---:|
| Initial | — | — | `1 + 12 - 5 - 6` | 2 | 2 |
| 1 | 50 | 1 | `2 - 1 + 50` | 51 | 51 |
| 2 | 3 | 12 | `51 - 12 + 3` | 42 | 51 |

Let's examine the pointer movement and arithmetic in more detail.

**First slide (`i = 4`):**

```text
Previous window: [1, 12, -5, -6]
New window:         [12, -5, -6, 50]
```

Code:

```java
windowSum = windowSum - nums[i - k] + nums[i];
```

Substituting the values:

```text
windowSum = 2 - nums[0] + nums[4]
          = 2 - 1 + 50
          = 51
```

Update:

```text
maxSum = max(2, 51) = 51
```

**Second slide (`i = 5`):**

```text
Previous window: [12, -5, -6, 50]
New window:          [-5, -6, 50, 3]
```

The outgoing element is `nums[1] = 12`. The incoming element is `nums[5] = 3`.

```text
windowSum = 51 - 12 + 3
          = 42
```

Update:

```text
maxSum = max(51, 42) = 51
```

The maximum sum remains `51`.

### Final answer

```text
maxSum = 51
k = 4

Maximum average = 51 / 4
               = 12.75
```

**Output: `12.75`**

Notice that after calculating the first window, we never recalculate a complete window sum. We only remove one element and add another.

---

## Step 8: Tricky Parts, Key Insights, and Revision

### 1. Why does the loop start at `i = k`?

Because the first window already contains indices `0` through `k - 1`.

The next element to enter the window is at index `k`.

For example, when `k = 4`:

```text
First window: indices 0, 1, 2, 3
Next incoming index: 4
```

Therefore:

```java
for (int i = k; i < n; i++)
```

is correct.

### 2. Why do we subtract `nums[i - k]`?

Because it is the element leaving the window.

When the incoming index is `i`, the window still has exactly `k` elements. The outgoing element is `k` positions behind the incoming element.

```java
windowSum = windowSum - nums[i - k] + nums[i];
```

Remember this formula. It is one of the most important lines in fixed-size sliding-window problems.

### 3. Why do we track the maximum sum instead of the maximum average?

Because the window size `k` is fixed.

Suppose `k = 4`:

- Window A has sum `20`, so its average is `5`.
- Window B has sum `32`, so its average is `8`.

The window with the greater sum necessarily has the greater average because both sums are divided by the same positive number.

Therefore, we can compare integer sums and divide only once at the end.

### 4. Why initialize `maxSum = windowSum`?

The first window is already a valid candidate.

If we initialized `maxSum` to `0`, the algorithm could fail when all window sums are negative.

For example:

```text
nums = [-5, -2, -8]
k = 2
```

The window sums are:

```text
[-5, -2] → -7
[-2, -8] → -10
```

The maximum sum is `-7`, and the maximum average is `-3.5`.

Initializing `maxSum` with the first window's sum ensures we handle negative numbers correctly.

### 5. Compare brute force with the optimal solution

| Feature | Brute force | Sliding window |
|---|---|---|
| First window | Calculate its complete sum | Calculate its complete sum |
| Subsequent windows | Recalculate all `k` elements | Remove one element and add one |
| Time complexity | `O(n × k)` | `O(n)` |
| Auxiliary space | `O(1)` | `O(1)` |
| Pattern | Nested loops | Fixed-size sliding window |

### 6. Your DSA-Java GitHub notes

For your repository, record:

- **Problem:** Find the maximum average of a contiguous subarray of length `k`.
- **Brute-force approach:** Calculate every window's sum independently.
- **Optimal approach:** Fixed-size sliding window.
- **Pattern:** Sliding Window — Fixed Size.
- **Time:** `O(n)`.
- **Space:** `O(1)` auxiliary space.
- **Key Insight:** Reuse the previous window's sum by subtracting the outgoing element and adding the incoming element. Since `k` is fixed, maximizing the sum also maximizes the average.

---

## Your Practice Questions

Try answering these without looking at the solution:

1. Why is this a fixed-size sliding-window problem rather than a variable-size sliding-window problem?
2. If `k = 3` and the incoming element is at index `5`, which element leaves the window? Explain why.
3. Why does the optimal solution not need to recalculate the entire window sum?
4. Why do we initialize `maxSum` with `windowSum` rather than `0`?
5. **Coding challenge:** Modify the optimal solution so it returns the maximum **sum** instead of the maximum average.

**My advice for your Infosys SP preparation:** Focus on understanding why the outgoing element is at `i - k` and why the incoming element is at `i`. Once you understand those two indices, you can derive the fixed-size sliding-window solution yourself rather than memorizing it.
