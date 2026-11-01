# LeetCode 153 — Find Minimum in Rotated Sorted Array



Medium

Binary Search



Pattern: Modified Binary Search

This problem builds directly on LC 33. There, we identified the sorted half and searched for a target. Here, we use the same rotated-array property, but our goal is to find the minimum element.

The important question is:

How can we identify which half contains the minimum without searching every element?

## 1. Understand the Problem

You are given an array of unique integers that was originally sorted in ascending order and then rotated between 1 and `n` times.

Return the minimum element.

### Example 1

```
Input: nums = [3,4,5,1,2]
Output: 1
```

Originally sorted:

```
[1,2,3,4,5]
```

After rotation:

```
[3,4,5,1,2]
```

Minimum element = `1`.

### Example 2

```
Input: nums = [4,5,6,7,0,1,2]
Output: 0
```

### Example 3

```
Input: nums = [11,13,15,17]
Output: 11
```

Even though the array is rotated, it can also look completely sorted.

Important constraints:

- All elements are unique.
- The array was originally sorted.
- We need an `O(log n)` solution.

## 2. Brute-Force Approach — Linear Search

### Idea

Traverse the entire array and keep track of the smallest element.

### Java Code

```
class Solution {
    public int findMin(int[] nums) {

        // Assume the first element is the minimum.
        int min = nums[0];

        // Check every element.
        for (int i = 1; i < nums.length; i++) {

            // Update minimum if a smaller element is found.
            if (nums[i] < min) {
                min = nums[i];
            }
        }

        return min;
    }
}
```

### Complexity

- Time: `O(n)` — we examine every element.
- Space: `O(1)`.

## 3. Identify the Bottleneck

Consider:

```
[4,5,6,7,0,1,2]
```

We know the array contains two sorted portions:

```
[4,5,6,7] | [0,1,2]
```

The minimum is the first element of the right sorted portion.

What if we could determine which half contains the minimum at each iteration?

We could eliminate the other half, just like binary search.

## 4. Recognize the Pattern

The key observation:

The minimum element is the point where the rotated array changes from its larger sorted portion to its smaller sorted portion.

Consider:

```
[4,5,6,7,0,1,2]
```

The minimum is `0`.

We maintain:

```
int left = 0;
int right = nums.length - 1;
```

Calculate:

```
int mid = left + (right - left) / 2;
```

Now compare `nums[mid]` with `nums[right]`.

Why `right`?

Because the rightmost element helps us identify whether the minimum lies to the left or right of `mid`.

There are two important cases.

## 5. Derive the Optimal Approach

### Case 1: `nums[mid] > nums[right]`

Example:

```
[4,5,6,7,0,1,2]
```

Initially:

```
left = 0
right = 6
mid = 3

nums[mid] = 7
nums[right] = 2
```

Comparison:

```
7 > 2
```

What does this tell us?

The middle element belongs to the larger sorted portion, while the rightmost element belongs to the smaller sorted portion.

Therefore, the rotation point — and hence the minimum — must be to the right of `mid`.

So:

```
left = mid + 1;
```

Visual:

```
[4,5,6,7,0,1,2]
 L        M     R

Eliminate:

[4,5,6,7] | [0,1,2]
             Search here
```

### Case 2: `nums[mid] <= nums[right]`

Consider:

```
[4,5,6,7,0,1,2]
```

Suppose our current search range is:

```
[0,1,2]
 L M R
```

Now:

```
left = 4
mid = 5
right = 6

nums[mid] = 1
nums[right] = 2
```

Comparison:

```
1 <= 2
```

This tells us that the right portion is sorted.

The minimum could be:

- At `mid`, or
- Somewhere to the left of `mid`.

Therefore, we cannot eliminate `mid`.

We do:

```
right = mid;
```

Notice the difference: `right = mid`, not `right = mid - 1`.

Why?

Because `nums[mid]` itself might be the minimum.

For example:

```
[1,2,3,4,5]
```

If `mid` points to `1`, we must not eliminate it.

## 6. Optimal Java Solution

O(log n) Time

```
class Solution {
    public int findMin(int[] nums) {

        int left = 0;
        int right = nums.length - 1;

        // Continue until only one candidate remains.
        while (left < right) {

            int mid = left + (right - left) / 2;

            // Minimum lies in the right half.
            if (nums[mid] > nums[right]) {

                left = mid + 1;

            } 
            // Minimum lies at mid or in the left half.
            else {

                right = mid;
            }
        }

        // left and right point to the same minimum element.
        return nums[left];
    }
}
```

### Complexity

- Time: `O(log n)` — we eliminate approximately half the search range each iteration.
- Space: `O(1)` — constant extra space.

## 7. Detailed Dry Run

Let's trace:

```
nums = [4,5,6,7,0,1,2]
```

### Initial state

```
left = 0
right = 6
```

Iteration 1

### 4

0

### 5

1

### 6

2

### 7

3

### 0

4

### 1

5

### 2

6

Left = 0Mid = 3Right = 6

Calculate:

```
mid = 0 + (6 - 0) / 2 = 3

nums[mid] = 7
nums[right] = 2
```

Compare:

```
7 > 2
```

Case 1

Minimum is in the right half.

```
left = mid + 1;
left = 4;
```

Iteration 2

### 4

0

### 5

1

### 6

2

### 7

3

### 0

4

### 1

5

### 2

6

Left = 4Mid = 5Right = 6

Calculate:

```
mid = 4 + (6 - 4) / 2 = 5

nums[mid] = 1
nums[right] = 2
```

Compare:

```
1 <= 2
```

Case 2

Minimum could be at `mid` or to its left.

```
right = mid;
right = 5;
```

Iteration 3

### 4

0

### 5

1

### 6

2

### 7

3

### 0

4

### 1

5

### 2

6

Left = 4Mid = 4Right = 5

Calculate:

```
mid = 4 + (5 - 4) / 2 = 4

nums[mid] = 0
nums[right] = 1
```

Compare:

```
0 <= 1
```

Again, minimum could be at `mid`.

```
right = mid;
right = 4;
```

Loop ends

```
left = 4
right = 4
```

Since `left == right`, only one candidate remains.

```
return nums[left];
```

Answer: 0

### Dry Run Table

| Iteration | Left | Mid | Right | `nums[mid]` | `nums[right]` | Action           |
| --------- | ---- | --- | ----- | ----------- | ------------- | ---------------- |
| 1         | 0    | 3   | 6     | 7           | 2             | `left = mid + 1` |
| 2         | 4    | 5   | 6     | 1           | 2             | `right = mid`    |
| 3         | 4    | 4   | 5     | 0           | 1             | `right = mid`    |

Final answer: `0`

## 8. Tricky Parts You Must Understand

### A. Why compare `nums[mid]` with `nums[right]`, not `nums[left]`?

Comparing with the right boundary gives us a reliable way to distinguish the two cases:

- If `nums[mid] > nums[right]`, the minimum must be to the right of `mid`.
- If `nums[mid] <= nums[right]`, the minimum is at `mid` or to its left.

It makes the two decisions straightforward.

### B. Why `right = mid` instead of `right = mid - 1`?

This is the most important difference from LC 33.

In LC 33, we were searching for a specific target. Once `mid` was checked and wasn't the target, we could eliminate it.

In LC 153, we are searching for the minimum. We haven't ruled out `mid` as the minimum.

Example:

```
nums = [3,4,5,1,2]
```

Suppose:

```
mid = 3
nums[mid] = 1
nums[right] = 2
```

Since `1 <= 2`, the minimum might be `nums[mid]` itself.

If we used:

```
right = mid - 1;
```

We would eliminate the correct answer!

Therefore:

```
right = mid;
```

### C. Why `while (left < right)` instead of `left <= right`?

Because we want to continue until exactly one candidate remains.

When:

```
left == right
```

We have found the minimum's index.

We don't need another iteration.

### D. What if the array is not rotated?

Example:

```
nums = [1,2,3,4,5]
```

Initially:

```
left = 0
right = 4
mid = 2

nums[mid] = 3
nums[right] = 5
```

Since:

```
3 <= 5
```

We do:

```
right = mid;
```

The search keeps shrinking toward index `0`, eventually returning `1`.

So the algorithm handles an unrotated array too.

## Compare LC 33 and LC 153

Both use modified binary search, but their goals and pointer movements differ.

| Concept              | LC 33                | LC 153                                |
| -------------------- | -------------------- | ------------------------------------- |
| Goal                 | Find target index    | Find minimum value                    |
| Main comparison      | Identify sorted half | Compare `nums[mid]` and `nums[right]` |
| Loop condition       | `left <= right`      | `left < right`                        |
| Right pointer update | `right = mid - 1`    | `right = mid`                         |
| Left pointer update  | `left = mid + 1`     | `left = mid + 1`                      |
| Return               | Index or `-1`        | `nums[left]`                          |

Remember: LC 33 eliminates `mid` after checking it. LC 153 keeps `mid` as a candidate when it could be the minimum.

## GitHub-Ready Notes

```
Problem: LeetCode 153 — Find Minimum in Rotated Sorted Array

Pattern:
Modified Binary Search

Approach:
1. Initialize left = 0 and right = n - 1.
2. While left < right:
   - Calculate mid.
   - If nums[mid] > nums[right]:
       Minimum lies in the right half.
       left = mid + 1.
   - Otherwise:
       Minimum is at mid or in the left half.
       right = mid.
3. Return nums[left].

Time Complexity: O(log n)
Space Complexity: O(1)

Key Insight:
Compare the middle element with the rightmost element.
This determines which side contains the minimum.

Important:
Use right = mid, not mid - 1, because mid itself
might be the minimum.
Use left < right because we stop when one candidate remains.
```

## Test Your Understanding

Let's check whether you can apply the logic independently.

1\. nums = [5,6,1,2,3,4]. If mid = 2, what happens?

left = mid + 1

right = mid

Return nums[mid]

2\. Why do we use right = mid instead of right = mid - 1?

Because mid might itself be the minimum

Because the array is not sorted

To make the loop run faster

3\. What is the loop condition?

left <= right

left < right

left != 0

4\. nums = [3,4,5,1,2]. If nums[mid] > nums[right], where is the minimum?

At mid or to its left

Strictly to the right of mid

At index 0

Check answers

Next recommended problem: LC 154 — Find Minimum in Rotated Sorted Array II. It is the natural extension of this problem, where duplicates are allowed and binary search becomes slightly more challenging.
