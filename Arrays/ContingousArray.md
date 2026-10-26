LeetCode 525 — Contiguous Array

Step 1: Understand the Problem

You are given a binary array containing only 0 and 1.

Your task is to find the maximum length of a contiguous subarray containing an equal number of zeros and ones.

Example:

Input:  [0, 1, 0, 1, 1, 0, 0]
Output: 6

Explanation:

The subarray [0, 1, 0, 1, 1, 0] contains three zeros and three ones.

Its length is 6, so the answer is 6.

Important points:

- The subarray must be contiguous.
- It must contain an equal number of 0s and 1s.
- We need the maximum length, not the number of valid subarrays.


---

Step 2: Think of the Brute-Force Approach

Before optimizing, think about how you would solve this problem without any special technique.

Logic

1. Start from every possible index.
2. Expand the subarray one element at a time.
3. Keep counting zeros and ones.
4. Whenever the counts are equal, calculate the subarray length.
5. Update the maximum length.

Brute-Force Code — Java

class Solution {
    public int findMaxLength(int[] nums) {
        int maxLength = 0;

        // Try every possible starting index
        for (int start = 0; start < nums.length; start++) {
            int zeros = 0;
            int ones = 0;

            // Expand the subarray from the current starting index
            for (int end = start; end < nums.length; end++) {

                // Count whether the current element is 0 or 1
                if (nums[end] == 0) {
                    zeros++;
                } else {
                    ones++;
                }

                // If zeros and ones are equal, this is a valid subarray
                if (zeros == ones) {
                    int length = end - start + 1;

                    // Keep track of the longest valid subarray
                    maxLength = Math.max(maxLength, length);
                }
            }
        }

        // Return the maximum length found
        return maxLength;
    }
}

Complexity

- Time: O(n²) — we examine all possible subarrays.
- Space: O(1) — we use only a few variables.

Identify the Bottleneck

The nested loops examine every possible subarray. We need to avoid checking all of them individually.

The key question is: Can we recognize a balanced subarray without recounting all its zeros and ones?

Yes! We can use a prefix sum.


---

Step 3: Find the Pattern

The important observation is that a subarray has an equal number of zeros and ones when their difference is zero.

Consider this array:

Index:  0  1  2  3  4  5
Array: [0, 1, 0, 1, 1, 0]

Let's assign values to the elements:

- Every 0 becomes -1.
- Every 1 becomes +1.

The array effectively becomes:

[-1, +1, -1, +1, +1, -1]

Why does this work?

Suppose a subarray contains:

[0, 1, 0, 1]

After transformation:

[-1, +1, -1, +1]

Its sum is:

-1 + 1 - 1 + 1 = 0

The number of zeros equals the number of ones, so their transformed values cancel each other out.

Therefore:

A subarray has equal numbers of zeros and ones if and only if its transformed sum is zero.

We don't actually need to modify the array. We can maintain a running balance:

- If nums[i] == 0, decrease the balance by 1.
- If nums[i] == 1, increase the balance by 1.

if (nums[i] == 0) {
    balance--;
} else {
    balance++;
}

Now we can solve the problem using a prefix balance and a HashMap.


---

Step 4: Derive the Optimal Approach

Understand the Prefix Balance

The balance represents the sum of all transformed elements from index 0 to the current index.

Consider:

Array:   [0, 1, 0, 1]
Balance: -1, 0, -1, 0

Notice that the balance becomes 0 at index 1 and again at index 3.

Why is this important?

The prefix ending at index 1 has a balance of 0.

The prefix ending at index 3 also has a balance of 0.

Because the balances are equal, the elements between those two prefix boundaries must have a sum of zero.

That means the subarray between the boundaries contains equal numbers of zeros and ones.

The Key Formula

If the same balance appears at indices previousIndex and currentIndex, then:

Subarray length = currentIndex - previousIndex

For example:

Index:    0   1   2   3
Array:   [0,  1,  0,  1]
Balance: -1,  0, -1,  0

At index 1, the balance is 0.

At index 3, the balance is also 0.

Therefore:

Length = 3 - 1 = 2

The subarray [0, 1] between those indices is balanced.

But there's an important detail: we should store the earliest index at which each balance appears.

Why?

Because the earliest index gives us the longest possible subarray when the same balance appears again.

What Should the HashMap Store?

We use:

Map<Integer, Integer> firstIndex = new HashMap<>();

The mapping is:

Balance → Earliest index where it appeared

For example:

Balance   Earliest Index
   -1           0
    0           1
    1           4

If the balance 0 appears again at index 5, we calculate:

Length = 5 - 1 = 4

We do not replace the original index with 5, because that would lose the opportunity to calculate the longer length later.

Why Initialize firstIndex.put(0, -1)?

We begin with:

firstIndex.put(0, -1);

This represents a balance of zero before the array begins.

Consider:

Array: [0, 1]

At index 0:

Balance = -1

At index 1:

Balance = 0

Since balance 0 was previously recorded at index -1, we calculate:

Length = 1 - (-1) = 2

This allows us to count balanced subarrays that start at index 0.

Without this initialization, we would miss such subarrays.


---

Step 5: Write the Optimal Algorithm

Now that we understand the logic, the algorithm becomes straightforward.

1. Initialize balance = 0.
2. Initialize maxLength = 0.
3. Create a HashMap to store each balance's earliest index.
4. Insert the initial balance 0 at index -1.
5. Traverse the array from left to right.
6. Update the balance based on the current element.
7. If the balance already exists in the map:
   - Calculate the length using the current index and the earliest recorded index.
   - Update maxLength.
8. Otherwise, store the current balance and index.
9. Return maxLength.

Optimal Code — Java

import java.util.HashMap;
import java.util.Map;

class Solution {
    public int findMaxLength(int[] nums) {
        int balance = 0;
        int maxLength = 0;

        // Store the earliest index where each balance appears
        Map<Integer, Integer> firstIndex = new HashMap<>();

        // Initial balance before the array starts
        firstIndex.put(0, -1);

        // Traverse the array and maintain the running balance
        for (int i = 0; i < nums.length; i++) {

            // Treat 0 as -1 and 1 as +1
            if (nums[i] == 0) {
                balance--;
            } else {
                balance++;
            }

            // If this balance appeared before, the subarray between
            // the previous index and the current index is balanced
            if (firstIndex.containsKey(balance)) {
                int length = i - firstIndex.get(balance);

                // Update the maximum balanced subarray length
                maxLength = Math.max(maxLength, length);
            } else {
                // Store only the earliest occurrence of this balance
                firstIndex.put(balance, i);
            }
        }

        // Return the longest balanced subarray length
        return maxLength;
    }
}

Complexity

- Time: O(n) average — we traverse the array once, with average constant-time HashMap operations.
- Space: O(n) — in the worst case, we store up to n + 1 distinct balances.


---

Step 6: Dry Run the Optimal Code

Let's trace the code using:

nums = [0, 1, 0, 1]

Initially:

balance = 0
maxLength = 0

firstIndex = {
    0: -1
}

We use the initial mapping 0: -1 to handle subarrays starting at index 0.

Index i	nums[i]	Updated balance	Balance already exists?	Length calculated	maxLength

0	0	-1	No	—	0
1	1	0	Yes, at -1	1 - (-1) = 2	2
2	0	-1	Yes, at 0	2 - 0 = 2	2
3	1	0	Yes, at -1	3 - (-1) = 4	4


Let's understand each iteration.

Iteration 1: Index 0

nums[0] = 0
balance = -1

Balance -1 is not in the map, so store:

firstIndex = {
     0: -1,
    -1:  0
}

No balanced subarray has been found yet.

Iteration 2: Index 1

nums[1] = 1
balance = 0

Balance 0 already exists at index -1.

Calculate:

length = 1 - (-1) = 2

The subarray is:

[0, 1]

It contains one zero and one one.

Update:

maxLength = 2

Iteration 3: Index 2

nums[2] = 0
balance = -1

Balance -1 already exists at index 0.

Calculate:

length = 2 - 0 = 2

The subarray is:

[1, 0]

The maximum length remains 2.

Notice: We do not overwrite the earliest index of balance -1.

Iteration 4: Index 3

nums[3] = 1
balance = 0

Balance 0 already exists at index -1.

Calculate:

length = 3 - (-1) = 4

The subarray is:

[0, 1, 0, 1]

It contains two zeros and two ones.

Update:

maxLength = 4

Final output:

4


---

Step 7: Understand the Tricky Parts

These are the most important details to remember when implementing this solution independently.

1. Why do we change 0 to -1?

Because we want zeros and ones to cancel each other out.

0 → -1
1 → +1

When a subarray contains equal numbers of zeros and ones, its transformed sum is zero.

2. Why does a repeated balance indicate a balanced subarray?

Suppose the balance is 2 at index i and becomes 2 again at index j.

The sum of the elements between those prefix boundaries must be zero:

balance[j] - balance[i] = 2 - 2 = 0

Therefore, the subarray between them contains equal numbers of zeros and ones.

Its length is:

j - i

3. Why store the earliest index?

We want the longest balanced subarray.

For a fixed current index, subtracting the earliest previous index gives the largest possible length.

Therefore, once a balance has been stored, we must not overwrite its first occurrence.

4. Why use firstIndex.put(0, -1)?

It lets us calculate the length of a balanced subarray that begins at index 0.

For example:

nums = [0, 1]

The balance returns to zero at index 1.

Because the initial balance was recorded at index -1:

length = 1 - (-1) = 2

5. Why is this different from LeetCode 560?

Both problems use prefix sums and HashMaps, but the map stores different information.

Feature	LC 560 — Subarray Sum Equals K	LC 525 — Contiguous Array

Goal	Count subarrays whose sum equals k	Find the longest balanced subarray
Prefix representation	Actual running sum	Running balance (0 → -1, 1 → +1)
HashMap stores	Prefix sum → frequency	Balance → earliest index
When a match occurs	Add the frequency to the count	Calculate the length and update the maximum
Initial mapping	0 → 1	0 → -1
Result	Number of valid subarrays	Maximum length


Remember: LC 560 counts how many valid subarrays exist, while LC 525 finds the longest valid subarray.


---

Step 8: Complexity Analysis and Final Revision

Brute Force vs. Optimal

Approach	Time Complexity	Auxiliary Space	Main Idea

Brute force	O(n²)	O(1)	Try every starting index and expand the subarray while counting zeros and ones
Optimal	O(n) average	O(n)	Transform the binary array into a running balance and use a HashMap to find repeated balances


The Complete Logic to Remember

When you encounter a problem asking for the longest subarray with equal numbers of zeros and ones, think:

1. Equal zeros and ones means their difference is zero.
2. Transform 0 to -1 and keep 1 as +1.
3. Maintain a running prefix balance.
4. A repeated balance means the subarray between the occurrences has a sum of zero.
5. Store the earliest index of every balance.
6. When a balance repeats, calculate currentIndex - earliestIndex.
7. Update the maximum length.
8. Initialize the map with 0 → -1 to handle subarrays starting at index 0.

Before Moving to the Next Problem

Try answering these three questions without looking at the solution:

1. Why does a repeated prefix balance guarantee that the subarray between those occurrences has equal zeros and ones?
2. What would go wrong if you updated the stored index every time the same balance appeared?
3. Why do we initialize the map with firstIndex.put(0, -1) instead of leaving it empty?

If you can explain all three in your own words, you understand the core idea behind LeetCode 525 — Contiguous Array.