# LeetCode 11: Container With Most Water

We'll solve this using your usual **8-step DSA framework**: understand the problem, derive brute force, identify the bottleneck, recognize the pattern, derive the optimal solution, write commented Java code, dry-run it, and revise the key insights.

## Step 1: Understand the Problem

You are given an integer array `height`, where `height[i]` represents the height of a vertical line at index `i`.

Choose **two lines** that, together with the x-axis, form a container that holds the maximum amount of water.

Return the maximum water the container can store.

### Example

```text
Input:  height = [1,8,6,2,5,4,8,3,7]
Output: 49
```

Imagine each number represents a vertical line:



**How do we calculate the water capacity?**

The area depends on two things:

1. **Width:** The distance between the selected lines, `right - left`.
2. **Height:** The height of the shorter line, `Math.min(height[left], height[right])`.

Therefore:

\[
\boxed{\text{Area}=(right-left)\times\min(height[left],height[right])}
\]

For the maximum-area container in the example, choose heights `8` and `7`:

- Width = `8 - 1 = 7`
- Effective height = `min(8, 7) = 7`
- Area = `7 × 7 = 49`

**Important:** We use the shorter line's height because water would spill over that side.

---

## Step 2: Brute-Force Approach

### Idea

Try every possible pair of lines and calculate the area they can hold. Keep track of the maximum area.

For each left line, check every line to its right.

### Java Code — Brute Force

```java
class Solution {
    public int maxArea(int[] height) {
        int n = height.length;
        int maxWater = 0;

        // Select the first line
        for (int left = 0; left < n - 1; left++) {

            // Try every possible second line to its right
            for (int right = left + 1; right < n; right++) {

                // Width is the distance between the two lines
                int width = right - left;

                // Water height is limited by the shorter line
                int minHeight = Math.min(height[left], height[right]);

                // Calculate the water area for this pair
                int area = width * minHeight;

                // Update the maximum area found so far
                maxWater = Math.max(maxWater, area);
            }
        }

        return maxWater;
    }
}
```

### Complexity

- **Time:** `O(n²)` — two nested loops examine every pair.
- **Auxiliary space:** `O(1)` — only a few variables are used.

This works, but can we avoid checking every pair?

---

## Step 3: Identify the Bottleneck

The brute-force solution checks every possible pair of lines.

For an array of `n` elements, there are:

\[
\frac{n(n-1)}{2}
\]

possible pairs.

For example, with 9 lines, we check:

\[
\frac{9\times8}{2}=36\text{ pairs}
\]

As `n` grows, checking every pair becomes expensive.

**The bottleneck:** We calculate areas for many pairs that we could eliminate without checking them individually.

Can we intelligently eliminate some possibilities while still guaranteeing the maximum area?

Yes! We can use **two pointers**.

---

## Step 4: Recognize the Pattern

### Pattern: Two Pointers — Opposite Directions

This problem is similar to **LeetCode 167: Two Sum II** and **LeetCode 15: 3Sum**.

In those problems, we used two pointers to reduce the search space. Here, we use the same general idea, but the reason for moving a pointer is different.

Initialize:

```java
int left = 0;
int right = height.length - 1;
```

The pointers begin at opposite ends of the array.

At each step:

1. Calculate the area between the two lines.
2. Update the maximum area.
3. Move the pointer pointing to the **shorter line**.

The third step is the key to this problem.

### Why move the shorter line?

Suppose:

```text
height[left]  = 4
height[right] = 7
```

The current width is:

```text
right - left
```

The current area is:

```text
width * 4
```

The shorter line has height `4`, so it limits the amount of water.

Now consider the two possible moves.

**Option A: Move the right pointer inward.**

- The width decreases.
- The left line remains at height `4`.
- The new right line could be taller or shorter, but the effective height cannot exceed `4` while the left line remains fixed.

Therefore, the area cannot increase beyond the current area.

**Option B: Move the left pointer inward.**

- The width decreases.
- The shorter line is removed from consideration.
- We might find a new line taller than `4`, potentially increasing the effective height enough to compensate for the smaller width.

Therefore, **we move the pointer at the shorter line**.

The same reasoning applies when the right line is shorter.

If both lines have equal heights, either pointer can move.

---

## Step 5: Derive the Optimal Approach

Let's build the algorithm step by step.

### Step 1: Initialize the pointers and maximum area

```java
int left = 0;
int right = height.length - 1;
int maxWater = 0;
```

- `left` points to the first line.
- `right` points to the last line.
- `maxWater` stores the largest area found.

### Step 2: Continue while the pointers have not crossed

```java
while (left < right)
```

We need two distinct lines to form a container. Once the pointers meet, the width is zero, so there is no valid container left to examine.

### Step 3: Calculate the current area

```java
int width = right - left;
int minHeight = Math.min(height[left], height[right]);
int area = width * minHeight;
```

The area is the width multiplied by the height of the shorter line.

### Step 4: Update the maximum

```java
maxWater = Math.max(maxWater, area);
```

Keep the largest area encountered so far.

### Step 5: Move the shorter line

```java
if (height[left] < height[right]) {
    left++;
} else {
    right--;
}
```

If the left line is shorter, move `left` forward. Otherwise, move `right` backward.

When both heights are equal, the `else` branch moves `right`. This is valid because either pointer can be moved.

### Complete algorithm

1. Place the pointers at the two ends.
2. Calculate the area.
3. Update the maximum.
4. Move the pointer at the shorter line.
5. Repeat until `left >= right`.
6. Return `maxWater`.

---

## Step 6: Optimal Java Solution — With Comments

```java
class Solution {
    public int maxArea(int[] height) {

        // Initialize pointers at opposite ends of the array
        int left = 0;
        int right = height.length - 1;

        // Store the maximum water area found so far
        int maxWater = 0;

        // Continue while two distinct lines remain
        while (left < right) {

            // Calculate the width between the selected lines
            int width = right - left;

            // The shorter line determines the water height
            int minHeight = Math.min(height[left], height[right]);

            // Calculate the area of the current container
            int area = width * minHeight;

            // Update the maximum area
            maxWater = Math.max(maxWater, area);

            // Move the pointer at the shorter line
            if (height[left] < height[right]) {
                left++;
            } else {
                right--;
            }
        }

        // Return the largest area found
        return maxWater;
    }
}
```

### Complexity Analysis

| Complexity | Explanation |
|---|---|
| **Time: `O(n)`** | Each iteration moves one pointer inward. Each pointer moves at most `n` times. |
| **Auxiliary space: `O(1)`** | We use only a few variables, regardless of input size. |

Compared with brute force, we have improved the time complexity from **`O(n²)` to `O(n)`**.

---

## Step 7: Dry Run the Optimal Solution

Consider:

```text
height = [1,8,6,2,5,4,8,3,7]
```

Initially:

```text
left = 0
right = 8
maxWater = 0
```

Let's follow the pointer movements.

| Step | Left index | Right index | Left height | Right height | Width | Area | Max area | Pointer moved |
|---:|---:|---:|---:|---:|---:|---:|---:|---|
| 1 | 0 | 8 | 1 | 7 | 8 | 8 | 8 | `left++` |
| 2 | 1 | 8 | 8 | 7 | 7 | 49 | 49 | `right--` |
| 3 | 1 | 7 | 8 | 3 | 6 | 18 | 49 | `right--` |
| 4 | 1 | 6 | 8 | 8 | 5 | 40 | 49 | `right--` |
| 5 | 1 | 5 | 8 | 4 | 4 | 16 | 49 | `right--` |
| 6 | 1 | 4 | 8 | 5 | 3 | 15 | 49 | `right--` |
| 7 | 1 | 3 | 8 | 2 | 2 | 4 | 49 | `right--` |
| 8 | 1 | 2 | 8 | 6 | 1 | 6 | 49 | `right--` |

When `left == right`, the loop terminates.

The maximum area encountered was:

\[
\boxed{49}
\]

### Let's understand the most important iterations

**Iteration 1:**

```text
left height = 1
right height = 7
width = 8

area = 8 * min(1, 7)
     = 8
```

The left line is shorter, so we move `left` forward.

Why? Keeping the line of height `1` and moving the other pointer inward would only decrease the width without allowing the effective height to exceed `1`.

**Iteration 2:**

Now:

```text
left height = 8
right height = 7
width = 7

area = 7 * min(8, 7)
     = 49
```

We have found a much larger container!

The right line is shorter, so we move `right` backward.

**The key lesson:** We don't need to check every pair because moving the shorter line eliminates possibilities that cannot improve the area with the current taller line.

---

## Step 8: Tricky Parts, Key Insights, and Revision

### 1. Why do we use `Math.min()`?

Because water cannot rise above the shorter line.

For example:

```text
height[left] = 3
height[right] = 10
width = 5
```

The area is:

```text
5 * min(3, 10) = 15
```

It is not `5 * 10 = 50`, because water would spill over the line of height `3`.

### 2. Why not always move the taller pointer?

Consider:

```text
left height = 3
right height = 10
```

The area is limited by height `3`.

If we move the taller line inward, the width decreases while the shorter line remains unchanged. The effective height cannot increase beyond `3`.

Moving the shorter line gives us a chance to find a taller boundary.

### 3. Why do we calculate the area before moving a pointer?

Because the current pair is a valid container that we must evaluate before discarding either boundary.

The correct order is:

```java
// 1. Calculate the current area
int area = (right - left)
           * Math.min(height[left], height[right]);

// 2. Update the maximum
maxWater = Math.max(maxWater, area);

// 3. Move the shorter pointer
if (height[left] < height[right]) {
    left++;
} else {
    right--;
}
```

### 4. Why is the time complexity `O(n)`?

Every iteration moves one pointer inward. Neither pointer ever moves backward, so there are at most `n - 1` iterations.

Therefore, the time complexity is **`O(n)`**.

### 5. Compare with the problems you've solved

| Problem | Pattern | Pointer/map strategy | Goal |
|---|---|---|---|
| LC 167 — Two Sum II | Two pointers | Move based on whether the sum is too small or too large | Find a pair with a target sum |
| LC 15 — 3Sum | Sorting + two pointers | Fix one element, then adjust pointers based on the sum | Find unique triplets |
| **LC 11 — Container With Most Water** | **Two pointers** | **Move the pointer at the shorter line** | **Maximize the area** |

The underlying technique is similar, but the pointer-movement rule depends on the problem's mathematical property.

### 6. Your DSA-Java GitHub notes

For your repository, record these key points:

- **Problem:** Find two lines that maximize the water container's area.
- **Approach:** Two pointers starting at opposite ends.
- **Pattern:** Greedy + Two Pointers.
- **Time:** `O(n)`.
- **Space:** `O(1)` auxiliary space.
- **Key Insight:** The shorter line limits the water height. Move that line to explore whether a taller boundary can compensate for the reduced width.

---

### Test Your Understanding

Before moving to the next problem, try answering these without looking back:

1. Why is the area calculated using `Math.min(height[left], height[right])`?
2. If `height[left] < height[right]`, why do we increment `left` instead of decrementing `right`?
3. Why can the optimal solution avoid checking every possible pair?
4. What would happen to the time complexity if you used two nested loops?
5. **Coding challenge:** Can you explain, in your own words, why the two-pointer algorithm is guaranteed to find the maximum area?

Focus especially on Question 5. Understanding *why* the greedy choice works is more valuable than simply memorizing the code.
