LeetCode 53 — Maximum Subarray

Let's solve Maximum Subarray using your 8-step problem-solving framework, with special attention to how to identify the bottleneck, derive Kadane's algorithm, and determine the correct order of the final Java code.

The goal is to develop a process you can reproduce independently during your Infosys SP/DSE coding assessment.


---

Step 1: Understand the Problem

Problem statement

Given an integer array nums, find the contiguous, non-empty subarray with the largest possible sum and return that sum.

Important rules:

- The subarray must be contiguous.
- It must contain at least one element.
- Elements can be negative, positive, or zero.
- Return the maximum sum, not the subarray itself.

Example 1

Input:  nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
Output: 6

The maximum-sum subarray is:

[4, -1, 2, 1]

Its sum is:

4 + (-1) + 2 + 1 = 6

Example 2

Input:  nums = [-3, -2, -5]
Output: -2

Why not return 0?

Because the subarray must be non-empty. The best choice is the single element -2.

Your first checkpoint: We need to find the largest sum among all contiguous subarrays, including subarrays containing just one element.


---

Step 2: Solve a Small Example Manually

Consider:

nums = [2, -4, 3, 5, -1]

Let's examine some contiguous subarrays.

Starting at index 0:

[2]             → sum = 2
[2, -4]         → sum = -2
[2, -4, 3]      → sum = 1
[2, -4, 3, 5]   → sum = 6
[2, -4, 3, 5, -1] → sum = 5

Starting at index 2:

[3]          → sum = 3
[3, 5]       → sum = 8
[3, 5, -1]   → sum = 7

The maximum sum is:

3 + 5 = 8

Therefore:

Output: 8

Now ask yourself:

When I reach a new element, do I always want to include the elements before it?

No.

For example, if the running sum is -6 and the next element is 4, extending the previous subarray gives:

-6 + 4 = -2

Starting a new subarray at 4 gives:

4

Starting fresh is better.

This observation will lead us to the optimized algorithm.


---

Step 3: Derive the Brute-Force Approach

When you don't immediately see the optimal solution, first derive the simplest correct solution.

Brute-force reasoning

A contiguous subarray is determined by two indices:

- i: the starting index.
- j: the ending index, where j >= i.

Therefore, we can:

1. Choose a starting index i.
2. Initialize a running sum to 0.
3. Extend the subarray by moving j from i to the end.
4. Add each new element to the running sum.
5. Update the maximum sum whenever the running sum is larger.
6. Repeat for every starting index.

Notice that we're reusing a running sum rather than recalculating each subarray's sum from scratch.

Java implementation

class Solution {
    public int maxSubArray(int[] nums) {
        int maxSum = nums[0];

        for (int i = 0; i < nums.length; i++) {
            int currentSum = 0;

            for (int j = i; j < nums.length; j++) {
                currentSum += nums[j];
                maxSum = Math.max(maxSum, currentSum);
            }
        }

        return maxSum;
    }
}

This is a correct brute-force solution with O(n²) time complexity and O(1) auxiliary space.

Why initialize maxSum to nums[0] instead of 0?

Because an array containing only negative numbers must return its largest negative element, not zero.


---

Step 4: Dry-Run the Brute Force

Consider:

nums = [2, -4, 3, 5, -1]

Initially:

maxSum = 2

For i = 0, we extend the subarray starting at index 0.

Starting index i	Ending index j	Subarray	currentSum	maxSum

0	0	[2]	2	2
0	1	[2, -4]	-2	2
0	2	[2, -4, 3]	1	2
0	3	[2, -4, 3, 5]	6	6
0	4	[2, -4, 3, 5, -1]	5	6


Now move to i = 1 and reset currentSum to 0.

Starting index i	Ending index j	Subarray	currentSum	maxSum

1	1	[-4]	-4	6
1	2	[-4, 3]	-1	6
1	3	[-4, 3, 5]	4	6
1	4	[-4, 3, 5, -1]	3	6


The same process continues for the remaining starting indices.

Eventually, we discover:

[3, 5] → sum = 8

The final answer is 8.

What have we learned?

The brute-force solution explores many subarrays that begin at different positions.

But when we process a new element, we don't actually need to remember every possible subarray. We need to determine which previous information is useful for constructing the best subarray ending at the current index.

That is the key to optimization.


---

Step 5: Identify the Bottleneck

This is the most important reasoning step.

Don't jump straight from O(n²) to Kadane's algorithm. Understand why we can eliminate the nested loop.

Ask yourself these three questions.

Question 1: What work am I repeating?

For every starting index i, we explore all possible ending indices j.

For an array of length n, the number of possible contiguous subarrays is:

\[
\frac{n(n+1)}{2}
\]

Therefore, the brute-force approach takes O(n²) time.

We need to process all possible subarrays conceptually, but we don't necessarily need to enumerate them individually.

Question 2: What information do I actually need at each index?

Suppose we're processing:

nums = [2, -4, 3, 5, -1]

When we reach the element 3, what is the best subarray ending exactly at this element?

The possible choices are:

Choice A: Start a new subarray at 3.

[3]
Sum = 3

Choice B: Extend the best subarray ending at the previous element.

The best subarray ending at -4 is:

[2, -4]
Sum = -2

Extend it with 3:

[2, -4, 3]
Sum = 1

Compare:

Start fresh: 3
Extend:      1

Starting fresh gives the larger sum.

Now consider the element 5.

The best subarray ending at 3 is [3], with sum 3.

We have two choices:

Start fresh: 5
Extend:      3 + 5 = 8

Extending is better.

Therefore, the best subarray ending at the current element either starts at the current element or extends the best subarray ending at the previous element.

This is the crucial observation.

Question 3: Can I represent all that information with one variable?

Yes.

We don't need to store every subarray.

We only need to track:

1. currentSum: the largest sum of a subarray that must end at the current index.
2. maxSum: the largest subarray sum encountered anywhere so far.

These two variables allow us to process the array in a single pass.


---

Step 6: Derive Kadane's Algorithm

Let's turn the observation into a precise algorithm.

Part A: Understand currentSum

At each index, there are two possibilities.

Option 1: Start a new subarray.

currentSum = nums[i];

This discards the previous subarray and begins at the current element.

Option 2: Extend the previous subarray.

currentSum = currentSum + nums[i];

This includes the current element in the best subarray ending at the previous index.

We choose whichever produces the larger sum:

currentSum = Math.max(nums[i], currentSum + nums[i]);

Mathematically:

\[
\text{currentSum}_i =
\max(\text{nums}[i],\ \text{currentSum}_{i-1}+\text{nums}[i])
\]

Why does this work?

Suppose the best subarray ending at the previous index has a sum of -5.

The next element is 4.

Then:

Extend previous subarray: -5 + 4 = -1
Start fresh:               4

Starting fresh is better because the negative previous sum only reduces the new element's sum.

Now suppose the previous sum is 6 and the next element is -2.

Extend:      6 + (-2) = 4
Start fresh: -2

Extending is better.

The rule automatically handles both situations.

Part B: Understand maxSum

currentSum only represents the best subarray ending at the current index.

But the overall best subarray might end anywhere.

Therefore, after calculating the new currentSum, update the best answer:

maxSum = Math.max(maxSum, currentSum);

The order matters:

1. Calculate the best subarray ending at the current index.
2. Compare it with the best subarray sum found anywhere so far.

If we update maxSum first, we'd compare it against the previous currentSum, potentially missing the best subarray ending at the current index.

Part C: Initialize both variables correctly

Use:

int currentSum = nums[0];
int maxSum = nums[0];

Why not initialize both to 0?

Consider:

nums = [-5, -2, -8]

The correct answer is -2.

If maxSum starts at 0 and only updates when a larger sum is found, the algorithm may incorrectly return 0.

Starting both variables at nums[0] ensures that at least one element is included.

Since index 0 has already been processed, begin the loop at index 1.

The complete algorithm in plain English

1. Initialize the current subarray sum and maximum sum to the first element.
2. Visit each remaining element from left to right.
3. Decide whether to start fresh or extend the previous subarray.
4. Update the current subarray sum.
5. Update the global maximum.
6. Return the global maximum after processing all elements.

This is Kadane's algorithm, which solves Maximum Subarray in O(n) time.


---

Step 7: Dry-Run the Optimized Algorithm

Use:

nums = [2, -4, 3, 5, -1]

Initialization:

currentSum = 2
maxSum = 2

We start the loop at index 1.

At every index, calculate:

extend = previous currentSum + nums[i]
start  = nums[i]

currentSum = max(start, extend)
maxSum = max(maxSum, currentSum)

Detailed trace

Index	nums[i]	Extend previous subarray	Start fresh	New currentSum	New maxSum

0	2	—	—	2	2
1	-4	2 + (-4) = -2	-4	-2	2
2	3	-2 + 3 = 1	3	3	3
3	5	3 + 5 = 8	5	8	8
4	-1	8 + (-1) = 7	-1	7	8


Final result:

maxSum = 8

The maximum-sum subarray is [3, 5].

Test the all-negative case

nums = [-3, -2, -5]

Initialization:

currentSum = -3
maxSum = -3

At index 1:

Extend: -3 + (-2) = -5
Start:  -2

currentSum = max(-5, -2) = -2
maxSum = max(-3, -2) = -2

At index 2:

Extend: -2 + (-5) = -7
Start:  -5

currentSum = max(-7, -5) = -5
maxSum = max(-2, -5) = -2

Final answer:

-2

This confirms that our initialization correctly handles negative numbers.


---

Step 8: Write the Final Java Code in the Correct Order

Now translate the algorithm into Java.

The order follows the dependencies between the variables.

1. Define the method

class Solution {
    public int maxSubArray(int[] nums) {

The method receives an integer array and returns the maximum subarray sum.

2. Initialize the variables

int currentSum = nums[0];
        int maxSum = nums[0];

Why first?

The algorithm needs an existing subarray before it can decide whether to extend or replace it. The first element provides that starting point.

3. Start the loop at index 1

for (int i = 1; i < nums.length; i++) {

Why not start at 0?

Index 0 has already initialized both variables. Starting at 1 prevents us from processing the first element twice.

4. Calculate the best subarray ending at the current index

currentSum = Math.max(nums[i], currentSum + nums[i]);

Why this statement comes first inside the loop:

We need the updated currentSum before we can compare it against the global maximum.

5. Update the global maximum

maxSum = Math.max(maxSum, currentSum);

Why after updating currentSum?

Because the global maximum must include the best subarray ending at the current index, which we've just calculated.

6. Return after the loop

}
        return maxSum;
    }
}

Why outside the loop?

We must process every element before we know the global maximum.

Complete optimized Java solution

class Solution {
    public int maxSubArray(int[] nums) {
        int currentSum = nums[0];
        int maxSum = nums[0];

        for (int i = 1; i < nums.length; i++) {
            currentSum = Math.max(nums[i], currentSum + nums[i]);
            maxSum = Math.max(maxSum, currentSum);
        }

        return maxSum;
    }
}

Complexity

- Time: O(n) — each element is processed once.
- Auxiliary space: O(1) — only two variables are maintained.


---

Your Revision Card: Maximum Subarray

When revising this problem, don't begin by memorizing the code. Reconstruct the reasoning in this order.

Step	Question to ask yourself	Answer

1	What am I finding?	The largest sum of a non-empty contiguous subarray
2	How can I solve it directly?	Enumerate starting and ending indices
3	What is the brute-force complexity?	O(n²) with a running sum
4	What is the bottleneck?	Exploring every possible subarray
5	What information is sufficient?	The best subarray sum ending at the previous index
6	What are my choices at each element?	Start fresh or extend
7	How do I calculate the current result?	currentSum = Math.max(nums[i], currentSum + nums[i])
8	How do I maintain the overall answer?	maxSum = Math.max(maxSum, currentSum)


The most important distinction

currentSum and maxSum are not interchangeable.

- currentSum answers: What is the maximum sum of a subarray that must end at this index?
- maxSum answers: What is the maximum sum found anywhere in the array so far?

Remembering these two meanings makes the code order much easier to reconstruct.

Your practice task

Close the solution and write it from memory. Before coding, say these two sentences aloud:

1. Current decision: Should I start a new subarray at this element or extend the previous best subarray ending at the preceding index?
2. Global decision: Is the new subarray sum better than the best sum I've found anywhere so far?

If you can explain why those two decisions are sufficient, you understand the core of Kadane's algorithm.