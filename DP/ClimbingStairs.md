# **LeetCode 70 — Climbing Stairs**

Easy

Dynamic Programming

Fibonacci Pattern

This is one of the best problems to understand how recursion gradually becomes Dynamic Programming (DP).

Let's derive the solution rather than directly memorizing the formula.

## **Step 1. Understand the problem**

**Problem:** You are climbing a staircase with `n` steps.

You can climb either:

- 1 step at a time.
- 2 steps at a time.

Find the total number of distinct ways to reach the top.

### **Example 1**

```
n = 2
```

There are two possible ways:

**Way 1**

1

1

Climb 1 step, then another 1 step.

**Way 2**

2

Climb 2 steps at once.

Answer:

```
2
```

### **Example 2**

```
n = 3
```

Possible ways:

1. `1 + 1 + 1`
2. `1 + 2`
3. `2 + 1`

Answer:

```
3
```

**Important:** `1 + 2` and `2 + 1` are different ways because the order of the steps matters.

## **Step 2. Think about the brute-force approach**

Before writing code, ask yourself:

**If I am currently standing at a particular step, where can I go next?**

I have two choices:

- Move 1 step forward.
- Move 2 steps forward.

For example, suppose `n = 3`.

Starting at step 0:

```
             Step 0
             /    \
         Step 1   Step 2
          /  \       |
      Step 2 Step 3 Step 3
        |
      Step 3
```

Every path that reaches step 3 represents one distinct way.

### **Recursive relationship**

Let:

\\[ f(n)=\text{number of ways to reach step }n \\]

To reach step `n`, we must have come from:

- Step `n-1` using a 1-step jump.
- Step `n-2` using a 2-step jump.

Therefore:

\\[ \boxed{f(n)=f(n-1)+f(n-2)} \\]

Base cases:

\\[ f(1)=1 \\]

\\[ f(2)=2 \\]

### **Brute-force Java code**

```

class Solution {
    public int climbStairs(int n) {
        return solve(n);
    }

    private int solve(int n) {

        // One way to reach step 1.
        if (n == 1) {
            return 1;
        }

        // Two ways to reach step 2.
        if (n == 2) {
            return 2;
        }

        // Try reaching from one step below
        // or two steps below.
        return solve(n - 1) + solve(n - 2);
    }
}

```

### **Dry run: n = 4**

We calculate:

```
solve(4)
= solve(3) + solve(2)
```

Expand `solve(3)`:

```
solve(4)
= solve(2) + solve(1) + solve(2)
```

Substitute base cases:

```
= 2 + 1 + 2
= 5
```

Answer = 5

The five possible sequences are:

```
1 + 1 + 1 + 1
1 + 1 + 2
1 + 2 + 1
2 + 1 + 1
2 + 2
```

### **Complexity**

- Time: \\(O(2^n)\\) — repeated recursive calculations.
- Space: \\(O(n)\\) — recursion call stack.

The problem is that we calculate the same results multiple times.

## **Step 3. Identify the bottleneck**

Look at this recursion:

```
solve(5)
    ├── solve(4)
    │     ├── solve(3)
    │     └── solve(2)
    │
    └── solve(3)
          ├── solve(2)
          └── solve(1)
```

Notice something?

`solve(3)` is calculated twice.

`solve(2)` is also calculated multiple times.

**Repeated calculations**

`solve(3)`

Calculated twice

`solve(2)`

Calculated multiple times

Instead of recalculating the same answer, we should save it and reuse it.

This is where Dynamic Programming comes in.

## **Step 4. Recognize the DP pattern**

This problem has two important DP properties:

1. **Overlapping subproblems:** The same smaller problems are solved repeatedly.
2. **Optimal substructure:** The answer for a larger problem is constructed from answers to smaller problems.

The recurrence is:

\\[ dp[n]=dp[n-1]+dp[n-2] \\]

Think of `dp[i]` as:

**The number of distinct ways to reach stair&#x20;**`i`**.**

Our DP array will store previously calculated answers.

## **Step 5. Derive the optimal approach**

Let's build the DP table for `n = 5`.

We know:

\\[ dp[1]=1 \\]

\\[ dp[2]=2 \\]

Every subsequent position is the sum of the previous two.

| Stair | Calculation | Ways |
| ----- | ----------- | ---- |
| 1     | Base case   | 1    |
| 2     | Base case   | 2    |
| 3     | 1 + 2       | 3    |
| 4     | 2 + 3       | 5    |
| 5     | 3 + 5       | 8    |

**DP array**

# **1**

dp[1]

# **2**

dp[2]

# **3**

dp[3]

# **5**

dp[4]

# **8**

dp[5]

Each new value is obtained by adding the previous two values.

### **Approach 1: Memoization (Top-down DP)**

We retain recursion but store calculated answers in an array.

```

import java.util.Arrays;

class Solution {

    public int climbStairs(int n) {

        int[] dp = new int[n + 1];

        // -1 means the answer has not been calculated.
        Arrays.fill(dp, -1);

        return solve(n, dp);
    }

    private int solve(int n, int[] dp) {

        if (n == 1) return 1;
        if (n == 2) return 2;

        // Reuse an already calculated answer.
        if (dp[n] != -1) {
            return dp[n];
        }

        // Calculate and store the answer.
        dp[n] = solve(n - 1, dp) + solve(n - 2, dp);

        return dp[n];
    }
}

```

Complexity:

- Time: \\(O(n)\\)
- Space: \\(O(n)\\) for the DP array and recursion stack.

### **Approach 2: Tabulation (Bottom-up DP)**

Instead of starting from `n` and going backwards, we start from the base cases and build towards `n`.

```
dp[1] = 1;
dp[2] = 2;
```

Then:

```
dp[i] = dp[i - 1] + dp[i - 2];
```

Java code:

```

class Solution {
    public int climbStairs(int n) {

        if (n <= 2) {
            return n;
        }

        int[] dp = new int[n + 1];

        dp[1] = 1;
        dp[2] = 2;

        for (int i = 3; i <= n; i++) {

            // Ways to reach i come from i-1 or i-2.
            dp[i] = dp[i - 1] + dp[i - 2];
        }

        return dp[n];
    }
}

```

Complexity:

- Time: \\(O(n)\\)
- Space: \\(O(n)\\)

### **Approach 3: Space optimization (Optimal)**

Now ask yourself:

Do we really need to store the entire DP array?

Look at the recurrence:

\\[ dp[i]=dp[i-1]+dp[i-2] \\]

We only need the previous two values to calculate the next value.

So instead of an array, we can maintain two variables.

```
int prev2 = 1;
int prev1 = 2;
```

For every next stair:

```
int current = prev1 + prev2;
prev2 = prev1;
prev1 = current;
```

This is enough!

## **Step 6. Optimal Java solution**

Recommended solution

```

class Solution {
    public int climbStairs(int n) {

        // Base cases.
        if (n <= 2) {
            return n;
        }

        // Ways to reach the previous two stairs.
        int prev2 = 1;
        int prev1 = 2;

        // Calculate ways for stairs 3 through n.
        for (int i = 3; i <= n; i++) {

            // Current ways = previous two ways.
            int current = prev1 + prev2;

            // Shift the previous values forward.
            prev2 = prev1;
            prev1 = current;
        }

        return prev1;
    }
}

```

Complexity:

| Complexity      | Value      |
| --------------- | ---------- |
| Time            | \\(O(n)\\) |
| Auxiliary space | \\(O(1)\\) |

## **Step 7. Complete dry run of optimal code**

Input:

```
n = 5
```

Initial values:

```
prev2 = 1;
prev1 = 2;
```

The loop starts at `i = 3`.

**Iteration 1: i = 3**

```
current = prev1 + prev2;
current = 2 + 1;
current = 3;
```

Now update:

```
prev2 = prev1;  // prev2 = 2
prev1 = current; // prev1 = 3
```

**Iteration 2: i = 4**

Current values:

```
prev2 = 2
prev1 = 3
```

Calculate:

```
current = 3 + 2;
current = 5;
```

Update:

```
prev2 = 3
prev1 = 5
```

**Iteration 3: i = 5**

Current values:

```
prev2 = 3
prev1 = 5
```

Calculate:

```
current = 5 + 3;
current = 8;
```

Update:

```
prev2 = 5
prev1 = 8
```

### **Complete iteration table**

| i | prev2 before | prev1 before | current | prev2 after | prev1 after |
| - | ------------ | ------------ | ------- | ----------- | ----------- |
| 3 | 1            | 2            | 3       | 2           | 3           |
| 4 | 2            | 3            | 5       | 3           | 5           |
| 5 | 3            | 5            | 8       | 5           | 8           |

Finally:

```
return prev1;
```

Answer:

# **8**

## **Step 8. Important things to remember**

### **1. Why do we initialize&#x20;**`prev2 = 1`**&#x20;and&#x20;**`prev1 = 2`**?**

Because:

- `prev2` represents ways to reach stair 1.
- `prev1` represents ways to reach stair 2.

These are our two base cases.

### **2. Why do we update&#x20;**`prev2`**&#x20;before&#x20;**`prev1`**?**

Because we need to shift the previous two values forward.

```
prev2 = prev1;
prev1 = current;
```

If you update `prev1` first, you lose its old value, which is needed to update `prev2`.

### **3. Why does the loop start from 3?**

Because the answers for stairs 1 and 2 are already known.

We only calculate the remaining stairs.

### **4. Why return&#x20;**`prev1`**?**

After the final iteration, `prev1` contains the number of ways to reach stair `n`.

## **GitHub-ready notes**

**LC 70 — Climbing Stairs**

- **Pattern:** Dynamic Programming — Fibonacci
- **Problem:** Count the distinct ways to reach the top using 1-step or 2-step moves.
- **Approach:** Each stair can be reached from the previous stair or the stair before it.
- **Recurrence:** `dp[i] = dp[i-1] + dp[i-2]`
- **Base cases:** `dp[1] = 1`, `dp[2] = 2`
- **Optimal complexity:** O(n) time, O(1) space.
- **Key insight:** Only the previous two results are needed to calculate the current result.

## **Test your understanding**

Try these without looking at the solution.

**Q1. If n = 3, how many distinct ways are there?**

2

3

4

**Q2. What is the recurrence relation?**

dp[i] = dp[i-1] + dp[i-2]

dp[i] = dp[i-1] \* dp[i-2]

dp[i] = dp[i-1] + 1

**Q3. Why can we optimize the space to O(1)?**

We don't need a loop

Only the previous two results are needed

The answer is always less than 10

Check answers

**Your next practice problem:** LC 746 — Min Cost Climbing Stairs.

It uses almost the same DP thinking, but instead of counting the number of ways, you calculate the minimum cost.
