# LeetCode 42: Trapping Rain Water

We'll solve this using your usual **8-step DSA framework**, with commented Java code for both brute force and optimal approaches.

This problem is especially important for your Infosys SP preparation because it combines **arrays, precomputation, and two pointers**. Since you've already solved Product of Array Except Self and Container With Most Water, we'll connect the concepts to those problems as we go.

## Step 1: Understand the Problem

You are given an integer array `height`, where `height[i]` represents the height of a vertical bar. Each bar has a width of `1`.

After it rains, calculate how much water can be trapped between the bars.

### Example

```text
Input:  height = [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
```

Imagine the bars as a landscape. Water collects in the valleys between taller bars.



### The fundamental idea

Water above a bar is limited by the tallest boundary on its left and the tallest boundary on its right.

For any index `i`:

\[
\text{water}[i]
=
\max\left(0,\min(\text{leftMax}[i],\text{rightMax}[i])-\text{height}[i]\right)
\]

Where:

- `leftMax[i]` = maximum height from index `0` through index `i`.
- `rightMax[i]` = maximum height from index `i` through the last index.
- `height[i]` = the height of the current bar.

**Why take the minimum of the two maximum heights?**

Imagine the tallest bar to the left is `4`, and the tallest bar to the right is `3`. Water cannot rise above height `3`, because it would spill over the right boundary.

If the current bar is height `1`, the trapped water is:

\[
\min(4,3)-1=2
\]

We calculate this for every index and add the results.

---

## Step 2: Brute-Force Approach

### Idea

For every index `i`:

1. Find the tallest bar to its left.
2. Find the tallest bar to its right.
3. Calculate the water trapped above the current bar.
4. Add that amount to the total.

We can find the left and right maximums by scanning the array each time.

### Java Code — Brute Force

```java
class Solution {
    public int trap(int[] height) {
        int n = height.length;
        int totalWater = 0;

        // Check how much water can be trapped above each bar
        for (int i = 0; i < n; i++) {

            int leftMax = 0;
            int rightMax = 0;

            // Find the tallest bar from the beginning through index i
            for (int j = 0; j <= i; j++) {
                leftMax = Math.max(leftMax, height[j]);
            }

            // Find the tallest bar from index i through the end
            for (int j = i; j < n; j++) {
                rightMax = Math.max(rightMax, height[j]);
            }

            // Water level is limited by the shorter maximum boundary
            int waterLevel = Math.min(leftMax, rightMax);

            // Only the space above the current bar can hold water
            int waterAtCurrentIndex = waterLevel - height[i];

            // Add the trapped water to the total
            totalWater += waterAtCurrentIndex;
        }

        return totalWater;
    }
}
```

### Complexity

- **Time:** `O(n²)` — for each index, we scan the array to find the two maximums.
- **Auxiliary space:** `O(1)` — we use only a few variables.

This approach is correct, but it repeatedly calculates the same maximum heights.

For example, the tallest bar on the left of index `5` is also relevant when processing indices `6`, `7`, and beyond.

**Our next goal is to avoid recalculating information we've already discovered.**

---

## Step 3: Identify the Bottleneck

Look at the brute-force code:

```java
for (int i = 0; i < n; i++) {

    // Repeatedly scan the left side
    for (int j = 0; j <= i; j++) {
        leftMax = Math.max(leftMax, height[j]);
    }

    // Repeatedly scan the right side
    for (int j = i; j < n; j++) {
        rightMax = Math.max(rightMax, height[j]);
    }
}
```

The problem is **repeated work**.

Suppose the tallest bar among the first five elements is `4`. When processing the sixth element, we don't need to scan those first five elements again to discover that maximum.

We can improve this in two ways:

1. **Prefix and suffix maximum arrays:** Precompute the maximum height on the left and right of every position.
2. **Two pointers:** Maintain the relevant maximum heights as we traverse the array, avoiding extra arrays.

Let's understand both.

---

## Step 4: Recognize the Pattern

This problem connects three important concepts:

| Concept | How it helps |
|---|---|
| Prefix maximum | Tracks the tallest bar encountered from the left |
| Suffix maximum | Tracks the tallest bar encountered from the right |
| Two pointers | Calculates trapped water without storing two additional arrays |

### Connection to LC 238: Product of Array Except Self

In LC 238, you used prefix and suffix products to avoid recalculating products for every index.

Here, we use a similar idea, but instead of products, we calculate **maximum heights**.

For example, with:

```text
height = [3, 0, 2, 0, 4]
```

The prefix maximum at each index is:

```text
height:    [3, 0, 2, 0, 4]
leftMax:   [3, 3, 3, 3, 4]
```

The suffix maximum at each index is:

```text
height:    [3, 0, 2, 0, 4]
rightMax:  [4, 4, 4, 4, 4]
```

For every position, the water level is the minimum of the two maximums.

For index `1`:

```text
height[1] = 0
leftMax[1] = 3
rightMax[1] = 4

water = min(3, 4) - 0
      = 3
```

This is exactly the prefix/suffix concept you already know.

However, we can do even better by using two pointers and `O(1)` auxiliary space.

---

## Step 5: Derive the Optimal Approach

### Approach A: Prefix and Suffix Maximum Arrays

Before jumping into the most space-efficient solution, let's understand this intermediate optimization.

#### Step 1: Build `leftMax`

For every index, store the maximum height seen from the beginning up to that index.

```java
leftMax[0] = height[0];

for (int i = 1; i < n; i++) {
    leftMax[i] = Math.max(leftMax[i - 1], height[i]);
}
```

Notice that we reuse the previous maximum rather than scanning all earlier elements again.

#### Step 2: Build `rightMax`

Start at the end and work backward.

```java
rightMax[n - 1] = height[n - 1];

for (int i = n - 2; i >= 0; i--) {
    rightMax[i] = Math.max(rightMax[i + 1], height[i]);
}
```

#### Step 3: Calculate the trapped water

```java
for (int i = 0; i < n; i++) {
    int waterLevel = Math.min(leftMax[i], rightMax[i]);
    totalWater += waterLevel - height[i];
}
```

Each position can now be processed in constant time after precomputation.

**Complexity:**
- Time: `O(n)`
- Auxiliary space: `O(n)` for the two arrays.

This is a good solution, but we can eliminate those arrays.

### Approach B: Optimal Two-Pointer Solution

The key observation is:

**If the maximum height encountered from the left is less than or equal to the maximum height encountered from the right, the left side determines the water level at the current left position.**

Similarly, if the right maximum is smaller, the right side determines the water level at the current right position.

Why?

The water level is determined by:

\[
\min(\text{leftMax},\text{rightMax})
\]

If `leftMax <= rightMax`, we already know the left maximum is the limiting boundary. The right side has a boundary at least as tall as the left maximum, so we don't need to know its exact maximum to calculate the water at the current left position.

We can safely process that position.

### Step-by-step derivation

Initialize:

```java
int left = 0;
int right = height.length - 1;

int leftMax = 0;
int rightMax = 0;

int totalWater = 0;
```

These variables mean:

- `left`: Current position being considered from the left.
- `right`: Current position being considered from the right.
- `leftMax`: Maximum height encountered while moving from the left.
- `rightMax`: Maximum height encountered while moving from the right.
- `totalWater`: Total trapped water calculated so far.

Then process the array while the pointers have not crossed.

```java
while (left < right) {
    if (height[left] <= height[right]) {
        // Process the left position
    } else {
        // Process the right position
    }
}
```

Why compare the current heights?

This is a useful way to choose which side can be processed safely. If the left current height is smaller than or equal to the right current height, the right current bar provides a boundary at least as tall as the left current bar. The left side can therefore be processed using `leftMax`.

If the right current height is smaller, process the right side using `rightMax`.

#### Case 1: Process the left side

```java
if (height[left] <= height[right]) {
    if (height[left] >= leftMax) {
        leftMax = height[left];
    } else {
        totalWater += leftMax - height[left];
    }

    left++;
}
```

There are two possibilities.

**Possibility A: The current bar is taller than or equal to `leftMax`.**

```java
leftMax = height[left];
```

This bar establishes a new maximum from the left. No water is trapped above it because there is no taller left boundary that could hold water above it.

**Possibility B: The current bar is shorter than `leftMax`.**

```java
totalWater += leftMax - height[left];
```

Since the right side is guaranteed to have a boundary at least as tall as the current left bar, the water at this position is determined by `leftMax`.

For example:

```text
leftMax = 3
height[left] = 1

water = 3 - 1 = 2
```

We add `2` units to the total.

Finally, move `left` forward.

#### Case 2: Process the right side

```java
else {
    if (height[right] >= rightMax) {
        rightMax = height[right];
    } else {
        totalWater += rightMax - height[right];
    }

    right--;
}
```

This is the mirror image of the left-side logic.

If the current right bar is at least as tall as `rightMax`, update `rightMax`.

Otherwise, the water above the bar is:

```java
rightMax - height[right]
```

Add it to the total and move `right` backward.

### The complete logic

```text
Initialize left, right, leftMax, rightMax, totalWater

While left < right:

    If height[left] <= height[right]:

        If height[left] >= leftMax:
            Update leftMax
        Else:
            Add leftMax - height[left] to totalWater

        Move left forward

    Else:

        If height[right] >= rightMax:
            Update rightMax
        Else:
            Add rightMax - height[right] to totalWater

        Move right backward

Return totalWater
```

---

## Step 6: Optimal Java Solution — With Comments

```java
class Solution {
    public int trap(int[] height) {

        // Pointers starting at opposite ends of the array
        int left = 0;
        int right = height.length - 1;

        // Maximum heights encountered from each side
        int leftMax = 0;
        int rightMax = 0;

        // Total amount of trapped rainwater
        int totalWater = 0;

        // Process positions until the pointers meet
        while (left < right) {

            // If the left current height is smaller or equal,
            // the left side determines the water at this position
            if (height[left] <= height[right]) {

                // A new maximum from the left means no water
                // can be trapped above this bar
                if (height[left] >= leftMax) {
                    leftMax = height[left];
                } else {

                    // The left maximum is taller than this bar,
                    // so the difference is the trapped water
                    totalWater += leftMax - height[left];
                }

                // Move to the next position from the left
                left++;

            } else {

                // Process the right position when its current
                // height is smaller than the left current height

                // A new maximum from the right means no water
                // can be trapped above this bar
                if (height[right] >= rightMax) {
                    rightMax = height[right];
                } else {

                    // The right maximum is taller than this bar,
                    // so the difference is the trapped water
                    totalWater += rightMax - height[right];
                }

                // Move to the next position from the right
                right--;
            }
        }

        // Return the total trapped rainwater
        return totalWater;
    }
}
```

### Complexity Analysis

| Complexity | Explanation |
|---|---|
| **Time: `O(n)`** | Each iteration processes one position and moves a pointer inward. |
| **Auxiliary space: `O(1)`** | Only pointers, maximum heights, and the running total are stored. |

### Which solution should you remember?

For coding assessments, understand both:

- **Prefix/suffix arrays:** Often easier to derive because they directly implement the water-level formula.
- **Two pointers:** More space-efficient and achieves `O(n)` time with `O(1)` auxiliary space.

If you are struggling to derive the two-pointer solution independently, derive the prefix/suffix solution first. It makes the reasoning behind the optimized version much clearer.

---

## Step 7: Dry Run the Optimal Solution

Let's use a smaller example first:

```text
height = [3, 0, 2, 0, 4]
```

The correct answer is `7`.

The bars have heights:

```text
Index:      0  1  2  3  4
Height:     3  0  2  0  4
```

The water trapped at each position is:

| Index | Height | Left maximum | Right maximum | Water trapped |
|---:|---:|---:|---:|---:|
| 0 | 3 | 3 | 4 | 0 |
| 1 | 0 | 3 | 4 | 3 |
| 2 | 2 | 3 | 4 | 1 |
| 3 | 0 | 3 | 4 | 3 |
| 4 | 4 | 4 | 4 | 0 |

Therefore:

\[
3+1+3=\boxed{7}
\]

Now let's trace the actual two-pointer algorithm.

### Initial state

```text
left = 0
right = 4

leftMax = 0
rightMax = 0

totalWater = 0
```

### Iteration 1

```text
height[left] = 3
height[right] = 4
```

Since `3 <= 4`, process the left side.

```java
if (height[left] >= leftMax)
```

Since `3 >= 0`, update:

```text
leftMax = 3
```

No water is added.

Move `left` forward:

```text
left = 1
```

### Iteration 2

```text
height[left] = 0
height[right] = 4
leftMax = 3
```

Since `0 <= 4`, process the left side.

The current height is below `leftMax`, so:

```text
water = leftMax - height[left]
      = 3 - 0
      = 3
```

Update:

```text
totalWater = 3
left = 2
```

### Iteration 3

```text
height[left] = 2
height[right] = 4
leftMax = 3
```

Since `2 <= 4`, process the left side again.

```text
water = leftMax - height[left]
      = 3 - 2
      = 1
```

Update:

```text
totalWater = 4
left = 3
```

### Iteration 4

```text
height[left] = 0
height[right] = 4
leftMax = 3
```

Again, process the left side.

```text
water = 3 - 0 = 3
```

Update:

```text
totalWater = 7
left = 4
```

Now `left == right`, so the loop ends.

Final answer:

```text
Output: 7
```

Notice that the algorithm never needed to build either maximum array.

---

## Step 8: Tricky Parts, Key Insights, and Revision

### 1. Why do we compare `height[left]` and `height[right]` instead of `leftMax` and `rightMax`?

The current heights determine which side can be processed safely.

When:

```java
height[left] <= height[right]
```

the right current bar is at least as tall as the left current bar. The left position can therefore be resolved using the maximum height encountered on the left.

Likewise, when the right current bar is shorter, the right position can be resolved using `rightMax`.

**Remember:** The current-height comparison chooses the side; the corresponding maximum height determines how much water is trapped there.

### 2. Why do we update the maximum before calculating water?

Consider:

```text
leftMax = 2
height[left] = 5
```

The current bar is taller than the previous maximum.

If we calculated:

```text
water = leftMax - height[left]
      = 2 - 5
      = -3
```

we would get an invalid negative amount of water.

Instead, update `leftMax` to `5`. The current bar establishes a new boundary and traps no water above itself.

The same principle applies to `rightMax`.

### 3. Why do we use `>=` rather than `>`?

Either comparison works for this particular algorithm if used consistently with the corresponding `else` branch. Using `>=` makes the meaning clear: the current bar is at least as tall as the previous maximum, so update the maximum and add no water at that position.

### 4. Comparison with your previous problems

| Problem | Main idea | What we track |
|---|---|---|
| LC 238 — Product of Array Except Self | Prefix and suffix | Products |
| LC 560 — Subarray Sum Equals K | Prefix sum + HashMap | Prefix-sum frequencies |
| LC 11 — Container With Most Water | Two pointers | Current boundary heights |
| **LC 42 — Trapping Rain Water** | **Two pointers + greedy processing** | **Maximum heights from both sides** |

The important distinction from Container With Most Water is that **LC 11 calculates the area between two selected lines, while LC 42 adds the water trapped above individual bars**.

### 5. Your DSA-Java GitHub notes

Use this summary for your problem file:

- **Problem:** Calculate the total rainwater trapped between bars.
- **Brute force:** Find the left and right maximum for each index.
- **Intermediate approach:** Precompute prefix and suffix maximum arrays.
- **Optimal approach:** Two pointers with running maximum heights.
- **Pattern:** Two Pointers + Greedy.
- **Brute-force time:** `O(n²)`.
- **Optimal time:** `O(n)`.
- **Optimal auxiliary space:** `O(1)`.
- **Key Insight:** Process the side whose current height is smaller or equal. Its corresponding running maximum determines the water at that position.

---

## Practice Questions

Before moving to the next problem, try answering these five questions:

1. Why is the water level at an index determined by `min(leftMax, rightMax)`?
2. Why does the two-pointer solution process the left position when `height[left] <= height[right]`?
3. What is the difference between the prefix/suffix solution and the two-pointer solution in terms of space complexity?
4. Why do we add `leftMax - height[left]` instead of simply adding `leftMax`?
5. **Most important:** Explain why comparing the current heights allows us to calculate the water on one side without knowing the exact maximum height on the opposite side.

If you can explain Question 5 clearly, you've understood the central insight of Trapping Rain Water rather than merely memorizing its code.
