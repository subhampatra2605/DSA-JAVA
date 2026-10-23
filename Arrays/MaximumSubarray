Problem: Maximum Subarray
Platform: LeetCode
Problem Number: 53
Difficulty: Medium
Topic: Arrays
Pattern: Kadane's Algorithm
============================================================

PROBLEM:
Given an integer array nums, find the contiguous subarray
with the largest sum and return that sum.

A subarray must contain at least one element.

Example:
Input: nums = [-2,1,-3,4,-1,2,1,-5,4]
Output: 6

Explanation:
The subarray [4,-1,2,1] has the maximum sum:
4 + (-1) + 2 + 1 = 6

------------------------------------------------------------

MY UNDERSTANDING:
We need to find a continuous portion of the array whose
sum is as large as possible.

Negative numbers can reduce the sum, but we cannot skip
elements inside a subarray.

At every element, we must decide:
1. Continue the previous subarray.
2. Start a new subarray from the current element.

------------------------------------------------------------

BRUTE FORCE:
Check every possible subarray and calculate its sum.

Time: O(n²) with running-sum optimization
Space: O(1)

------------------------------------------------------------

OPTIMAL APPROACH:
Use Kadane's Algorithm.

Maintain two variables:

currentSum -> Maximum sum of a subarray ending at the
              current position.

maxSum     -> Maximum subarray sum found so far.

For every element nums[i]:

1. Decide whether to extend the previous subarray or
   start a new one:
   currentSum = max(nums[i], currentSum + nums[i])

2. Update the overall maximum:
   maxSum = max(maxSum, currentSum)

Initialize both variables with nums[0] so that the
algorithm correctly handles arrays containing only
negative numbers.

------------------------------------------------------------

PATTERN / CONCEPT:
Kadane's Algorithm — Greedy + Dynamic Programming

Core decision:

currentSum = max(current element,
                 previous currentSum + current element)

If extending the previous subarray gives a smaller sum
than starting fresh, discard the previous subarray.

------------------------------------------------------------

IMPORTANT INSIGHT:
At each index, we only need the best subarray sum ending
at that index. We do not need to store every subarray.

If the previous running sum hurts the current element,
start a new subarray.

This reduces the time complexity from O(n²) to O(n).

------------------------------------------------------------

COMPLEXITY:
Time: O(n)
Space: O(1)

============================================================
SOLUTION
============================================================
*/

class Maximum_Subarray {

    public static int maxSubArray(int[] nums) {

        int currentSum = nums[0];
        int maxSum = nums[0];

        for (int i = 1; i < nums.length; i++) {

            currentSum = Math.max(
                nums[i],
                currentSum + nums[i]
            );

            maxSum = Math.max(maxSum, currentSum);
        }

        return maxSum;
    }

    public static void main(String[] args) {

        int[] nums = {-2, 1, -3, 4, -1, 2, 1, -5, 4};

        System.out.println(maxSubArray(nums)); // Output: 6
    }
}

### Dry run — understand the order of thinking

For `nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]`:

| Current element | currentSum | maxSum |
| --------------- | ---------- | ------ |
| -2              | -2         | -2     |
| 1               | 1          | 1      |
| -3              | -2         | 1      |
| 4               | 4          | 4      |
| -1              | 3          | 4      |
| 2               | 5          | 5      |
| 1               | 6          | 6      |
| -5              | 1          | 6      |
| 4               | 5          | 6      |

Answer: `6`, from the subarray `[4, -1, 2, 1]`.

### Remember this pattern

* `currentSum` answers: What is the best subarray sum ending here?

* `maxSum` answers: What is the best subarray sum anywhere so far?

* Use `Math.max(nums[i], currentSum + nums[i])` to decide whether to restart or extend.

Important edge case: For `[-5, -2, -8]`, the answer is `-2`, not `0`. That is why we initialize with `nums[0]`, rather than zero.
