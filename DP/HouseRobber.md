# **LeetCode 198 — House Robber**

Medium

Dynamic Programming

Take / Not-Take

This is an important DP problem because it introduces the **Take / Not-Take pattern**, which appears frequently in coding assessments.

Let's derive it step by step using your 8-step framework.

## **Step 1. Understand the problem**

You are a professional robber planning to rob houses along a street.

Each house contains some amount of money, represented by an array:

```
nums = [2, 7, 9, 3, 1]
```

There is one restriction:

**You cannot rob two adjacent houses.**

If you rob one house, you cannot rob the house immediately before or after it.

Your goal is to find the maximum amount of money you can rob.

### **Example**

**Houses and their money**

# **2**

House 0

# **7**

House 1

# **9**

House 2

# **3**

House 3

# **1**

House 4

Selected houses: 0, 2, 4

Valid selection

# **12**

2 + 9 + 1 = 12

We can rob houses 0, 2 and 4 because none of them are adjacent.

We cannot rob houses 0 and 1 together because they are next to each other.

**Expected answer: 12**

## **Step 2. Brute-force approach — Recursion**

First, think about what choices we have at every house.

At house `i`, we have exactly two choices:

1. **Take:** Rob the current house and move to `i + 2`.
2. **Not Take:** Skip the current house and move to `i + 1`.

Why do we move to `i + 2` after taking?

Because we cannot rob the adjacent house.

### **Define the recursive function**

Let:

\\[ f(i)=\text{maximum money obtainable starting from house }i \\]

Our two choices:

**Choice 1: Take the current house**

We earn `nums[i]` and continue from `i + 2`.

\\[ take=nums[i]+f(i+2) \\]

**Choice 2: Don't take the current house**

We earn nothing from this house and continue from `i + 1`.

\\[ notTake=f(i+1) \\]

We choose the maximum:

\\[ \boxed{f(i)=\max(take,notTake)} \\]

### **Base case**

If we move beyond the last house, there is nothing left to rob.

\\[ f(i)=0 \quad \text{if }i\ge n \\]

### **Brute-force Java code**

```

class Solution {

    public int rob(int[] nums) {
        return solve(nums, 0);
    }

    private int solve(int[] nums, int i) {

        // No houses left to rob.
        if (i >= nums.length) {
            return 0;
        }

        // Choice 1: Rob the current house.
        // We must skip the adjacent house.
        int take = nums[i] + solve(nums, i + 2);

        // Choice 2: Skip the current house.
        // Move to the next house.
        int notTake = solve(nums, i + 1);

        // Choose the option giving more money.
        return Math.max(take, notTake);
    }
}

```

### **Complete dry run:&#x20;**`nums = [2, 7, 9]`

Start:

```
solve(nums, 0)
```

At index 0, we have two choices:

```
TAKE:
2 + solve(2)

NOT TAKE:
solve(1)
```

Let's calculate both.

**Branch 1: Take house 0**

```
2 + solve(2)
```

At index 2:

- Take: `9 + solve(4)` = 9
- Not Take: `solve(3)` = 0

Therefore:

```
solve(2) = max(9, 0) = 9
```

So:

```
take = 2 + 9 = 11
```

**Branch 2: Skip house 0**

```
solve(1)
```

At index 1:

- Take: `7 + solve(3)` = 7
- Not Take: `solve(2)` = 9

Therefore:

```
solve(1) = max(7, 9) = 9
```

Finally:

```
solve(0) = max(11, 9)
         = 11
```

Answer = 11

The optimal selection is house 0 and house 2.

### **Complexity**

- Time: \\(O(2^n)\\) — many repeated recursive calls.
- Space: \\(O(n)\\) — recursion stack.

## **Step 3. Identify the bottleneck**

Consider this recursion:

```
solve(0)
├── take → solve(2)
│           ├── solve(4)
│           └── solve(3)
│
└── notTake → solve(1)
              ├── solve(3)
              └── solve(2)
```

Notice:

- `solve(2)` is calculated more than once.
- `solve(3)` is calculated more than once.

We are solving the same subproblems repeatedly.

Can we save their answers?

Yes! That's where Dynamic Programming comes in.

## **Step 4. Recognize the DP pattern**

This problem follows the **Take / Not-Take DP pattern**.

At every index, we decide whether to take the current element or skip it.

Our recurrence remains:

\\[ \boxed{dp[i]=\max(nums[i]+dp[i+2],dp[i+1])} \\]

Where:

- `dp[i]` = maximum money we can rob starting from index `i`.
- `nums[i] + dp[i+2]` = take.
- `dp[i+1]` = not take.

Base case:

\\[ dp[i]=0 \quad \text{when }i\ge n \\]

We have overlapping subproblems and optimal substructure, so DP is suitable.

## **Step 5. Derive the optimal approach**

### **Approach 1: Memoization (Top-down DP)**

We keep the recursive structure but store calculated results.

```
dp[i] = -1;
```

Initially, `-1` means the answer hasn't been calculated.

When a state is requested:

- If already calculated, return its stored value.
- Otherwise, calculate both choices and save the maximum.

```

import java.util.Arrays;

class Solution {

    public int rob(int[] nums) {

        int n = nums.length;

        int[] dp = new int[n];

        // -1 indicates an uncalculated state.
        Arrays.fill(dp, -1);

        return solve(nums, 0, dp);
    }

    private int solve(int[] nums, int i, int[] dp) {

        if (i >= nums.length) {
            return 0;
        }

        // Reuse previously calculated results.
        if (dp[i] != -1) {
            return dp[i];
        }

        // Take the current house.
        int take = nums[i] + solve(nums, i + 2, dp);

        // Skip the current house.
        int notTake = solve(nums, i + 1, dp);

        // Store the best answer for this index.
        dp[i] = Math.max(take, notTake);

        return dp[i];
    }
}

```

Complexity:

- Time: \\(O(n)\\)
- Space: \\(O(n)\\) for the DP array and recursion stack.

### **Approach 2: Tabulation (Bottom-up DP)**

Instead of recursion, we calculate answers from the end of the array towards the beginning.

Let's use:

```
nums = [2, 7, 9, 3, 1]
```

Our recurrence:

\\[ dp[i]=\max(nums[i]+dp[i+2],dp[i+1]) \\]

We need two base cases:

```
dp[n] = 0;
dp[n + 1] = 0;
```

Why?

Because there are no houses to rob beyond the array.

### **Build the DP table backwards**

| Index | House money | Take        | Not Take | dp[i] |
| ----- | ----------- | ----------- | -------- | ----- |
| 5     | —           | 0           | 0        | 0     |
| 4     | 1           | 1 + 0 = 1   | 0        | 1     |
| 3     | 3           | 3 + 0 = 3   | 1        | 3     |
| 2     | 9           | 9 + 1 = 10  | 3        | 10    |
| 1     | 7           | 7 + 3 = 10  | 10       | 10    |
| 0     | 2           | 2 + 10 = 12 | 10       | 12    |

Answer:

\\[ dp[0]=12 \\]

### **Java code**

```

class Solution {

    public int rob(int[] nums) {

        int n = nums.length;

        // Extra two positions for base cases.
        int[] dp = new int[n + 2];

        // Build the answer from right to left.
        for (int i = n - 1; i >= 0; i--) {

            // Choice 1: Rob current house.
            int take = nums[i] + dp[i + 2];

            // Choice 2: Skip current house.
            int notTake = dp[i + 1];

            // Store the maximum of both choices.
            dp[i] = Math.max(take, notTake);
        }

        return dp[0];
    }
}

```

Complexity:

- Time: \\(O(n)\\)
- Space: \\(O(n)\\)

## **Step 6. Optimal Java solution — Space optimization**

Look carefully at our recurrence:

\\[ dp[i]=\max(nums[i]+dp[i+2],dp[i+1]) \\]

To calculate `dp[i]`, we only need:

- `dp[i+1]`
- `dp[i+2]`

We don't need the entire DP array.

So we can maintain two variables:

```
prev1 = dp[i + 1];
prev2 = dp[i + 2];
```

And calculate:

```
current = Math.max(nums[i] + prev2, prev1);
```

Then shift:

```
prev2 = prev1;
prev1 = current;
```

### **Optimal code**

Recommended solution

```

class Solution {

    public int rob(int[] nums) {

        // Maximum money from i+1 and i+2.
        int prev1 = 0;
        int prev2 = 0;

        // Traverse houses from right to left.
        for (int i = nums.length - 1; i >= 0; i--) {

            // Choice 1: Rob current house.
            int take = nums[i] + prev2;

            // Choice 2: Skip current house.
            int notTake = prev1;

            // Choose the maximum amount.
            int current = Math.max(take, notTake);

            // Shift the previous DP states.
            prev2 = prev1;
            prev1 = current;
        }

        return prev1;
    }
}

```

Complexity:

| Metric          | Complexity |
| --------------- | ---------- |
| Time            | \\(O(n)\\) |
| Auxiliary space | \\(O(1)\\) |

## **Step 7. Complete dry run of optimal code**

Input:

```
nums = [2, 7, 9, 3, 1]
```

Initial values:

```
prev1 = 0;
prev2 = 0;
```

We iterate backwards, starting from index 4.

### **Iteration 1:&#x20;**`i = 4`

```
nums[4] = 1

take = 1 + 0 = 1
notTake = 0

current = max(1, 0) = 1
```

Update:

```
prev2 = 0;
prev1 = 1;
```

### **Iteration 2:&#x20;**`i = 3`

```
nums[3] = 3

take = 3 + 0 = 3
notTake = 1

current = max(3, 1) = 3
```

Update:

```
prev2 = 1;
prev1 = 3;
```

### **Iteration 3:&#x20;**`i = 2`

```
nums[2] = 9

take = 9 + 1 = 10
notTake = 3

current = max(10, 3) = 10
```

Update:

```
prev2 = 3;
prev1 = 10;
```

### **Iteration 4:&#x20;**`i = 1`

```
nums[1] = 7

take = 7 + 3 = 10
notTake = 10

current = max(10, 10) = 10
```

Update:

```
prev2 = 10;
prev1 = 10;
```

### **Iteration 5:&#x20;**`i = 0`

```
nums[0] = 2

take = 2 + 10 = 12
notTake = 10

current = max(12, 10) = 12
```

Update:

```
prev2 = 10;
prev1 = 12;
```

### **Complete dry-run table**

| i | nums[i] | prev2 before | prev1 before | Take | Not Take | Current |
| - | ------- | ------------ | ------------ | ---- | -------- | ------- |
| 4 | 1       | 0            | 0            | 1    | 0        | 1       |
| 3 | 3       | 0            | 1            | 3    | 1        | 3       |
| 2 | 9       | 1            | 3            | 10   | 3        | 10      |
| 1 | 7       | 3            | 10           | 10   | 10       | 10      |
| 0 | 2       | 10           | 10           | 12   | 10       | 12      |

Finally:

```
return prev1;
```

Maximum money robbed

# **12**

Houses 0, 2 and 4 → 2 + 9 + 1

## **Step 8. Tricky parts and key insights**

### **1. Why do we move to&#x20;**`i + 2`**&#x20;after taking?**

Because adjacent houses cannot both be robbed.

If we rob house 2, we must skip house 3.

### **2. Why do we use&#x20;**`Math.max()`**?**

Because we want the maximum possible money, not the minimum cost.

Compare:

- LC 746: `Math.min()` because we minimize cost.
- LC 198: `Math.max()` because we maximize profit.

### **3. Why do we iterate backwards?**

Our recurrence uses the next two positions:

```
dp[i + 1]
dp[i + 2]
```

So we calculate the answers for later houses first.

### **4. What if there is only one house?**

```
nums = [5]
```

We rob it and return 5.

The space-optimized solution handles this automatically.

### **5. What if all houses contain zero?**

```
nums = [0, 0, 0]
```

The answer is 0. The algorithm correctly chooses the maximum between taking and skipping.

### **6. What is the key difference between LC 70, 746 and 198?**

| Problem | DP decision                               |
| ------- | ----------------------------------------- |
| LC 70   | Add ways from previous two positions      |
| LC 746  | Minimize cost from previous two positions |
| LC 198  | Maximize money by taking or skipping      |

## **GitHub-ready notes — DSA-Java**

**LC 198 — House Robber**

- **Pattern:** 1D DP — Take / Not-Take
- **Problem:** Maximize money robbed without robbing adjacent houses.
- **State:** `dp[i]` = maximum money obtainable from index `i` onwards.
- **Take:** `nums[i] + dp[i+2]`
- **Not Take:** `dp[i+1]`
- **Recurrence:** `dp[i] = max(nums[i] + dp[i+2], dp[i+1])`
- **Base cases:** `dp[n] = dp[n+1] = 0`
- **Optimal time:** O(n)
- **Optimal space:** O(1)
- **Key insight:** At every house, choose between robbing it and skipping the next house, or skipping it and considering the next house.

## **Practice — Test your understanding**

Don't rush to the answers. Try to reason through each question.

**Q1. If we rob house i, which index should we consider next?**

i + 1

i + 2

i - 1

**Q2. What does the Take choice calculate?**

nums[i] + dp[i+1]

nums[i] + dp[i+2]

dp[i+1] + dp[i+2]

**Q3. For nums = [2, 1, 1, 2], what is the maximum money?**

3

4

5

**Q4. Why do we traverse backwards in the tabulation solution?**

Because the recurrence needs dp[i+1] and dp[i+2]

Because the array must be sorted

Because we need the minimum value

Check answers

**Next recommended problem: LC 213 — House Robber II.**

It is the natural extension of this problem: the houses are arranged in a circle, so the first and last houses are also adjacent. You'll reuse the same Take / Not-Take DP logic with one additional observation.
