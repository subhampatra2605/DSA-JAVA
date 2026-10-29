# LeetCode 209: Minimum Size Subarray Sum

We'll solve this using your usual **8-step DSA framework**, with commented Java code for both brute-force and optimal approaches, a step-by-step dry run, and a clear derivation of the sliding-window technique.

This problem builds directly on LC 643 (Maximum Average Subarray I) and LC 3 (Longest Substring Without Repeating Characters).

**The key difference:** LC 643 uses a fixed-size window, LC 3 changes the window to maintain unique characters, and LC 209 changes the window to find the shortest subarray satisfying a sum condition.

## Step 1: Understand the Problem

You are given:

- An array of **positive integers** `nums`.
- A positive integer `target`.

Return the **minimum length of a contiguous subarray** whose sum is greater than or equal to `target`.

If no such subarray exists, return `0`.

### Example 1

```text
Input:
nums = [2, 3, 1, 2, 4, 3]
target = 7

Output: 2
```

Explanation:

We need a contiguous subarray whose sum is at least `7`.

Consider these possibilities:

```text
[2, 3, 1, 2]    Sum = 8    Length = 4
[3, 1, 2, 4]    Sum = 10   Length = 4
[4, 3]          Sum = 7    Length = 2
```

The subarray `[4, 3]` has a sum of `7` and a length of `2`.

No single element is at least `7`, so no valid subarray can have length `1`.

Therefore:

```text
Output: 2
```

### Example 2

```text
Input:
nums = [1, 4, 4]
target = 4

Output: 1
```

The subarray `[4]` meets the target, and its length is `1`.

### Example 3

```text
Input:
nums = [1, 1, 1, 1, 1, 1, 1, 1]
target = 11

Output: 0
```

The sum of the entire array is only `8`, so no valid subarray exists.

### What exactly are we optimizing?

We are **minimizing the length**, not maximizing the sum.

Our conditions are:

1. The subarray must be contiguous.
2. Its sum must be at least `target`.
3. Among all valid subarrays, choose the shortest.
4. If no valid subarray exists, return `0`.

Since all elements are positive, adding an element increases the sum, while removing an element decreases it. This property is what makes the sliding-window solution possible.

---

## Step 2: Brute-Force Approach

### Idea

Try every possible starting index and extend the subarray to the right until its sum reaches or exceeds `target`.

Because all elements are positive, once the sum reaches `target`, extending the subarray further cannot produce a shorter valid subarray. We can update the answer and stop exploring that starting position.

### Java Code — Brute Force

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int n = nums.length;

        // Initialize with a value larger than any possible answer
        int minLength = n + 1;

        // Try every possible starting position
        for (int start = 0; start < n; start++) {
            int sum = 0;

            // Extend the subarray to the right
            for (int end = start; end < n; end++) {

                // Include the current element in the sum
                sum += nums[end];

                // Check whether the target has been reached
                if (sum >= target) {

                    // Calculate the length of this valid subarray
                    int length = end - start + 1;

                    // Keep the shortest valid length
                    minLength = Math.min(minLength, length);

                    // Further extensions cannot shorten this subarray
                    break;
                }
            }
        }

        // Return 0 if no valid subarray was found
        return minLength == n + 1 ? 0 : minLength;
    }
}
```

### Complexity

- **Time:** `O(n²)` in the worst case, because we may explore many subarrays.
- **Auxiliary space:** `O(1)`, because we use only a few variables.

### Why is this still inefficient?

Consider an array containing many positive numbers, where the target is large.

For each starting position, we repeatedly add elements until the target is reached or the array ends.

Many of those additions overlap with work performed for other starting positions.

For example, after calculating the sum of `[2, 3, 1, 2]`, we might calculate sums of other subarrays containing many of the same elements.

**Can we reuse the sum of the current window and remove elements from its left side when the sum is large enough?**

Yes. That leads to the optimal solution.

---

## Step 3: Identify the Bottleneck

Consider:

```text
nums = [2, 3, 1, 2, 4, 3]
target = 7
```

Start with the first window:

```text
[2, 3, 1, 2]

sum = 8
length = 4
```

The window is valid because `8 >= 7`.

But is it the shortest valid window?

No. We can try removing the leftmost element.

```text
[3, 1, 2]

sum = 6
length = 3
```

Now the sum is too small, so we cannot remove any more elements from the left while maintaining validity.

Next, continue expanding the right boundary to include `4`:

```text
[3, 1, 2, 4]

sum = 10
length = 4
```

Again, the window is valid. We can shrink it from the left:

```text
[1, 2, 4]

sum = 7
length = 3
```

Remove the leftmost element once more:

```text
[2, 4]

sum = 6
```

The window is now invalid, so we stop shrinking.

Continue expanding to include `3`:

```text
[2, 4, 3]

sum = 9
length = 3
```

Shrink from the left:

```text
[4, 3]

sum = 7
length = 2
```

This is the shortest valid window found.

The key observation is that we do not need to restart the search for each starting index. We can move the right pointer forward and, whenever the sum reaches the target, move the left pointer forward to find shorter valid windows.

---

## Step 4: Recognize the Pattern

### Pattern: Variable-Size Sliding Window

This problem uses the same general sliding-window structure as LC 3, but the condition we maintain is different.

| Feature | LC 3 — Longest Substring | LC 209 — Minimum Size Subarray Sum |
|---|---|---|
| Window condition | No repeated characters | Sum is at least `target` |
| Expand right | Add the next character | Add the next number |
| Shrink left | While a duplicate exists | While the sum is at least `target` |
| Update answer | When the window is valid | While the window is valid |
| Objective | Maximize length | Minimize length |

### The central idea

Maintain a window using:

```java
int left = 0;
int sum = 0;
int minLength = Integer.MAX_VALUE;
```

Then expand the right boundary:

```java
for (int right = 0; right < nums.length; right++) {
    sum += nums[right];
}
```

Whenever the sum reaches the target, shrink the window:

```java
while (sum >= target) {
    // Update the minimum length
    // Remove the leftmost element
    // Move left forward
}
```

The `while` loop is crucial because **one valid window may contain several shorter valid windows**.

For example, if the current window is `[2, 4, 3]` with sum `9` and target `7`, we should check `[4, 3]` before moving the right pointer again.

---

## Step 5: Derive the Optimal Approach

Let's construct the algorithm one statement at a time.

### Step 1: Initialize the variables

```java
int left = 0;
int sum = 0;
int minLength = Integer.MAX_VALUE;
```

- `left`: Beginning of the current window.
- `sum`: Sum of the elements inside the current window.
- `minLength`: Shortest valid subarray length found so far.

Why initialize `minLength` to `Integer.MAX_VALUE`?

We haven't found a valid subarray yet, so we need an initial value larger than any possible valid answer.

If we finish without finding a valid window, we'll return `0`.

### Step 2: Expand the window

```java
for (int right = 0; right < nums.length; right++) {
    sum += nums[right];
}
```

Every iteration adds the element at `right` to the current window.

For example:

```text
nums = [2, 3, 1, 2, 4, 3]
target = 7
```

As the right pointer moves:

| `right` | Added element | Current window | Sum |
|---:|---:|---|---:|
| 0 | 2 | `[2]` | 2 |
| 1 | 3 | `[2, 3]` | 5 |
| 2 | 1 | `[2, 3, 1]` | 6 |
| 3 | 2 | `[2, 3, 1, 2]` | 8 |

At `right = 3`, the sum reaches `8`, which is at least `7`.

Now we can try shrinking the window.

### Step 3: Shrink the window while it is valid

```java
while (sum >= target) {
    // Process the current valid window
}
```

Why use `while` instead of `if`?

Because we want to find the shortest valid window ending at the current `right` position.

Suppose:

```text
Window = [2, 3, 1, 2]
Sum = 8
Length = 4
```

The window is valid.

We first update the answer:

```java
minLength = Math.min(minLength, right - left + 1);
```

The current length is:

```text
right - left + 1
= 3 - 0 + 1
= 4
```

Then remove the leftmost element:

```java
sum -= nums[left];
left++;
```

Now:

```text
Previous window = [2, 3, 1, 2]
New window      = [3, 1, 2]

Previous sum = 8
New sum      = 8 - 2 = 6
```

Since `6 < 7`, the `while` loop stops.

We've found a valid window of length `4`, but we cannot shrink it any further at this moment.

### Step 4: Continue expanding

The right pointer continues moving, adding the next element, `4`.

The window becomes:

```text
[3, 1, 2, 4]
```

Its sum is:

```text
6 + 4 = 10
```

Since `10 >= 7`, we enter the shrinking loop again.

The current length is:

```text
right - left + 1
= 4 - 1 + 1
= 4
```

Our best answer remains `4`.

Remove `3`:

```text
New window = [1, 2, 4]
New sum = 10 - 3 = 7
```

The window is still valid!

Therefore, we process it again.

Its length is:

```text
4 - 2 + 1 = 3
```

Our best answer becomes `3`.

Remove `1`:

```text
New window = [2, 4]
New sum = 7 - 1 = 6
```

Now the sum is too small, so we stop shrinking.

This illustrates why the shrinking loop must continue **as long as the window remains valid**.

### Step 5: Return the result

After processing every element:

```java
return minLength == Integer.MAX_VALUE ? 0 : minLength;
```

If we found a valid window, return its minimum length.

Otherwise, return `0`.

### The complete algorithm

1. Initialize `left`, `sum`, and `minLength`.
2. Move `right` through the array.
3. Add `nums[right]` to `sum`.
4. While `sum >= target`:
   - Update `minLength`.
   - Subtract `nums[left]` from `sum`.
   - Increment `left`.
5. Return the minimum length, or `0` if no valid window exists.

---

## Step 6: Optimal Java Solution — With Comments

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {

        // Left boundary of the current window
        int left = 0;

        // Sum of the elements in the current window
        int sum = 0;

        // Initialize with a value larger than any valid length
        int minLength = Integer.MAX_VALUE;

        // Expand the window by moving the right pointer
        for (int right = 0; right < nums.length; right++) {

            // Include the new element in the current window
            sum += nums[right];

            // Shrink the window while its sum meets the target
            while (sum >= target) {

                // Update the shortest valid subarray length
                int length = right - left + 1;
                minLength = Math.min(minLength, length);

                // Remove the leftmost element from the window
                sum -= nums[left];

                // Move the left boundary forward
                left++;
            }
        }

        // Return 0 if no valid subarray was found
        return minLength == Integer.MAX_VALUE ? 0 : minLength;
    }
}
```

### Complexity Analysis

| Complexity | Explanation |
|---|---|
| **Time: `O(n)`** | The right pointer moves forward at most `n` times, and the left pointer also moves forward at most `n` times. |
| **Auxiliary space: `O(1)`** | We use only a constant number of variables. |

Although the code contains a `while` loop inside a `for` loop, the time complexity is not `O(n²)`.

Each pointer moves only forward. Across the entire algorithm, the total number of pointer movements is linear.

**Important assumption:** This sliding-window solution relies on every number being positive. With negative numbers, removing an element could increase the sum, and the same shrinking strategy would not be guaranteed to work.

---

## Step 7: Dry Run the Optimal Solution

Let's trace the complete algorithm using:

```text
nums = [2, 3, 1, 2, 4, 3]
target = 7
```

Initially:

```text
left = 0
sum = 0
minLength = Integer.MAX_VALUE
```

### Detailed execution

| Step | `right` | Added element | Window before shrinking | Sum | Minimum length |
|---:|---:|---:|---|---:|---:|
| 1 | 0 | 2 | `[2]` | 2 | Not found |
| 2 | 1 | 3 | `[2, 3]` | 5 | Not found |
| 3 | 2 | 1 | `[2, 3, 1]` | 6 | Not found |
| 4 | 3 | 2 | `[2, 3, 1, 2]` | 8 | 4 |
| 5 | 4 | 4 | `[3, 1, 2, 4]` | 10 | 4 |
| 6 | 5 | 3 | `[2, 4, 3]` | 9 | 3 |

The table shows the window before shrinking at each step. Let's examine the shrinking operations, which are the most important part of this problem.

### At `right = 3`

We have:

```text
Window = [2, 3, 1, 2]
sum = 8
left = 0
right = 3
```

Since `8 >= 7`, calculate the length:

```text
length = 3 - 0 + 1 = 4
minLength = 4
```

Remove the first element, `2`:

```text
sum = 8 - 2 = 6
left = 1
```

Now `sum < target`, so stop shrinking.

### At `right = 4`

Add `nums[4] = 4`:

```text
Window = [3, 1, 2, 4]
sum = 6 + 4 = 10
```

The sum is valid.

**First shrinking operation:**

```text
Window = [3, 1, 2, 4]
Length = 4
minLength = 4
```

Remove `3`:

```text
sum = 10 - 3 = 7
left = 2
```

The sum is still at least `7`, so continue shrinking.

**Second shrinking operation:**

```text
Window = [1, 2, 4]
Length = 4 - 2 + 1 = 3
minLength = min(4, 3) = 3
```

Remove `1`:

```text
sum = 7 - 1 = 6
left = 3
```

The sum is now too small, so stop shrinking.

### At `right = 5`

Add `nums[5] = 3`:

```text
Window = [2, 4, 3]
sum = 6 + 3 = 9
```

The sum is valid.

**First shrinking operation:**

```text
Window = [2, 4, 3]
Length = 5 - 3 + 1 = 3
minLength = 3
```

Remove `2`:

```text
sum = 9 - 2 = 7
left = 4
```

The window is still valid.

**Second shrinking operation:**

```text
Window = [4, 3]
Length = 5 - 4 + 1 = 2
minLength = min(3, 2) = 2
```

Remove `4`:

```text
sum = 7 - 4 = 3
left = 5
```

Now the sum is below `7`, so stop shrinking.

The array has been fully processed.

### Final answer

```text
Minimum valid subarray length = 2
```

The shortest valid subarray is:

```text
[4, 3]
```

Therefore:

```text
Output: 2
```

---

## Step 8: Tricky Parts, Key Insights, and Revision

### 1. Why do we use `while (sum >= target)`?

Because a single valid window may contain multiple shorter valid windows.

Consider:

```text
nums = [2, 3, 1, 2, 4, 3]
target = 7
```

When the window is `[3, 1, 2, 4]`, its sum is `10`.

Shrinking once gives:

```text
[1, 2, 4]
Sum = 7
```

This is still valid.

If we used `if` instead of `while`, we would stop after removing just one element and miss the shorter valid window `[1, 2, 4]` at that stage. More generally, we must keep shrinking until the window becomes invalid.

### 2. Why do we update `minLength` before removing the leftmost element?

Because the current window is valid before removal.

For example:

```text
Window = [4, 3]
Sum = 7
Length = 2
```

We must record its length before removing `4`. Otherwise, we would miss this valid window.

The correct order is:

```java
// First, record the current valid window
minLength = Math.min(minLength, right - left + 1);

// Then remove the outgoing element
sum -= nums[left];

// Finally, move the left boundary
left++;
```

### 3. Why does the algorithm require positive numbers?

This is crucial.

Suppose negative numbers were allowed. Removing an element from the left could increase the sum if that element were negative.

For example:

```text
Window = [-5, 8]
Sum = 3
```

Removing `-5` would increase the sum to `8`.

The positive-number constraint guarantees that expanding the right boundary cannot decrease the sum and shrinking the left boundary cannot increase it. This monotonic behavior makes the sliding-window strategy reliable.

### 4. What happens if no subarray reaches the target?

Consider:

```text
nums = [1, 1, 1, 1]
target = 10
```

The sum never reaches `10`, so the shrinking loop never executes.

`minLength` remains `Integer.MAX_VALUE`.

Therefore, we return:

```java
return minLength == Integer.MAX_VALUE ? 0 : minLength;
```

The output is `0`, as required.

### 5. Compare the three sliding-window problems

| Feature | LC 643 — Maximum Average | LC 3 — Longest Substring | LC 209 — Minimum Size Subarray |
|---|---|---|---|
| Window type | Fixed-size | Variable-size | Variable-size |
| Condition | Exactly `k` elements | Unique characters | Sum ≥ target |
| When to shrink | Every slide | When a duplicate exists | While the sum meets the target |
| Goal | Maximum average | Maximum length | Minimum length |
| Main data structure | Running sum | `HashSet` | Running sum |

The main distinction between LC 3 and LC 209 is **when and why we shrink the window**.

- LC 3 shrinks when the window becomes invalid because a character repeats.
- LC 209 shrinks while the window is valid because we want to find a shorter valid subarray.

This is a useful interview insight: sliding window is not one fixed algorithm. It is a technique whose window condition and shrinking rule depend on the problem.

### 6. Your DSA-Java GitHub notes

For your repository, record:

- **Problem:** Find the minimum length of a contiguous subarray whose sum is at least `target`.
- **Brute-force approach:** Try every starting position and extend the subarray.
- **Optimal approach:** Variable-size sliding window.
- **Pattern:** Sliding Window — Minimum Length.
- **Brute-force time:** `O(n²)`.
- **Optimal time:** `O(n)`.
- **Optimal auxiliary space:** `O(1)`.
- **Key Insight:** Expand the right boundary to increase the sum. Whenever the sum reaches the target, update the answer and repeatedly shrink from the left to find shorter valid windows.
- **Important constraint:** The standard sliding-window solution relies on all array elements being positive.

---

## Test Your Understanding

Try answering these questions without looking back:

1. Why is LC 209 a variable-size sliding-window problem?
2. Why do we use `while (sum >= target)` instead of `if (sum >= target)`?
3. Why do we calculate `right - left + 1` before moving `left`?
4. Why does the algorithm work for positive integers but not generally for arrays containing negative numbers?
5. **Coding challenge:** For `nums = [1, 2, 3, 4, 5]` and `target = 11`, trace every window and determine the output.

**Focus especially on Question 5.** Try to derive the answer by tracking only `left`, `right`, `sum`, and `minLength`. If you can do that confidently, you understand the core of LC 209.
