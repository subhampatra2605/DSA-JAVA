# LeetCode 15 — 3Sum

We'll solve **3Sum using our usual Step 1–8 framework**, starting with brute force and then optimizing it using sorting and two pointers.

This problem is especially important for coding assessments because it combines two patterns you've already learned:
- **Sorting**
- **Two pointers (LC 167)**

The new challenge is handling **duplicate triplets** correctly.

---

## Step 1: Understand the Problem

You are given an integer array `nums`. Find all unique triplets `[nums[i], nums[j], nums[k]]` such that:

```text
nums[i] + nums[j] + nums[k] = 0
```

The indices must be different:

```text
i != j
i != k
j != k
```

You must return all unique triplets. The answer must not contain duplicate triplets.

### Example

```text
Input: nums = [-1, 0, 1, 2, -1, -4]

Output: [[-1, -1, 2], [-1, 0, 1]]
```

Explanation:

The first triplet satisfies:

```text
-1 + -1 + 2 = 0
```

The second triplet satisfies:

```text
-1 + 0 + 1 = 0
```

Both triplets have a sum of zero.

Although `-1` appears twice in the array, we return each unique triplet only once.

**Important:** The order of the triplets in the output does not matter.

---

## Step 2: Think of the Brute-Force Approach

Before optimizing, think about how you would find every possible triplet.

### Logic

1. Choose the first element using loop `i`.
2. Choose the second element using loop `j`, starting from `i + 1`.
3. Choose the third element using loop `k`, starting from `j + 1`.
4. Check whether their sum equals zero.
5. If it does, add the triplet to the result.
6. Avoid duplicate triplets.

One way to handle duplicates is to sort each triplet and store it in a `Set`, so identical triplets are stored only once.

### Brute-Force Code — Java

```java
import java.util.*;

class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Set<List<Integer>> uniqueTriplets = new HashSet<>();

        int n = nums.length;

        // Choose the first element
        for (int i = 0; i < n; i++) {

            // Choose the second element after the first
            for (int j = i + 1; j < n; j++) {

                // Choose the third element after the second
                for (int k = j + 1; k < n; k++) {

                    // Check whether the three elements sum to zero
                    if (nums[i] + nums[j] + nums[k] == 0) {

                        // Store the three values in a list
                        List<Integer> triplet = Arrays.asList(
                            nums[i], nums[j], nums[k]
                        );

                        // Sort the triplet so different orderings
                        // of the same values become identical
                        Collections.sort(triplet);

                        // A Set automatically prevents duplicate triplets
                        uniqueTriplets.add(triplet);
                    }
                }
            }
        }

        // Convert the set of unique triplets into a list
        return new ArrayList<>(uniqueTriplets);
    }
}
```

### Complexity

- **Time:** `O(n³)` — three nested loops examine every possible triplet. Sorting each three-element triplet takes constant time.
- **Auxiliary space:** `O(u)` for the set of unique triplets, excluding the returned result, where `u` is the number of unique triplets stored.

### Identify the Bottleneck

The brute-force approach examines every possible combination of three elements.

For an array of `n` elements, this takes cubic time.

**Can we fix one element and find the other two using the two-pointer technique from LC 167?**

Yes! That is the key optimization.

---

## Step 3: Find the Pattern

The key observation is:

**If we fix one number, finding the other two numbers becomes a Two Sum problem.**

Suppose:

```text
nums = [-1, 0, 1, 2, -1, -4]
```

First, sort the array:

```text
[-4, -1, -1, 0, 1, 2]
```

Now suppose we fix the first element:

```text
fixed = -1
```

We need two other numbers whose sum is:

```text
target = 0 - (-1) = 1
```

In other words:

```text
nums[left] + nums[right] = 1
```

We can use two pointers to find those numbers.

For example:

```text
Sorted array: [-4, -1, -1, 0, 1, 2]
                  ↑
                fixed
```

The remaining elements can be searched using `left` and `right`.

This reduces the problem from checking every triplet to:

1. Fix one element.
2. Use two pointers to find the other two.

Sorting also allows us to skip duplicate values efficiently.

**Pattern identified: Sorting + Fixed Element + Two Pointers.**

---

## Step 4: Derive the Optimal Approach

Let's derive the solution step by step.

### Part A: Sort the Array

```java
Arrays.sort(nums);
```

Why?

Sorting allows us to:
- Use two pointers.
- Move pointers predictably based on the sum.
- Skip duplicate values easily.

For our example:

```text
Original: [-1, 0, 1, 2, -1, -4]
Sorted:   [-4, -1, -1, 0, 1, 2]
```

### Part B: Fix the First Element

We iterate through the array:

```java
for (int i = 0; i < nums.length - 2; i++)
```

Why stop at `nums.length - 2`?

We need at least two elements after `i` to form a triplet.

For each fixed element, the required sum of the other two elements is:

```java
int target = -nums[i];
```

This is because:

```text
nums[i] + nums[left] + nums[right] = 0
```

Rearranging:

```text
nums[left] + nums[right] = -nums[i]
```

### Part C: Initialize Two Pointers

```java
int left = i + 1;
int right = nums.length - 1;
```

The `left` pointer begins immediately after the fixed element.

The `right` pointer begins at the end of the array.

Now calculate:

```java
int sum = nums[i] + nums[left] + nums[right];
```

There are three possibilities.

**Case 1: `sum == 0`**

We have found a valid triplet.

Add it to the result:

```java
result.add(Arrays.asList(nums[i], nums[left], nums[right]));
```

Then move both pointers:

```java
left++;
right--;
```

Why both?

We have already recorded this triplet. Moving both pointers lets us search for another candidate without reusing the same pair.

**Case 2: `sum < 0`**

The sum is too small.

Because the array is sorted, we need a larger value.

Move the left pointer:

```java
left++;
```

**Case 3: `sum > 0`**

The sum is too large.

We need a smaller value.

Move the right pointer:

```java
right--;
```

### Part D: Handle Duplicate Values

This is the trickiest part of LC 15.

Consider:

```text
[-4, -1, -1, 0, 1, 2]
      ↑   ↑
```

The value `-1` appears twice.

If we fix the first `-1`, we may discover valid triplets.

If we fix the second `-1` as well, we may discover the same triplets again.

We can prevent this with:

```java
if (i > 0 && nums[i] == nums[i - 1]) {
    continue;
}
```

This skips repeated fixed values.

**Why check `i > 0`?**

Because `nums[i - 1]` would be invalid when `i == 0`.

We must also skip duplicates after finding a valid triplet.

For the left pointer:

```java
while (left < right && nums[left] == nums[left - 1]) {
    left++;
}
```

For the right pointer:

```java
while (left < right && nums[right] == nums[right + 1]) {
    right--;
}
```

These checks prevent the same triplet from being added multiple times.

### Part E: Use an Early Break

Because the array is sorted, if the fixed element is positive:

```java
if (nums[i] > 0) {
    break;
}
```

All remaining elements are also positive or zero, so adding any three of them cannot produce a sum of zero.

This optimization is optional, but useful.

---

## Step 5: Write the Optimal Algorithm

### Optimal Code — Java

```java
import java.util.*;

class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();

        // Sort the array to enable the two-pointer technique
        Arrays.sort(nums);

        int n = nums.length;

        // Fix the first element of each triplet
        for (int i = 0; i < n - 2; i++) {

            // Skip duplicate fixed elements
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }

            // All remaining values are non-negative, so no
            // triplet starting here can sum to zero
            if (nums[i] > 0) {
                break;
            }

            // Find two numbers whose sum equals -nums[i]
            int left = i + 1;
            int right = n - 1;

            while (left < right) {

                // Calculate the sum of the current triplet
                int sum = nums[i] + nums[left] + nums[right];

                // If the sum is zero, we found a valid triplet
                if (sum == 0) {
                    result.add(Arrays.asList(
                        nums[i], nums[left], nums[right]
                    ));

                    // Move both pointers to search for another pair
                    left++;
                    right--;

                    // Skip duplicate values at the left pointer
                    while (left < right &&
                           nums[left] == nums[left - 1]) {
                        left++;
                    }

                    // Skip duplicate values at the right pointer
                    while (left < right &&
                           nums[right] == nums[right + 1]) {
                        right--;
                    }
                }

                // If the sum is too small, increase it
                else if (sum < 0) {
                    left++;
                }

                // If the sum is too large, decrease it
                else {
                    right--;
                }
            }
        }

        // Return all unique triplets
        return result;
    }
}
```

### Complexity

- **Time:** `O(n²)` — sorting takes `O(n log n)`, and the fixed-element loop performs an `O(n)` two-pointer scan for each element.
- **Auxiliary space:** `O(1)` for the two-pointer search, excluding the returned result. Java's sorting implementation may use additional space depending on the implementation.

The returned result itself can contain `O(n²)` triplets in the general complexity analysis.

---

## Step 6: Dry Run the Optimal Code

Let's use:

```text
nums = [-1, 0, 1, 2, -1, -4]
```

After sorting:

```text
nums = [-4, -1, -1, 0, 1, 2]
```

### Iteration 1: `i = 0`

The fixed element is:

```text
nums[i] = -4
```

Initialize:

```text
left = 1
right = 5
```

Array:

```text
[-4, -1, -1, 0, 1, 2]
  ↑    ↑              ↑
 fixed left          right
```

**Check 1:**

```text
sum = -4 + (-1) + 2
    = -3
```

The sum is negative, so move `left` rightward.

```text
left = 2
```

**Check 2:**

```text
sum = -4 + (-1) + 2
    = -3
```

Again, the sum is negative.

Move `left` rightward.

**Check 3:**

```text
sum = -4 + 0 + 2
    = -2
```

The sum is still negative.

Move `left` rightward.

**Check 4:**

```text
sum = -4 + 1 + 2
    = -1
```

The sum remains negative.

Move `left` rightward. Now `left == right`, so this search ends.

No valid triplet was found for `-4`.

### Iteration 2: `i = 1`

The fixed element is:

```text
nums[i] = -1
```

Initialize:

```text
left = 2
right = 5
```

**Check 1:**

```text
sum = -1 + (-1) + 2
    = 0
```

We found:

```text
[-1, -1, 2]
```

Add it to the result.

Move both pointers:

```text
left = 3
right = 4
```

**Check 2:**

```text
sum = -1 + 0 + 1
    = 0
```

We found another valid triplet:

```text
[-1, 0, 1]
```

Add it to the result.

Move both pointers:

```text
left = 4
right = 3
```

Since `left >= right`, this search ends.

Current result:

```text
[[-1, -1, 2], [-1, 0, 1]]
```

### Iteration 3: `i = 2`

The fixed element is again `-1`.

Check:

```java
if (i > 0 && nums[i] == nums[i - 1])
```

Both conditions are true.

We skip this iteration to avoid generating duplicate triplets.

### Iteration 4: `i = 3`

The fixed element is `0`.

Initialize:

```text
left = 4
right = 5
```

Calculate:

```text
sum = 0 + 1 + 2
    = 3
```

The sum is positive, so move `right` leftward.

Now `left == right`, so this search ends.

No valid triplet is found for `0`.

### Iteration 5: `i = 4`

There are not two elements after this index, so the loop condition `i < n - 2` fails.

The algorithm ends.

### Final Output

```text
[[-1, -1, 2], [-1, 0, 1]]
```

---

## Step 7: Understand the Tricky Parts

### 1. Why sort the array?

Sorting makes the two-pointer technique possible and makes duplicate values easy to identify.

For example:

```text
Original: [1, -1, 0, -1]
Sorted:   [-1, -1, 0, 1]
```

Duplicate values become adjacent.

### 2. Why skip duplicate fixed elements?

```java
if (i > 0 && nums[i] == nums[i - 1]) {
    continue;
}
```

If we process the same fixed value more than once, we may generate the same triplets again.

Skipping repeated fixed values avoids that.

### 3. Why skip duplicate left and right values after finding a triplet?

Suppose we have already found:

```text
[-1, 0, 1]
```

If the next left or right element has the same value as the previous one, we may generate the same triplet again.

Skipping duplicates ensures that each unique triplet is returned only once.

### 4. Why move both pointers when the sum equals zero?

Once a valid triplet is found, we record it and move both pointers to look for another pair.

We then skip duplicate values so the same triplet is not repeated.

### 5. Why break when `nums[i] > 0`?

The array is sorted.

If the fixed element is positive, every element after it is also positive. Their sum cannot be zero.

Therefore, we can stop searching.

### 6. Why not use a HashSet for the optimal solution?

A HashSet can help remove duplicate triplets, but the sorted two-pointer approach handles duplicates directly.

This avoids needing a separate set of triplets and gives us the standard optimal `O(n²)` solution.

---

## Step 8: Complexity Analysis and Final Revision

### Brute Force vs. Optimal

| Approach | Time Complexity | Main Idea |
|---|---|---|
| Brute force | `O(n³)` | Check every possible triplet |
| Optimal | `O(n²)` | Sort, fix one element, then use two pointers |

### The Complete Logic to Remember

When you encounter **3Sum**, think:

1. Sort the array.
2. Fix one element using a loop.
3. Skip duplicate fixed values.
4. If the fixed element is positive, stop searching.
5. Initialize `left = i + 1` and `right = n - 1`.
6. Calculate the sum of the three selected elements.
7. If the sum is zero, record the triplet, move both pointers, and skip duplicates.
8. If the sum is negative, move `left` rightward.
9. If the sum is positive, move `right` leftward.
10. Return all unique triplets.

### Connection to LC 167

| Feature | LC 167 — Two Sum II | LC 15 — 3Sum |
|---|---|---|
| Number of elements | Two | Three |
| Sorting | Array is already sorted | Sort the array first |
| Main technique | Two pointers | Fix one element + two pointers |
| Duplicate handling | Not the main challenge | Essential |
| Time complexity | `O(n)` | `O(n²)` |

**The key lesson:** 3Sum is essentially Two Sum repeated for every possible fixed element, with sorting and duplicate handling added.

### Your Practice Questions

Before moving to the next problem, try answering these without looking at the solution:

1. Why do we sort the array before using two pointers?
2. Why do we set `left = i + 1` rather than `left = 0`?
3. Why do we skip duplicate fixed elements?
4. When the sum is negative, why do we move `left` instead of `right`?
5. Why is the optimal time complexity `O(n²)` rather than `O(n³)`?

If you can explain these five points and implement the solution independently, you've understood the core pattern behind **LeetCode 15 — 3Sum**.
