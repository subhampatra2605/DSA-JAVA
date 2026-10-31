### LeetCode 33 — Search in Rotated Sorted Array

Pattern: Modified Binary Search

Goal: Learn how to perform binary search when an originally sorted array has been rotated, and understand how to identify which half is sorted.

This problem is an important step after LC 704 because binary search remains the foundation, but the decision-making becomes slightly more complex.

### 1\. Understand the Problem

You are given an integer array `nums` that was originally sorted in ascending order, but then rotated at some pivot.

You are also given a `target`.

Return the index of the target if it exists. Otherwise, return `-1`.

All values in the array are unique.

### Example 1

```text
Input: nums = [4,5,6,7,0,1,2]
       target = 0

Output: 4
```

Originally sorted:

```text
[0,1,2,4,5,6,7]
```

After rotation:

```text
[4,5,6,7,0,1,2]
```

The target `0` is at index `4`.

### Example 2

```text
Input: nums = [4,5,6,7,0,1,2]
       target = 3

Output: -1
```

The target doesn't exist.

### Example 3

```text
Input: nums = [1]
       target = 0

Output: -1
```

### 2\. Brute-Force Approach — Linear Search

### Idea

Just traverse the array and compare every element with the target.

Even though the array is rotated, linear search works because it doesn't depend on sorting.

### Java Code

```java
class Solution {
    public int search(int[] nums, int target) {

        // Traverse every element.
        for (int i = 0; i < nums.length; i++) {

            // Check whether the current element is the target.
            if (nums[i] == target) {
                return i;
            }
        }

        // Target does not exist.
        return -1;
    }
}
```

### Complexity

* Time: `O(n)`

* Space: `O(1)`

### 3\. Identify the Bottleneck

In LC 704, the array was completely sorted.

For example:

```text
[1,2,3,4,5,6,7]
```

We could directly compare the middle element and eliminate half.

But now consider:

```text
[4,5,6,7,0,1,2]
```

The middle element is `7`.

If our target is `1`, how do we decide whether to search left or right?

We cannot simply use the LC 704 logic without modification.

However, notice something interesting:

Even after rotation, at least one half of the current search range is always sorted.

That observation is the key to solving LC 33.

### 4\. Recognize the Pattern

![Search Element in a Rotated Sorted Array - Tutorial](https://images.openai.com/static-rsc-4/hre-u3_8KsNdenCMsNJXtSqaLSCqwkz_MwmZRuaMebAFBD2kRlCVuxFhAK-NzZrx_X822U0a98R9DMhxD1qIchLb-jMoAu1hnnK4s8UsuHRpoSoH6St-yqEKw4Sqvjnp9y13smyku8CSMR7mkI0XNITFk0PPCvbrJz44zAqvZAevujrv3tamTB093V5ykxxw?purpose=fullsize)

Consider:

```text
[4,5,6,7,0,1,2]
```

The array has two sorted portions:

```text
Left sorted portion:  [4,5,6,7]
Right sorted portion: [0,1,2]
```

When we calculate `mid`, we can determine which half is sorted.

There are two possibilities:

* Left half is sorted.

* Right half is sorted.

Once we identify the sorted half, we check whether the target lies within its value range.

* If yes, search that half.

* If no, search the other half.

This is Modified Binary Search.

### 5\. Derive the Optimal Approach

Initialize the same pointers as LC 704:

```java
int left = 0;
int right = nums.length - 1;
```

Continue while:

```java
while (left <= right)
```

Calculate:

```java
int mid = left + (right - left) / 2;
```

First, check whether we found the target:

```java
if (nums[mid] == target) {
    return mid;
}
```

Now comes the important part.

### Case 1: Left half is sorted

How do we identify it?

```java
if (nums[left] <= nums[mid])
```

Why?

Because the values from `left` to `mid` are in ascending order.

Example:

```text
Index:   0  1  2  3  4  5  6
Array:  [4, 5, 6, 7, 0, 1, 2]
         L        M        R
```

Here:

```text
nums[left] = 4
nums[mid]  = 7

4 <= 7
```

So the left half is sorted.

Now check whether the target lies inside this sorted range:

```java
if (target >= nums[left] && target < nums[mid])
```

Why these boundaries?

Because the sorted left portion ranges from `nums[left]` to `nums[mid]`, but we already checked `mid` and know it isn't the target.

For example:

```text
Sorted left half = [4,5,6,7]
```

If target is `5`:

```text
4 <= 5 < 7
```

Target is inside the left half.

So:

```java
right = mid - 1;
```

Otherwise, target must be in the right half:

```java
left = mid + 1;
```

### Case 2: Right half is sorted

If the left half isn't sorted, the right half must be sorted.

Example:

```text
Index:   0  1  2  3  4  5  6
Array:  [4, 5, 6, 7, 0, 1, 2]
         L        M        R
```

Right sorted half:

```text
[0,1,2]
```

Check whether the target lies inside it:

```java
if (target > nums[mid] && target <= nums[right])
```

For example, target is `1`:

```text
7 < 1 <= 2
```

This is false because `1` is not greater than `7`.

Wait! In this particular example, the middle index is `3`, and the right half is `[0,1,2]`. The correct range check must account for the fact that the sorted half begins at `nums[mid + 1]`, not `nums[mid]`.

The equivalent correct condition is:

```java
if (target > nums[mid] && target <= nums[right])
```

This condition is used only when the right half is sorted in the current search range. In our example, the left half `[4,5,6,7]` is sorted, so we never enter this branch. We enter the right-sorted branch in iterations where the rotation pivot lies in the left half.

For the right-sorted case, if the target is within the range, search right:

```java
left = mid + 1;
```

Otherwise, search left:

```java
right = mid - 1;
```

### 6\. Optimal Java Solution

O(log n) Time

```java
class Solution {
    public int search(int[] nums, int target) {

        int left = 0;
        int right = nums.length - 1;

        while (left <= right) {

            int mid = left + (right - left) / 2;

            // Case 1: Target found.
            if (nums[mid] == target) {
                return mid;
            }

            // Case 2: Left half is sorted.
            if (nums[left] <= nums[mid]) {

                // Check whether target belongs to the sorted left half.
                if (target >= nums[left] && target < nums[mid]) {

                    // Search left.
                    right = mid - 1;

                } else {

                    // Search right.
                    left = mid + 1;
                }

            }
            // Case 3: Right half is sorted.
            else {

                // Check whether target belongs to the sorted right half.
                if (target > nums[mid] && target <= nums[right]) {

                    // Search right.
                    left = mid + 1;

                } else {

                    // Search left.
                    right = mid - 1;
                }
            }
        }

        return -1;
    }
}
```

### Complexity

* Time: `O(log n)` — each iteration eliminates approximately half the search range.

* Space: `O(1)` — only a few variables are used.

### 7\. Detailed Dry Run

Let's take:

```text
nums = [4,5,6,7,0,1,2]
target = 1
```

### Initial state

```text
left = 0
right = 6
```

### Iteration 1

Calculate middle:

```text
mid = 0 + (6 - 0) / 2
mid = 3
```

Array:

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

```text
nums[mid] = 7
target = 1
```

Target not found.

Check whether the left half is sorted:

```text
nums[left] <= nums[mid]
4 <= 7
```

True!

Left half:

```text
[4,5,6,7]
```

Check whether target belongs to this half:

```text
target >= nums[left] && target < nums[mid]

1 >= 4 && 1 < 7
```

False.

Therefore, search the right half:

```java
left = mid + 1;
```

New boundaries:

```text
left = 4
right = 6
```

### Iteration 2

Calculate middle:

```text
mid = 4 + (6 - 4) / 2
mid = 5
```

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

```text
nums[mid] = 1
target = 1
```

They are equal!

Target found — return 5

### Dry Run Summary

| Iteration | Left | Mid | Right | Mid value | Action       |
| --------- | ---- | --- | ----- | --------- | ------------ |
| 1         | 0    | 3   | 6     | 7         | Search right |
| 2         | 4    | 5   | 6     | 1         | Found        |

### 8\. Tricky Parts You Must Understand

### A. Why `nums[left] <= nums[mid]`?

This identifies whether the left half is sorted.

Remember: all elements are unique.

If the leftmost value is smaller than the middle value, the left half must be in ascending order.

### B. Why do we check the target's range?

Because identifying the sorted half alone is not enough.

We need to know whether the target actually belongs to that half.

For a sorted left half:

```java
target >= nums[left] && target < nums[mid]
```

For a sorted right half:

```java
target > nums[mid] && target <= nums[right]
```

These conditions tell us exactly where the target could be.

### C. Why use strict inequality around `mid`?

We already checked:

```java
if (nums[mid] == target)
```

So we don't need to include `mid` again in either search range.

That's why we use:

```java
target < nums[mid]
```

and:

```java
target > nums[mid]
```

### D. Why does at least one half always remain sorted?

The original array was sorted and then rotated once.

The rotation introduces only one break in ascending order.

When we split the current search range at `mid`, that break can only lie in one of the two halves. The other half must be sorted.

This is what makes modified binary search possible.

### GitHub-Ready Notes

```text
Problem: LeetCode 33 — Search in Rotated Sorted Array

Pattern:
Modified Binary Search

Approach:
1. Initialize left = 0 and right = n - 1.
2. Calculate mid.
3. If nums[mid] == target, return mid.
4. Identify which half is sorted.
5. If left half is sorted:
   - Check if target lies between nums[left] and nums[mid].
   - Search left if true, otherwise search right.
6. Otherwise, right half is sorted:
   - Check if target lies between nums[mid] and nums[right].
   - Search right if true, otherwise search left.
7. Return -1 if target is not found.

Time Complexity: O(log n)
Space Complexity: O(1)

Key Insight:
At least one half of the current search range is always sorted.
Identify that half and check whether the target belongs to it.
Eliminate the other half.

Important:
Use nums[left] <= nums[mid] to identify the sorted left half.
Use inclusive range checks for the sorted boundaries.
Exclude mid because it has already been checked.
```

### Test Your Understanding

Try these four questions. Focus on the reasoning, not memorizing the conditions.

1. nums = \[6,7,8,1,2,3,4\], left = 0, mid = 3. Which half is sorted?

Left half

Right half

Neither

2. nums = \[6,7,8,1,2,3,4\], target = 3. At the first iteration, which direction should we search?

Left

Right

3. Why do we check nums\[mid\] == target before identifying the sorted half?

To avoid checking mid again in either half

To sort the array

To reduce the space complexity

4. What is the time complexity of the optimal solution?

O(n)

O(log n)

O(n log n)

Check answers

Next recommended problem: LC 153 — Find Minimum in Rotated Sorted Array.

It uses the same rotated-array observation, but instead of searching for a target, you must find the minimum element.
