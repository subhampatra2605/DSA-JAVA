# **LeetCode 1004 — Max Consecutive Ones III**



Medium

Sliding Window + Two Pointers



This is an important **variable-size sliding window** problem. It builds on the same pattern used in LC 3 (Longest Substring Without Repeating Characters) and LC 209 (Minimum Size Subarray Sum).

The key idea:

**Find the longest contiguous subarray that contains at most&#x20;**`k`**&#x20;zeros.**

Instead of actually flipping zeros to ones, we track how many zeros are inside our current window.

## **1. Understand the problem**

You are given:

- A binary array `nums` containing only `0` and `1`.
- An integer `k`, representing the maximum number of zeros you can flip into ones.

Return the maximum number of consecutive ones you can obtain after flipping at most `k` zeros.

**Example 1**

```
Input:
nums = [1,1,1,0,0,0,1,1,1,1,0]
k = 2

Output: 6
```

One optimal window is:

```
[1, 1, 0, 0, 1, 1, 1, 1]
       ↑  ↑
      flip both zeros
```

After flipping:

```
[1, 1, 1, 1, 1, 1, 1, 1]
```

Length = `6`.

**Example 2**

```
Input:
nums = [0,0,1,1,1,0,0]
k = 0

Output: 3
```

We cannot flip any zeros, so the longest existing sequence of ones has length 3.

## **2. Brute-force approach**

Let's first consider every possible subarray.

For each starting index:

1. Extend the ending index.
2. Count zeros in the current subarray.
3. If zeros are at most `k`, update the maximum length.
4. If zeros exceed `k`, stop extending this subarray because adding more elements cannot reduce the zero count.

### **Java code — Brute force**

```
class Solution {
    public int longestOnes(int[] nums, int k) {

        int maxLength = 0;

        // Try every possible starting index.
        for (int start = 0; start < nums.length; start++) {

            int zeroCount = 0;

            // Extend the subarray from this starting index.
            for (int end = start; end < nums.length; end++) {

                // Count zeros in the current subarray.
                if (nums[end] == 0) {
                    zeroCount++;
                }

                // Stop if we exceed the allowed flips.
                if (zeroCount > k) {
                    break;
                }

                // Calculate the valid subarray length.
                int length = end - start + 1;

                // Update the maximum valid length.
                maxLength = Math.max(maxLength, length);
            }
        }

        return maxLength;
    }
}
```

**Complexity:**

- Time: \\(O(n^2)\\) — we examine multiple subarrays.
- Space: \\(O(1)\\) — only a few variables are used.

## **3. Identify the bottleneck**

The brute-force solution repeatedly checks subarrays that share many of the same elements.

For example:

```
[1, 1, 0, 1, 0, 1]
```

We might check:

```
[1, 1, 0]
[1, 1, 0, 1]
[1, 1, 0, 1, 0]
```

Each time, we are recounting zeros from the beginning.

Can we avoid this repeated work?

Yes. Use a sliding window.

Instead of creating every possible subarray, maintain one window using two pointers:

- `left`: beginning of the window.
- `right`: end of the window.

As `right` moves forward, we add new elements. If the number of zeros exceeds `k`, move `left` forward until the window becomes valid again.

## **4. Recognize the pattern — Variable-size Sliding Window**

The condition we need to maintain is:

\\[ \text{zeroCount} \leq k \\]

Our window is valid when it contains at most `k` zeros.

Our goal is to maximize:

\\[ \text{windowLength} = right-left+1 \\]

The sliding-window pattern works here because:

- Expanding the window by moving `right` can increase the number of zeros.
- Shrinking it by moving `left` can remove zeros.
- Each pointer moves only forward, so we don't repeatedly process the same elements.

## **5. Derive the optimal approach**

Let's build the logic step by step.

**Step 1 — Initialize variables**

```
int left = 0;
int zeroCount = 0;
int maxLength = 0;
```

**Step 2 — Expand the window**

```
for (int right = 0; right < nums.length; right++)
```

We add the new element to our window.

**Step 3 — Count zeros**

```
if (nums[right] == 0) {
    zeroCount++;
}
```

**Step 4 — Shrink while the window is invalid**

```
while (zeroCount > k) {
    if (nums[left] == 0) {
        zeroCount--;
    }

    left++;
}
```

Why `while` and not `if`?

Because removing one element might not be enough to bring the zero count back within the allowed limit. We keep shrinking until the window is valid.

**Step 5 — Update the maximum length**

```
int length = right - left + 1;
maxLength = Math.max(maxLength, length);
```

We update the answer only after the window is valid.

## **6. Optimal Java solution**

```
class Solution {
    public int longestOnes(int[] nums, int k) {

        int left = 0;
        int zeroCount = 0;
        int maxLength = 0;

        // Expand the window using the right pointer.
        for (int right = 0; right < nums.length; right++) {

            // Count a zero when it enters the window.
            if (nums[right] == 0) {
                zeroCount++;
            }

            // Shrink the window until it becomes valid.
            while (zeroCount > k) {

                // If the outgoing element is zero,
                // reduce the zero count.
                if (nums[left] == 0) {
                    zeroCount--;
                }

                // Move the left pointer forward.
                left++;
            }

            // Current window contains at most k zeros.
            int length = right - left + 1;

            // Store the maximum valid window length.
            maxLength = Math.max(maxLength, length);
        }

        return maxLength;
    }
}
```

Optimal solution

- **Time:** \\(O(n)\\) — each element enters and leaves the window at most once.
- **Space:** \\(O(1)\\) — only three variables are maintained.

## **7. Detailed dry run**

Input:

```
nums = [1,1,1,0,0,0,1,1,1,1,0]
k = 2
```

We can flip at most two zeros.

Initial state:

```
left = 0
zeroCount = 0
maxLength = 0
```

Let's trace every iteration.

| right | nums[right] | Action        | left | zeros | Window length | maxLength |
| ----- | ----------- | ------------- | ---- | ----- | ------------- | --------- |
| 0     | 1           | Expand        | 0    | 0     | 1             | 1         |
| 1     | 1           | Expand        | 0    | 0     | 2             | 2         |
| 2     | 1           | Expand        | 0    | 0     | 3             | 3         |
| 3     | 0           | Add zero      | 0    | 1     | 4             | 4         |
| 4     | 0           | Add zero      | 0    | 2     | 5             | 5         |
| 5     | 0           | Shrink window | 4    | 2     | 2             | 5         |
| 6     | 1           | Expand        | 4    | 2     | 3             | 5         |
| 7     | 1           | Expand        | 4    | 2     | 4             | 5         |
| 8     | 1           | Expand        | 4    | 2     | 5             | 5         |
| 9     | 1           | Expand        | 4    | 2     | 6             | 6         |
| 10    | 0           | Shrink window | 6    | 2     | 5             | 6         |

Let's understand the two important shrinking operations.

### **Iteration 1 — right = 5**

The current window becomes:

```
[1, 1, 1, 0, 0, 0]
```

There are three zeros, but `k = 2`.

So:

```
zeroCount = 3
```

We must shrink from the left.

- `nums[0] = 1` → remove it; zero count remains 3.
- `nums[1] = 1` → remove it; zero count remains 3.
- `nums[2] = 1` → remove it; zero count remains 3.
- `nums[3] = 0` → remove it; zero count becomes 2.

Now:

```
left = 4
zeroCount = 2
window = [0, 0]
```

The window is valid again.

Notice why we needed a `while` loop: we had to remove four elements before the zero count became valid.

### **Iteration 2 — right = 10**

The window before shrinking is:

```
[0, 1, 1, 1, 1, 0]
```

There are three zeros, exceeding `k = 2`.

Shrink from the left:

\- Remove `nums[4] = 0` → zero count becomes 2.

Now:

```
left = 5
window = [1, 1, 1, 1, 0]
```

This window is valid, with length 5.

Final maximum length:

# **6**

The longest valid window was:

```
[0, 0, 1, 1, 1, 1]
```

Flip the first two zeros:

```
[1, 1, 1, 1, 1, 1]
```

Answer = `6`.

## **8. Tricky parts and interview insights**

### **A. Why do we shrink using&#x20;**`while`**?**

Suppose:

```
nums = [1,1,0,1,0,0,1]
k = 1
```

When the window contains two zeros, removing one element from the left might remove a `1`, not a `0`.

We must continue shrinking until a zero leaves the window.

That's why:

```
while (zeroCount > k)
```

is essential.

### **B. Why do we update&#x20;**`maxLength`**&#x20;after shrinking?**

Because we only want to calculate the length of a valid window.

Correct order:

1. Expand.
2. Count zeros.
3. Shrink if invalid.
4. Update maximum length.

### **C. Why don't we actually flip zeros?**

We don't need to modify the original array.

The condition `zeroCount <= k` guarantees that we can convert all zeros in the current window into ones.

Therefore, the window length represents the number of consecutive ones achievable after flipping.

### **D. Why is this O(n), even with a nested while loop?**

Although there is a `while` loop inside a `for` loop, the total work is linear.

- `right` moves from left to right at most `n` times.
- `left` also moves from left to right at most `n` times.

Neither pointer moves backward.

Therefore:

\\[ O(n)+O(n)=O(n) \\]

## **9. Compare with related sliding-window problems**

| Problem | Window condition              | Objective      |
| ------- | ----------------------------- | -------------- |
| LC 3    | No repeated characters        | Maximum length |
| LC 209  | Sum at least target           | Minimum length |
| LC 424  | Replacements needed at most k | Maximum length |
| LC 1004 | Zeros at most k               | Maximum length |

**Important connection:** LC 1004 and LC 424 use almost the same sliding-window structure.

In LC 1004, we count zeros directly.

In LC 424, we calculate:

\\[ \text{replacementsNeeded} = \text{windowLength} - \text{maxFrequency} \\]

Both maintain a window that can be made valid using at most `k` replacements.

## **GitHub-ready notes — LC 1004**

**Problem:** Find the longest contiguous subarray containing at most `k` zeros.

**Approach:**

1. Initialize `left`, `zeroCount`, and `maxLength`.
2. Expand the window using `right`.
3. Increment `zeroCount` when a zero enters.
4. While `zeroCount > k`, shrink from the left.
5. Decrement `zeroCount` if the outgoing element is zero.
6. Calculate the current valid window length.
7. Update `maxLength`.

**Pattern:** Variable-size Sliding Window + Two Pointers

**Time Complexity:** \\(O(n)\\)

**Space Complexity:** \\(O(1)\\)

**Key Insights:**

- We don't actually flip zeros; we track their count.
- A valid window contains at most `k` zeros.
- Use `while` to restore validity.
- Update the maximum only after restoring validity.
- Each pointer moves forward at most `n` times.

## **10. Test your understanding**

Try these four questions without looking back at the code.

**Q1. If k = 0, what does this problem become?**

Find the longest subarray containing at most one zero

Find the longest existing consecutive sequence of ones

Find the shortest subarray



**Q2. Why do we use while (zeroCount > k)?**

To remove all ones

Because removing one element may not make the window valid

To move right backwards



**Q3. When should maxLength be updated?**

Before adding the right element

After restoring window validity

Only when a zero is found



**Q4. What is the time complexity?**

O(n²)

O(n log n)

O(n)



Check answers

**Recommended next problem:** LC 1493 — Longest Subarray of 1's After Deleting One Element. It uses a very similar sliding-window idea, but requires exactly one deletion rather than up to `k` flips.
