LeetCode 525: Contiguous Array

Step 1: Understand the Problem

You're given a binary array containing only 0s and 1s.

Find the maximum length of a contiguous subarray containing an equal number of zeros and ones.

Example:

nums = [0, 1, 0, 1, 1, 0]

The entire array contains:

- Three zeros.
- Three ones.

Therefore, the longest balanced subarray is the entire array.

Output:

6

What does contiguous mean?

The elements must be next to each other without skipping any elements.

For example:

[0, 1, 0, 1]  → Contiguous subarray
[0, 0, 1, 1]  → Not necessarily a subarray of the original array

Our objective is to return the length, not the subarray itself.


---

Step 2: Understand the Example

Consider:

nums = [0, 1, 0, 1, 1, 0]

Let's examine some possible subarrays.

Subarray	Zeros	Ones	Length	Balanced?

[0]	1	0	1	No
[0, 1]	1	1	2	Yes
[0, 1, 0]	2	1	3	No
[0, 1, 0, 1]	2	2	4	Yes
[0, 1, 0, 1, 1]	2	3	5	No
[0, 1, 0, 1, 1, 0]	3	3	6	Yes


The longest balanced subarray has length 6.

The challenge is finding this answer efficiently without repeatedly counting zeros and ones.


---

Step 3: Develop the Brute Force Approach

Let's first solve the problem without thinking about optimization.

3.1 How should we approach it?

We need to examine every possible subarray.

To generate every subarray, we need two loops:

- The outer loop chooses the starting index i.
- The inner loop chooses the ending index j.

For every starting index, we expand the subarray one element at a time.

While expanding, we count zeros and ones.

Whenever the counts become equal, we calculate the length and update our answer.

3.2 Why do we check only nums[j]?

Remember your previous question!

Suppose:

nums = [0, 1, 0, 1]

When i = 0, the inner loop expands the subarray:

j = 0 → [0]
j = 1 → [0, 1]
j = 2 → [0, 1, 0]
j = 3 → [0, 1, 0, 1]

We count only nums[j] because each iteration introduces exactly one new element.

The previous elements have already been counted.

3.3 Brute Force Java Code — With Detailed Comments

class Solution {
    public int findMaxLength(int[] nums) {

        // Stores the maximum length of a balanced subarray found so far.
        int maxLength = 0;

        // STEP 1: Choose every possible starting index.
        for (int i = 0; i < nums.length; i++) {

            // Reset the counts for each new starting index.
            int zeros = 0;
            int ones = 0;

            // STEP 2: Expand the subarray from starting index i.
            for (int j = i; j < nums.length; j++) {

                // Count only the newly added element.
                // Earlier elements between i and j - 1
                // have already been counted in previous iterations.
                if (nums[j] == 0) {
                    zeros++;
                } else {
                    ones++;
                }

                // STEP 3: Check whether this subarray is balanced.
                if (zeros == ones) {

                    // The subarray includes both endpoints i and j,
                    // so its length is j - i + 1.
                    int length = j - i + 1;

                    // Keep the longest balanced subarray found so far.
                    maxLength = Math.max(maxLength, length);
                }
            }
        }

        // Return the longest balanced subarray length.
        return maxLength;
    }
}

3.4 Dry Run

Input:

nums = [0, 1, 0, 1]

Initially:

maxLength = 0

For i = 0:

j	Element	Zeros	Ones	Balanced?	maxLength

0	0	1	0	No	0
1	1	1	1	Yes	2
2	0	2	1	No	2
3	1	2	2	Yes	4


The maximum length is now 4.

No longer balanced subarray exists, so the final answer is 4.

3.5 Complexity

Time complexity: O(n²)

There are two nested loops. In the worst case, we examine approximately n² / 2 subarrays.

Auxiliary space complexity: O(1)

We only maintain a few variables: maxLength, zeros, ones, i, and j.


---

Step 4: Identify the Bottleneck

Now ask yourself:

What is inefficient about the brute-force approach?

The biggest issue is that we examine every possible subarray.

Consider:

[0, 1, 0, 1, 1, 0]

We repeatedly count zeros and ones for different subarrays.

But there is a mathematical property that allows us to identify balanced subarrays without counting both values separately.

Let's derive it.

The key observation

Imagine transforming the array using this rule:

0 → -1
1 → +1

For example:

Original:     [0, 1, 0, 1]
Transformed:  [-1, 1, -1, 1]

Calculate the sum:

-1 + 1 - 1 + 1 = 0

Why?

Because every zero contributes -1, and every one contributes +1.

When the numbers of zeros and ones are equal, their contributions cancel out.

For example:

[0, 1, 1, 0]

Transformed:
[-1, 1, 1, -1]

Sum = -1 + 1 + 1 - 1
    = 0

Therefore:

A subarray has equal numbers of zeros and ones if and only if its transformed sum is zero.

We have now converted the original problem into finding the longest subarray with a sum of zero.

This is the breakthrough that leads to the optimal solution.


---

Step 5: Derive the Optimal Approach

We will use two concepts:

1. Prefix balance.
2. A HashMap.

Let's understand each before writing the code.

5.1 Concept 1: Prefix Balance

Instead of creating a separate transformed array, we can calculate the balance as we traverse the original array.

Use these rules:

If nums[i] == 0:
    balance--

If nums[i] == 1:
    balance++

Let's trace:

nums = [0, 1, 0, 1, 1, 0]

Initially:

balance = 0

Now process each element.

Index	Number	Operation	Balance

0	0	balance--	-1
1	1	balance++	0
2	0	balance--	-1
3	1	balance++	0
4	1	balance++	1
5	0	balance--	0


The balance values are:

Index:    0    1    2    3    4    5
Number:   0    1    0    1    1    0
Balance: -1    0   -1    0    1    0

Notice that the balance becomes 0 at indices 1, 3, and 5.

At index 5, the balance is zero because the entire array contains equal numbers of zeros and ones.

But there is something even more useful here.

We don't need the balance to be zero. We only need to find the same balance at two different positions.

Let's understand why.

5.2 Concept 2: Why Does a Repeated Balance Matter?

Consider:

nums = [1, 1, 0, 0, 1]

Let's calculate the balance.

Position	Number	Balance

Before the array	—	0
0	1	1
1	1	2
2	0	1
3	0	0
4	1	1


Notice:

Index 0 → Balance = 1
Index 2 → Balance = 1
Index 4 → Balance = 1

The balance 1 occurs at three positions.

Let's focus on indices 0 and 2.

At index 0, the balance is 1.

At index 2, the balance is also 1.

What happened between those positions?

Index 0: [1]
Index 1:  1
Index 2:  0

The balance increased by 1 when we processed the 1 at index 1, then decreased by 1 when we processed the 0 at index 2.

Those two changes cancelled out.

Therefore, the subarray between the repeated balances is:

nums[1..2] = [1, 0]

It contains one zero and one one!

Its length is:

2 - 0 = 2

Now compare indices 0 and 4.

The balance is 1 at both positions, so the elements between them also have a net balance change of zero.

The subarray is:

nums[1..4] = [1, 0, 0, 1]

It contains two ones and two zeros.

Its length is:

4 - 0 = 4

The rule: If the same balance occurs at indices j and i, the subarray from j + 1 through i has equal numbers of zeros and ones. Its length is i - j.

This is the central idea of the optimal solution.

5.3 Concept 3: Why Do We Need a HashMap?

We need to remember the first index where each balance appeared.

We can use:

Map<Integer, Integer> firstIndex = new HashMap<>();

Here:

- The key is the balance.
- The value is the earliest index where that balance appeared.

For example:

Balance → Earliest index

1 → 0
2 → 1
0 → 3

When a balance appears again, we calculate:

int length = i - firstIndex.get(balance);

Then update:

maxLength = Math.max(maxLength, length);

Why store the earliest index?

Suppose balance 1 appears at indices 0, 2, and 4.

Using the earliest index:

4 - 0 = 4

If we overwrite the earliest index with 2, we get:

4 - 2 = 2

We would miss the longer subarray.

Therefore, we store a balance's index only the first time it appears.

One important initialization

We begin with:

firstIndex.put(0, -1);

This represents a balance of zero immediately before the array starts.

Why is this necessary?

Consider:

nums = [0, 1, 0, 1]

At index 3, the balance becomes zero.

We want to calculate the length of the entire array:

length = 3 - (-1)
       = 4

Without initializing balance zero at index -1, we could miss balanced subarrays that start at index 0.


---

Step 6: Build the Optimal Algorithm Step by Step

Now that we understand the concepts, let's convert them into an algorithm.

Step 1: Initialize the variables

int balance = 0;
int maxLength = 0;

Map<Integer, Integer> firstIndex = new HashMap<>();

firstIndex.put(0, -1);

- balance tracks the difference between ones and zeros.
- maxLength stores the longest balanced subarray found.
- firstIndex remembers the earliest position of each balance.
- firstIndex.put(0, -1) handles subarrays that begin at index 0.

Step 2: Traverse the array

for (int i = 0; i < nums.length; i++) {

We process each element exactly once.

Step 3: Update the balance

if (nums[i] == 0) {
    balance--;
} else {
    balance++;
}

A zero decreases the balance; a one increases it.

Step 4: Check whether this balance has appeared before

if (firstIndex.containsKey(balance)) {

If it has, we have found a balanced subarray ending at the current index.

Step 5: Calculate the length

int length = i - firstIndex.get(balance);

We subtract the earliest index of the same balance from the current index.

Step 6: Update the maximum length

maxLength = Math.max(maxLength, length);

We retain whichever is larger: the previous maximum or the newly discovered length.

Step 7: Otherwise, record the balance

else {
    firstIndex.put(balance, i);
}

We record the current index only if this balance has never appeared before.

Important: Do not overwrite an existing index. The earliest occurrence gives us the longest possible subarray for that balance.


---

Step 7: Optimal Java Solution With Detailed Comments

Here is the complete solution with comments explaining both what each statement does and why it is needed.

import java.util.HashMap;
import java.util.Map;

class Solution {
    public int findMaxLength(int[] nums) {

        // Tracks the difference between the number of ones and zeros.
        // We treat 1 as +1 and 0 as -1.
        int balance = 0;

        // Stores the length of the longest balanced subarray found so far.
        int maxLength = 0;

        // Maps each balance to the earliest index where it appeared.
        // Repeated balances help us identify balanced subarrays.
        Map<Integer, Integer> firstIndex = new HashMap<>();

        // A balance of 0 exists before the array starts, at index -1.
        // This allows us to count balanced subarrays beginning at index 0.
        firstIndex.put(0, -1);

        // Traverse the array once, updating the balance at each index.
        for (int i = 0; i < nums.length; i++) {

            // A zero decreases the balance because it contributes -1.
            // A one increases the balance because it contributes +1.
            if (nums[i] == 0) {
                balance--;
            } else {
                balance++;
            }

            // If this balance appeared earlier, the elements between
            // the earlier index and i have a net balance change of zero.
            // Therefore, that subarray has equal numbers of zeros and ones.
            if (firstIndex.containsKey(balance)) {

                // Calculate the subarray length using the earliest
                // occurrence of this balance and the current index.
                int length = i - firstIndex.get(balance);

                // Keep the maximum length found across all balanced subarrays.
                maxLength = Math.max(maxLength, length);

            } else {

                // This is the first time we have encountered this balance.
                // Save its index so future occurrences can form longer subarrays.
                // Do not overwrite this index later: the earliest index
                // gives us the maximum possible length for this balance.
                firstIndex.put(balance, i);
            }
        }

        // Return the longest contiguous subarray with equal zeros and ones.
        return maxLength;
    }
}

Dry Run of the Optimal Approach

Input:

nums = [0, 1, 0, 1]

Initially:

balance = 0
maxLength = 0
firstIndex = {0 = -1}

Let's process the array.

i	nums[i]	Balance after update	Previously seen?	Length found	maxLength

0	0	-1	No	—	0
1	1	0	Yes, at -1	1 - (-1) = 2	2
2	0	-1	Yes, at 0	2 - 0 = 2	2
3	1	0	Yes, at -1	3 - (-1) = 4	4


Let's examine the last iteration.

At index 3:

nums[3] = 1
balance = 0

We already encountered balance 0 at index -1.

Therefore:

length = 3 - (-1)
       = 4

The entire array is balanced, so:

maxLength = 4

Final output: 4

Why is the optimal solution faster?

The brute-force approach checks every possible subarray, requiring O(n²) time.

The optimal approach:

1. Processes each element once.
2. Updates the balance.
3. Performs an average O(1) HashMap lookup.
4. Updates the answer when it finds a repeated balance.

Therefore:

Complexity	Brute Force	Optimal

Time	O(n²)	O(n) average
Auxiliary space	O(1)	O(n)



---

Step 8: Edge Cases and Independent Practice

Before moving on, understand how the algorithm handles these cases.

Input	Output	Reason

[0, 1]	2	One zero and one one
[0, 1, 0, 1]	4	The entire array is balanced
[0, 0, 0]	0	No balanced non-empty subarray
[1, 1, 1]	0	No balanced non-empty subarray
[0, 1, 1, 0]	4	The entire array is balanced


Your independent practice

Without looking at the code, try answering these three questions:

1. Balance calculation

For this array:

nums = [1, 0, 0, 1, 1]

Write the balance after processing every element.

2. HashMap logic

If the balance is 2 at index 2 and appears again at index 6, what is the length of the balanced subarray?

3. Earliest index

Suppose balance -1 first appears at index 0, appears again at index 3, and appears again at index 5.

Why must we keep index 0 rather than update the stored index every time?

Once you can answer these questions, you'll be able to reconstruct the key logic of LC 525 rather than memorize the solution.