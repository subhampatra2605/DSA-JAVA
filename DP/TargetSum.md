# **LeetCode 494 — Target Sum**



Medium

0/1 Knapsack · Subset Sum · Dynamic Programming



This problem is a direct extension of **LC 416 — Partition Equal Subset Sum**. In LC 416, we asked whether a subset could form a target. Here, we count how many ways we can form a target by assigning `+` and `-` signs to the array elements.

The key is to convert the sign-assignment problem into a **subset-sum counting problem**.

## **Step 1 — Understand the problem**

You are given an integer array `nums` and an integer `target`.

You must put either a `+` or `-` sign before every element so that the final sum equals `target`.

Return the number of different ways to do this.

**Example:**

```
nums = [1, 1, 1, 1, 1]
target = 3
```

One valid assignment is:

```
+1 +1 +1 +1 -1 = 3
```

There are five different assignments that produce 3.

Answer = 5

### **Understand what we are counting**

Each element must be used exactly once. We are not choosing whether to use an element; we are choosing whether to add or subtract it.

For `[1, 1, 1]`, the possible assignments include:

| Assignment | Result |
| ---------- | ------ |
| `+1 +1 +1` | 3      |
| `+1 +1 -1` | 1      |
| `+1 -1 +1` | 1      |
| `-1 +1 +1` | 1      |
| `+1 -1 -1` | -1     |
| `-1 +1 -1` | -1     |
| `-1 -1 +1` | -1     |
| `-1 -1 -1` | -3     |

For target `1`, there are 3 ways. Notice that different positions matter, even when values are equal.

## **Step 2 — Derive the subset-sum transformation**

This is the most important part of the problem. Let's derive it instead of memorizing a formula.

Imagine dividing the numbers into two groups:

- **Positive group (P):** numbers assigned `+`.
- **Negative group (N):** numbers assigned `-`.

For example:

```
nums = [1, 2, 3, 4]

P = [1, 3, 4]
N = [2]

P sum = 8
N sum = 2

8 - 2 = 6
```

We want:

\\[ P-N=target \\]

We also know that every element is used exactly once, so:

\\[ P+N=totalSum \\]

Now add these two equations:

\\[ (P-N)+(P+N)=target+totalSum \\]

The negative group cancels:

\\[ 2P=target+totalSum \\]

Therefore:

\\[ \boxed{P=\frac{totalSum+target}{2}} \\]

This means:

**Instead of assigning plus and minus signs, we can count how many subsets have a sum equal to&#x20;**`(totalSum + target) / 2`**.**

For our example:

```
nums = [1, 2, 3, 4]
target = 2
totalSum = 10

P = (10 + 2) / 2
P = 6
```

So we need to count subsets whose sum is 6.

For example:

```
[2, 4] → sum = 6
[1, 2, 3] → sum = 6
```

There are 2 subsets, so there are 2 valid sign assignments.

## **Step 3 — Brute-force recursion**

Before optimizing, let's solve the problem exactly as stated.

At every index, we have two choices:

1. Add the current number.
2. Subtract the current number.

For `nums = [1,1,1]` and `target = 1`, the recursion explores all \\(2^3=8\\) assignments.

### **Define the recursive state**

Let:

```
countWays(index, currentSum)
```

mean:

> The number of ways to reach the required target, starting from `index`, when the sum accumulated so far is `currentSum`.

At each index:

- Add choice: `currentSum + nums[index]`
- Subtract choice: `currentSum - nums[index]`

When all elements have been processed, check whether the sum equals the target.

### **Java — brute force**

```
class Solution {
    public int findTargetSumWays(int[] nums, int target) {
        return solve(nums, 0, 0, target);
    }

    private int solve(int[] nums, int index, int currentSum, int target) {

        // All elements have been assigned a sign.
        // This complete assignment contributes one way
        // only if its final sum equals the target.
        if (index == nums.length) {
            return currentSum == target ? 1 : 0;
        }

        // Choice 1: Assign a positive sign to the current number.
        int add = solve(
            nums, index + 1, currentSum + nums[index], target
        );

        // Choice 2: Assign a negative sign to the current number.
        int subtract = solve(
            nums, index + 1, currentSum - nums[index], target
        );

        // Count the successful assignments from both choices.
        return add + subtract;
    }
}
```

**Why do we return&#x20;**`add + subtract`**&#x20;instead of&#x20;**`add || subtract`**?**

Because we need the number of ways, not just whether at least one way exists.

For example:

- Positive choice produces 2 valid assignments.
- Negative choice produces 3 valid assignments.
- Total = 2 + 3 = 5 ways.

### **Dry run:&#x20;**`nums = [1,1]`**,&#x20;**`target = 0`

Follow the recursion one choice at a time.

**Index 0, sum = 0**

Choose `+1` → index 1, sum = 1

**Index 1, sum = 1**

- Choose `+1` → final sum = 2 → returns 0
- Choose `-1` → final sum = 0 → returns 1

This branch returns `0 + 1 = 1`.

**Back at index 0, sum = 0**

Now choose `-1` → index 1, sum = -1

**Index 1, sum = -1**

- Choose `+1` → final sum = 0 → returns 1
- Choose `-1` → final sum = -2 → returns 0

This branch returns `1 + 0 = 1`.

**Final result: 1 + 1 = 2 ways**

The two valid assignments are `+1 -1` and `-1 +1`.

**Complexity**

- Time: \\(O(2^n)\\), because every element creates two recursive choices.
- Recursion stack: \\(O(n)\\).

## **Step 4 — Identify the bottleneck**

The brute-force solution explores every possible sign assignment.

But notice something:

Different choices can lead to the **same index and the same current sum**.

For example, with repeated values, multiple paths can reach the same state. Once we know the number of ways from that state, we don't need to calculate it again.

This is the overlapping-subproblems property of Dynamic Programming.

Our recursive state is:

```
(index, currentSum)
```

We can store the answer for each state using memoization.

## **Step 5 — Memoization (Top-Down DP)**

We create a 2D DP array:

```
dp[index][currentSum]
```

It stores the number of ways to reach the target from that state.

However, `currentSum` can be negative. A Java array cannot use a negative index.

We solve this by using an offset.

If:

```
totalSum = sum of all elements
```

The possible current sums lie between `-totalSum` and `+totalSum`.

So we use:

```
offset = totalSum;
dp[index][currentSum + offset]
```

For example, if `totalSum = 5`:

| Actual sum | Array index |
| ---------- | ----------- |
| -5         | 0           |
| -3         | 2           |
| 0          | 5           |
| 2          | 7           |
| 5          | 10          |

### **Java — memoization**

```
class Solution {
    public int findTargetSumWays(int[] nums, int target) {
        int totalSum = 0;

        for (int num : nums) {
            totalSum += num;
        }

        // No assignment can produce a target outside this range.
        if (Math.abs(target) > totalSum) {
            return 0;
        }

        int offset = totalSum;

        // -1 means that this state has not been calculated yet.
        int[][] dp = new int[nums.length][2 * totalSum + 1];

        for (int[] row : dp) {
            Arrays.fill(row, -1);
        }

        return solve(nums, 0, 0, target, offset, dp);
    }

    private int solve(
        int[] nums, int index, int currentSum,
        int target, int offset, int[][] dp
    ) {
        if (index == nums.length) {
            return currentSum == target ? 1 : 0;
        }

        int col = currentSum + offset;

        // Reuse the answer if this state was already calculated.
        if (dp[index][col] != -1) {
            return dp[index][col];
        }

        // Assign a positive sign.
        int add = solve(
            nums, index + 1, currentSum + nums[index],
            target, offset, dp
        );

        // Assign a negative sign.
        int subtract = solve(
            nums, index + 1, currentSum - nums[index],
            target, offset, dp
        );

        // Store the total number of valid ways from this state.
        dp[index][col] = add + subtract;

        return dp[index][col];
    }
}
```

Add this import if using the code in a standalone Java file:

```
import java.util.Arrays;
```

**Complexity**

- Time: \\(O(n \times totalSum)\\)
- Space: \\(O(n \times totalSum)\\) for the memo table, plus \\(O(n)\\) recursion stack.

## **Step 6 — Convert it into 0/1 Knapsack counting**

Now let's use the transformation from Step 2.

We derived:

\\[ P=\frac{totalSum+target}{2} \\]

So our task is:

> Count the number of subsets whose sum equals `requiredSum`.

This is almost the same as LC 416, but there is one important difference:

| LC 416                                | LC 494                            |
| ------------------------------------- | --------------------------------- |
| Is a subset with target sum possible? | How many subsets have target sum? |
| Boolean DP                            | Integer DP                        |
| `true` / `false`                      | Number of ways                    |
| Combine using `OR`                    | Combine using `+`                 |

### **6.1 Define the DP state**

```
dp[t]
```

means:

> The number of ways to form sum `t` using the elements processed so far.

Initially:

```
dp[0] = 1;
```

Why 1?

There is exactly one way to make sum zero before processing any elements: choose nothing (the empty subset).

All other sums initially have zero ways.

### **6.2 Transition**

For every number `num`, we have two choices for making sum `t`:

1. Do not take `num`: the number of ways remains `dp[t]`.
2. Take `num`: add the number of ways to form `t - num`, which is `dp[t - num]`.

Therefore:

\\[ \boxed{dp[t]=dp[t]+dp[t-num]} \\]

We iterate backwards from `requiredSum` down to `num`, just like LC 416.

**Why backwards?** Each array element can be used only once. Descending iteration ensures that `dp[t-num]` still represents combinations formed without reusing the current element.

### **6.3 Optimal Java solution**

Recommended solution

```
class Solution {
    public int findTargetSumWays(int[] nums, int target) {

        int totalSum = 0;

        // Calculate the sum of all elements.
        for (int num : nums) {
            totalSum += num;
        }

        // The target cannot be larger than the total sum
        // in absolute value.
        if (Math.abs(target) > totalSum) {
            return 0;
        }

        // The required subset sum must be an integer.
        if ((totalSum + target) % 2 != 0) {
            return 0;
        }

        // Convert Target Sum into a subset-sum counting problem.
        int requiredSum = (totalSum + target) / 2;

        // dp[t] = number of ways to form sum t.
        int[] dp = new int[requiredSum + 1];

        // There is one way to form sum 0: choose no elements.
        dp[0] = 1;

        // Process every array element exactly once.
        for (int num : nums) {

            // Go backwards to avoid using the same element twice.
            for (int t = requiredSum; t >= num; t--) {

                // Ways without this number + ways using this number.
                dp[t] += dp[t - num];
            }
        }

        return dp[requiredSum];
    }
}
```

**Complexity**

- Time: \\(O(n \times requiredSum)\\)
- Space: \\(O(requiredSum)\\)

## **Step 7 — Complete dry run of the optimal solution**

Let's use the original example:

```
nums = [1, 1, 1, 1, 1]
target = 3
```

### **7.1 Calculate the required subset sum**

```
totalSum = 1 + 1 + 1 + 1 + 1
         = 5

requiredSum = (totalSum + target) / 2
            = (5 + 3) / 2
            = 4
```

We need to count subsets that add up to 4.

### **7.2 Initialize DP**

```
int[] dp = new int[5];
dp[0] = 1;
```

Index represents the sum; value represents the number of ways.

| Sum        | 0 | 1 | 2 | 3 | 4 |
| ---------- | - | - | - | - | - |
| Initial DP | 1 | 0 | 0 | 0 | 0 |

Initially, only sum 0 is possible, in one way: selecting nothing.

### **7.3 Process the first&#x20;**`1`

The inner loop runs:

```
for (int t = 4; t >= 1; t--)
```

- `t = 4`: `dp[4] += dp[3]` → 0 + 0 = 0
- `t = 3`: `dp[3] += dp[2]` → 0 + 0 = 0
- `t = 2`: `dp[2] += dp[1]` → 0 + 0 = 0
- `t = 1`: `dp[1] += dp[0]` → 0 + 1 = 1

DP becomes:

| Sum           | 0 | 1 | 2 | 3 | 4 |
| ------------- | - | - | - | - | - |
| After 1st `1` | 1 | 1 | 0 | 0 | 0 |

We can form:

- Sum 0: 1 way
- Sum 1: 1 way

### **7.4 Process the second&#x20;**`1`

Again, iterate backwards from 4 to 1.

- `t = 4`: `dp[4] += dp[3]` → 0 + 0 = 0
- `t = 3`: `dp[3] += dp[2]` → 0 + 0 = 0
- `t = 2`: `dp[2] += dp[1]` → 0 + 1 = 1
- `t = 1`: `dp[1] += dp[0]` → 1 + 1 = 2

DP becomes:

| Sum           | 0 | 1 | 2 | 3 | 4 |
| ------------- | - | - | - | - | - |
| After 2nd `1` | 1 | 2 | 1 | 0 | 0 |

Why is `dp[1] = 2`?

Because there are two different elements, and either one can form sum 1.

### **7.5 Process the remaining elements**

The same calculation happens for each remaining `1`.

Complete DP progression

| Processed elements | Sum 0 | Sum 1 | Sum 2 | Sum 3 | Sum 4 |
| ------------------ | ----- | ----- | ----- | ----- | ----- |
| None               | 1     | 0     | 0     | 0     | 0     |
| `[1]`              | 1     | 1     | 0     | 0     | 0     |
| `[1,1]`            | 1     | 2     | 1     | 0     | 0     |
| `[1,1,1]`          | 1     | 3     | 3     | 1     | 0     |
| `[1,1,1,1]`        | 1     | 4     | 6     | 4     | 1     |
| `[1,1,1,1,1]`      | 1     | 5     | 10    | 10    | 5     |

Final answer

# **5 ways**

`dp[requiredSum] = dp[4] = 5`

There are five ways to select a subset of sum 4: choose any four of the five `1`s.

The remaining one element gets a negative sign, giving five different sign assignments.

## **Step 8 — Tricky cases and interview takeaways**

### **8.1 When target is greater than total sum**

```
nums = [1, 2, 3]
target = 10
totalSum = 6
```

The maximum possible positive sum is 6. So target 10 is impossible.

Return `0`.

### **8.2 When&#x20;**`(totalSum + target)`**&#x20;is odd**

```
nums = [1, 2, 3]
target = 1

requiredSum = (6 + 1) / 2 = 3.5
```

A subset sum must be an integer. Therefore, return `0`.

This is why we check:

```
if ((totalSum + target) % 2 != 0) {
    return 0;
}
```

### **8.3 When the array contains zero**

Consider:

```
nums = [0, 0, 1]
target = 1
```

Each zero has two choices:

- `+0`
- `-0`

Both produce the same sum, but they represent different sign assignments.

Therefore, the answer is 4.

Our DP handles this correctly:

```
dp[t] += dp[t];
```

When `num = 0`, each existing way doubles because the zero can receive either sign.

### **8.4 Why do we iterate backwards?**

This is the same important rule you learned in LC 416.

Suppose:

```
nums = [2]
requiredSum = 4
```

If we iterate forwards:

- `t = 2`: `dp[2] = dp[2] + dp[0] = 1`
- `t = 4`: `dp[4] = dp[4] + dp[2] = 1`

We have incorrectly formed 4 using the same `2` twice!

But with descending iteration:

- `t = 4`: `dp[4] += dp[2]` → 0
- `t = 2`: `dp[2] += dp[0]` → 1

The same element is not reused.

**Remember:**

- 0/1 Knapsack → iterate backwards.
- Unbounded Knapsack → usually iterate forwards.

## **GitHub-ready notes — DSA-Java**

```

# 494. Target Sum

## Problem
Assign either + or - to every element in nums such that the resulting sum equals target. Return the number of valid assignments.

## Approach
Convert the sign-assignment problem into subset-sum counting.

Let:
- P = sum of positively assigned elements
- N = sum of negatively assigned elements
- S = total sum

P - N = target
P + N = S

Therefore:
P = (S + target) / 2

Count the subsets whose sum equals (S + target) / 2.

## Pattern
0/1 Knapsack — Subset Sum Counting

## DP State
dp[t] = number of ways to form sum t using the processed elements.

## Transition
dp[t] += dp[t - num]

## Initialization
dp[0] = 1

## Important Conditions
- If abs(target) > totalSum, return 0.
- If (totalSum + target) is odd, return 0.
- Iterate backwards to avoid reusing an element.
- Zero doubles the number of existing ways.

## Complexity
Time: O(n * requiredSum)
Space: O(requiredSum)

## Key Insight
Target Sum is a subset-sum counting problem, not merely a subset-sum existence problem. The difference from LC 416 is that we count ways using addition instead of boolean OR.

## Java Code

```

## **Compare LC 416 and LC 494**

| Concept       | LC 416                       | LC 494               |
| ------------- | ---------------------------- | -------------------- |
| Goal          | Find whether a subset exists | Count subsets        |
| DP type       | Boolean                      | Integer              |
| Initial state | `dp[0] = true`               | `dp[0] = 1`          |
| Transition    | `dp[t] \\|\\| dp[t-num]`     | `dp[t] += dp[t-num]` |
| Iteration     | Backwards                    | Backwards            |
| Pattern       | 0/1 Knapsack                 | 0/1 Knapsack         |

The important learning progression is:

- **LC 416:** Can I form this sum?
- **LC 494:** How many ways can I form this sum?

## **Test your understanding**

Try answering these without looking at the explanation.

**LC 494 mini quiz**

0/4 answered

**1. Why do we calculate (totalSum + target) / 2?**

To find the number of elements

To convert sign assignment into subset-sum counting

To calculate the maximum element

**2. What does dp[t] represent in LC 494?**

Whether sum t is possible

The minimum elements needed for sum t

The number of ways to form sum t

**3. Why do we iterate backwards?**

To sort the array

To prevent using the same element more than once

To avoid negative numbers

**4. nums = [0, 0, 1], target = 1. What is the answer?**

1

2

4

0

Check answers

**Next recommended problem: LC 518 — Coin Change II.** It introduces a useful contrast: counting combinations in Unbounded Knapsack, where the inner loop goes forwards rather than backwards.
