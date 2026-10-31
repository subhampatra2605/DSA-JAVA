# LeetCode 704 — Binary Search



Easy

Binary Search



Pattern: Binary Search on a Sorted Array

Goal: Understand how binary search eliminates half of the search space in every iteration, and why the boundary conditions matter.

## 1. Understand the Problem

You are given an array of integers `nums`, sorted in ascending order, and an integer `target`.

Return the index of `target` if it exists in the array. Otherwise, return `-1`.

### Example 1

```
Input: nums = [-1,0,3,5,9,12], target = 9
Output: 4
```

Because `nums[4] = 9`.

### Example 2

```
Input: nums = [-1,0,3,5,9,12], target = 2
Output: -1
```

Because `2` does not exist in the array.

Important observation: The array is already sorted. This allows us to use Binary Search.

## 2. Brute-Force Approach — Linear Search

### Idea

Start from the first element and check every element until:

- We find the target → return its index.
- We reach the end → return `-1`.

### Java Code

```
class Solution {
    public int search(int[] nums, int target) {

        // Check every element in the array.
        for (int i = 0; i < nums.length; i++) {

            // If the current element matches the target,
            // return its index.
            if (nums[i] == target) {
                return i;
            }
        }

        // Target was not found.
        return -1;
    }
}
```

### Complexity

- Time: `O(n)` — in the worst case, we check every element.
- Space: `O(1)`.

## 3. Identify the Bottleneck

Consider this array:

```
[-1, 0, 3, 5, 9, 12]
```

Suppose the target is `9`.

Linear search:

```
Check -1 → No
Check  0 → No
Check  3 → No
Check  5 → No
Check  9 → Found!
```

We checked five elements.

But notice something important: because the array is sorted, we don't need to check every element.

We can look at the middle element and eliminate half of the remaining search space.

This is the central idea of Binary Search.

## 4. Recognize the Pattern

Binary Search works when the search space is ordered and we can eliminate one side based on a comparison.

We maintain two pointers:

```
int left = 0;
int right = nums.length - 1;
```

They represent the current search range.

Calculate the middle index:

```
int mid = left + (right - left) / 2;
```

Now compare:

| Condition             | What it tells us                | Action            |
| --------------------- | ------------------------------- | ----------------- |
| `nums[mid] == target` | Target found                    | Return `mid`      |
| `nums[mid] < target`  | Target can only be on the right | `left = mid + 1`  |
| `nums[mid] > target`  | Target can only be on the left  | `right = mid - 1` |

Why?

Because the array is sorted in ascending order.

For example:

```
[-1, 0, 3, 5, 9, 12]
```

If the middle element is `3` and the target is `9`, everything to the left of `3` can be eliminated.

## 5. Derive the Optimal Approach

Let's understand the pointer movement.

Initially:

```
left = 0
right = 5
```

Our search space is:

```
Index:  0   1   2   3   4   5
Value: -1   0   3   5   9  12
        L           R
```

Calculate middle:

```
mid = left + (right - left) / 2;
mid = 0 + (5 - 0) / 2;
mid = 2;
```

So:

```
nums[mid] = nums[2] = 3
```

Suppose target is `9`.

Since:

```
3 < 9
```

We know the target must be to the right of index `2`.

Therefore:

```
left = mid + 1;
```

Now:

```
left = 3
right = 5
```

We have eliminated indices `0`, `1`, and `2`.

Repeat until the target is found or the search range becomes empty.

## 6. Optimal Java Solution

Optimal — O(log n)

```
class Solution {
    public int search(int[] nums, int target) {

        // Initialize the search boundaries.
        int left = 0;
        int right = nums.length - 1;

        // Continue while the search range is not empty.
        while (left <= right) {

            // Calculate the middle index safely.
            int mid = left + (right - left) / 2;

            // Case 1: Target found.
            if (nums[mid] == target) {
                return mid;
            }

            // Case 2: Target is greater than middle element.
            // Search in the right half.
            else if (nums[mid] < target) {
                left = mid + 1;
            }

            // Case 3: Target is smaller than middle element.
            // Search in the left half.
            else {
                right = mid - 1;
            }
        }

        // Search range is empty; target does not exist.
        return -1;
    }
}
```

### Complexity

- Time: `O(log n)` — the search space is approximately halved in every iteration.
- Space: `O(1)` — we only use a few variables.

## 7. Detailed Dry Run

Let's take:

```
nums = [-1, 0, 3, 5, 9, 12]
target = 9
```

Iteration 1

left = 0

mid = 2

right = 5

### -1

0

### 0

1

### 3

2

### 5

3

### 9

4

### 12

5

`nums[mid] = 3`

Since `3 < 9`, move left to `mid + 1 = 3`.

Iteration 2

left = 3

mid = 4

right = 5

### -1

0

### 0

1

### 3

2

### 5

3

### 9

4

### 12

5

`nums[mid] = 9`

Target found!

Return index 4

### Dry Run Table

| Iteration | Left | Mid | Right | `nums[mid]` | Action         |
| --------- | ---- | --- | ----- | ----------- | -------------- |
| 1         | 0    | 2   | 5     | 3           | Move left to 3 |
| 2         | 3    | 4   | 5     | 9           | Return 4       |

## 8. Tricky Parts You Must Understand

### A. Why `while (left <= right)` and not `left < right`?

Because the search range includes both boundaries.

Consider:

```
nums = [5]
target = 5
```

Initially:

```
left = 0
right = 0
```

The only element still needs to be checked.

With:

```
while (left <= right)
```

The loop executes.

With:

```
while (left < right)
```

It does not execute, and we incorrectly return `-1`.

### B. Why `left = mid + 1`?

If:

```
nums[mid] < target
```

The middle element is too small. Since the array is sorted, the target cannot be at `mid` or anywhere before it.

So we eliminate that entire portion:

```
left = mid + 1;
```

### C. Why `right = mid - 1`?

If:

```
nums[mid] > target
```

The middle element is too large. The target cannot be at `mid` or anywhere after it.

So:

```
right = mid - 1;
```

### D. Why calculate middle like this?

```
int mid = left + (right - left) / 2;
```

Instead of:

```
int mid = (left + right) / 2;
```

When `left + right` becomes larger than Java's maximum integer value, integer overflow can occur. The first formula avoids that particular overflow.

## GitHub-Ready Notes

```
Problem: LeetCode 704 — Binary Search

Pattern:
Binary Search

Approach:
Maintain left and right pointers.
Calculate mid.
Compare nums[mid] with target.
If equal, return mid.
If nums[mid] < target, search right half.
If nums[mid] > target, search left half.
Return -1 if the search range becomes empty.

Time Complexity: O(log n)
Space Complexity: O(1)

Key Insight:
Since the array is sorted, each comparison eliminates
approximately half of the remaining search space.

Important:
Use left <= right because both boundaries are inclusive.
Use mid + 1 and mid - 1 to eliminate the already-checked mid.
```

## Test Your Understanding

Try these before moving to the next binary-search problem.

1\. If nums[mid] is 5 and target is 9, what should we do?

right = mid - 1

left = mid + 1

Return mid

2\. Why do we use left <= right?

To check the final remaining element

To make the algorithm faster

To allow duplicate elements

3\. If nums[mid] is 12 and target is 7, what should we do?

left = mid + 1

right = mid - 1

Return -1 immediately

4\. What is the time complexity of binary search?

O(n)

O(n log n)

O(log n)

Check answers

Next recommended problem: LC 35 — Search Insert Position. It uses almost the exact same binary-search structure, but introduces an important variation: finding the correct insertion position when the target is absent.
