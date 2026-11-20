# **LeetCode 746 — Min Cost Climbing Stairs**

Easy

Dynamic Programming

Fibonacci Variation

**Problem:** LeetCode 746 — Min Cost Climbing Stairs

This problem is closely related to LC 70 (Climbing Stairs), but there is one important difference:

| LC 70                                | LC 746                         |
| ------------------------------------ | ------------------------------ |
| Count the total number of ways       | Find the minimum cost          |
| `dp[i] = dp[i-1] + dp[i-2]`          | `dp[i] = min(...)`             |
| Every path contributes to the answer | Only the cheapest path matters |

We'll use your 8-step DSA framework, starting with the problem itself.

## **Step 1. Understand the problem**

You are given an array `cost[]`, where `cost[i]` is the amount you must pay when you step on stair `i`.

Rules:

- You can climb either 1 or 2 steps at a time.
- You can start at index `0` or index `1` for free, but you must pay the cost of any stair you step on, including your starting stair.
- Your goal is to reach the **top**, which is one position beyond the last array index.
- Find the minimum total cost required.

### **Example 1**

```
cost = [10, 15, 20]
```

Staircase visualization

One optimal route: start at stair 1 (pay 15), then jump 2 steps to the top.

Possible paths:

| Path                    | Total cost   |
| ----------------------- | ------------ |
| Stair 0 → Stair 1 → Top | 10 + 15 = 25 |
| Stair 0 → Stair 2 → Top | 10 + 20 = 30 |
| Stair 1 → Top           | 15           |

Minimum cost = **15**.

### **Example 2**

```
cost = [1, 100, 1, 1, 1, 100, 1, 1, 100, 1]
```

One minimum-cost path visits indices:

`0 → 2 → 4 → 6 → 7 → 9 → Top`

Total cost:

\\[ 1+1+1+1+1+1=\boxed{6} \\]

Notice that we don't have to visit every stair.

## **Step 2. Brute-force approach — Recursion**

First, ask:

**If I am standing at stair&#x20;**`i`**, what choices do I have?**

I can either:

1. Jump to `i + 1`.
2. Jump to `i + 2`.

Since I want the minimum cost, I should choose whichever path has the smaller total cost.

Define:

`solve(i)` = minimum cost to reach the top starting from stair `i`, including the cost of stair `i`.

The recurrence is:

\\[ solve(i)=cost[i]+\min(solve(i+1),solve(i+2)) \\]

Base case:

\\[ solve(i)=0 \quad \text{when } i\ge n \\]

Why zero? Because we have already reached or passed the top, and there are no more stairs to pay for.

We can start at stair 0 or stair 1:

\\[ answer=\min(solve(0),solve(1)) \\]

### **Brute-force Java code**

```

class Solution {

    public int minCostClimbingStairs(int[] cost) {

        // We may start from stair 0 or stair 1.
        return Math.min(
            solve(cost, 0),
            solve(cost, 1)
        );
    }

    private int solve(int[] cost, int i) {

        // Reached or passed the top.
        // No additional cost is required.
        if (i >= cost.length) {
            return 0;
        }

        // Minimum cost if we jump one step.
        int oneStep = solve(cost, i + 1);

        // Minimum cost if we jump two steps.
        int twoSteps = solve(cost, i + 2);

        // Pay for the current stair and choose
        // the cheaper of the two future paths.
        return cost[i] + Math.min(oneStep, twoSteps);
    }
}

```

### **Complete recursion dry run**

Input:

```
cost = [10, 15, 20]
```

First:

```
minCostClimbingStairs(cost)
```

Calls:

```
Math.min(solve(0), solve(1))
```

Let's evaluate `solve(0)` first.

Recursive calls for solve(0)

Execution proceeds as follows:

**Call 1:&#x20;**`solve(0)`

The function pauses at:

```
int oneStep = solve(cost, 1);
```

**Call 2:&#x20;**`solve(1)`

It pauses at:

```
int oneStep = solve(cost, 2);
```

**Call 3:&#x20;**`solve(2)`

It calls `solve(3)`.

**Call 4:&#x20;**`solve(3)`

Since `3 >= cost.length`, it returns `0`.

Back at `solve(2)`, we now have:

```
oneStep = 0;
```

Next, it calls `solve(4)`, which also returns `0`.

So:

```
solve(2) = 20 + Math.min(0, 0);
solve(2) = 20;
```

Back at `solve(1)`:

```
oneStep = 20;
```

It next calls `solve(3)`, which returns `0`.

Therefore:

```
solve(1) = 15 + Math.min(20, 0);
solve(1) = 15;
```

Back at `solve(0)`:

```
oneStep = 15;
```

It now calls `solve(2)` again, obtaining `20`.

Therefore:

```
solve(0) = 10 + Math.min(15, 20);
solve(0) = 25;
```

Finally, `minCostClimbingStairs` evaluates its second starting choice, `solve(1)`, again and obtains `15`.

```
return Math.min(25, 15);
```

Answer = 15

### **Brute-force complexity**

- **Time:** \\(O(2^n)\\) — overlapping recursive calculations.
- **Space:** \\(O(n)\\) — recursion stack.

## **Step 3. Identify the bottleneck**

Look at the recursion.

`solve(2)` gets calculated multiple times. The same is true for other states as `n` increases.

This is the same problem we encountered in LC 70.

Instead of repeatedly solving the same subproblem, we should save its result.

That leads us to Dynamic Programming.

## **Step 4. Recognize the DP pattern**

We can use a slightly different state definition that makes the bottom-up approach easier.

Let:

`dp[i]`**&#x20;= minimum total cost required to reach position&#x20;**`i`**, before paying any cost at position&#x20;**`i`**.**

The top is position `n`, which is beyond the last stair.

How can we reach position `i`?

- From stair `i - 1`, paying `cost[i - 1]`.
- From stair `i - 2`, paying `cost[i - 2]`.

Therefore:

\\[ \boxed{dp[i]=\min(dp[i-1]+cost[i-1],\ dp[i-2]+cost[i-2])} \\]

Base cases:

\\[ dp[0]=0,\qquad dp[1]=0 \\]

Why both zero?

Because we can start at stair 0 or stair 1 without paying anything before stepping onto the chosen starting stair. The stair's cost is included when we move away from it.

**Important:** This is a different state definition from our recursive `solve(i)`, but both correctly solve the same problem.

## **Step 5. Derive the DP approaches**

### **Approach A — Memoization**

We can optimize the original recursion by storing `solve(i)` in a DP array.

```

import java.util.Arrays;

class Solution {

    public int minCostClimbingStairs(int[] cost) {

        int[] dp = new int[cost.length];

        // -1 means the result has not been calculated.
        Arrays.fill(dp, -1);

        return Math.min(
            solve(cost, 0, dp),
            solve(cost, 1, dp)
        );
    }

    private int solve(int[] cost, int i, int[] dp) {

        // Reached the top.
        if (i >= cost.length) {
            return 0;
        }

        // Reuse an already computed answer.
        if (dp[i] != -1) {
            return dp[i];
        }

        int oneStep = solve(cost, i + 1, dp);
        int twoSteps = solve(cost, i + 2, dp);

        // Save the result for future calls.
        dp[i] = cost[i] + Math.min(oneStep, twoSteps);

        return dp[i];
    }
}

```

Time: \\(O(n)\\)

Space: \\(O(n)\\), including the DP array and recursion stack.

### **Approach B — Tabulation**

Let's use:

```
cost = [10, 15, 20]
```

Initialize:

```
dp[0] = 0;
dp[1] = 0;
```

Calculate:

| Position | Calculation       | Minimum cost |
| -------- | ----------------- | ------------ |
| 0        | Starting position | 0            |
| 1        | Starting position | 0            |
| 2        | min(0+15, 0+10)   | 10           |
| 3 (Top)  | min(10+20, 0+15)  | 15           |

So `dp[3] = 15`.

```

class Solution {
    public int minCostClimbingStairs(int[] cost) {

        int n = cost.length;

        // dp[i] = minimum cost to reach position i.
        int[] dp = new int[n + 1];

        // We can start at index 0 or 1.
        dp[0] = 0;
        dp[1] = 0;

        for (int i = 2; i <= n; i++) {

            // Reach i from the previous stair.
            int oneStep = dp[i - 1] + cost[i - 1];

            // Reach i from two stairs below.
            int twoSteps = dp[i - 2] + cost[i - 2];

            // Choose the cheaper path.
            dp[i] = Math.min(oneStep, twoSteps);
        }

        return dp[n];
    }
}

```

Time: \\(O(n)\\)

Space: \\(O(n)\\)

## **Step 6. Optimal Java solution — Space optimization**

Look at our recurrence:

\\[ dp[i]=\min(dp[i-1]+cost[i-1],dp[i-2]+cost[i-2]) \\]

We only need two previous DP values.

So why store the entire array?

We can use:

- `prev2` = minimum cost to reach position `i - 2`.
- `prev1` = minimum cost to reach position `i - 1`.
- `current` = minimum cost to reach position `i`.

```

class Solution {
    public int minCostClimbingStairs(int[] cost) {

        int n = cost.length;

        // dp[0] = 0 and dp[1] = 0.
        // Both are valid starting positions.
        int prev2 = 0;
        int prev1 = 0;

        // Compute the minimum cost to reach
        // every position from 2 through the top (n).
        for (int i = 2; i <= n; i++) {

            // Reach i by taking one step from i-1.
            int oneStep = prev1 + cost[i - 1];

            // Reach i by taking two steps from i-2.
            int twoSteps = prev2 + cost[i - 2];

            // Choose the cheaper of the two paths.
            int current = Math.min(oneStep, twoSteps);

            // Shift the DP values forward.
            prev2 = prev1;
            prev1 = current;
        }

        // prev1 now represents dp[n].
        return prev1;
    }
}

```

**Time complexity**

O(n)

**Auxiliary space**

O(1)

## **Step 7. Complete dry run of optimal code**

Input:

```
cost = [10, 15, 20]
```

Initial state:

```
n = 3;
prev2 = 0;
prev1 = 0;
```

The loop runs from `i = 2` to `i = 3`.

### **Iteration 1:&#x20;**`i = 2`

Calculate:

```
oneStep = prev1 + cost[1];
        = 0 + 15;
        = 15;

twoSteps = prev2 + cost[0];
         = 0 + 10;
         = 10;
```

Choose the minimum:

```
current = Math.min(15, 10);
current = 10;
```

Update:

```
prev2 = prev1;   // 0
prev1 = current; // 10
```

### **Iteration 2:&#x20;**`i = 3`

Current values:

```
prev2 = 0;
prev1 = 10;
```

Calculate:

```
oneStep = prev1 + cost[2];
        = 10 + 20;
        = 30;

twoSteps = prev2 + cost[1];
         = 0 + 15;
         = 15;
```

Choose:

```
current = Math.min(30, 15);
current = 15;
```

Update:

```
prev2 = prev1;   // 10
prev1 = current; // 15
```

The loop ends because `i` becomes 4, which is greater than `n = 3`.

### **Complete iteration table**

| i | prev2 before | prev1 before | oneStep | twoSteps | current |
| - | ------------ | ------------ | ------- | -------- | ------- |
| 2 | 0            | 0            | 15      | 10       | 10      |
| 3 | 0            | 10           | 30      | 15       | 15      |

Finally:

```
return prev1; // 15
```

Final answer: 15

## **Step 8. Tricky parts and key insights**

### **1. Why does the loop start at&#x20;**`i = 2`**?**

Because the minimum cost to reach positions 0 and 1 is already known:

```
dp[0] = 0;
dp[1] = 0;
```

We calculate from position 2 onward.

### **2. Why do we access&#x20;**`cost[i - 1]`**&#x20;and&#x20;**`cost[i - 2]`**?**

Because to reach position `i`, we must jump from either of those two stairs.

We pay for the stair we jump **from**, and its cost hasn't yet been included in the DP value for reaching that position.

### **3. Why do we return&#x20;**`prev1`**?**

After the final iteration, `prev1` contains `dp[n]`, the minimum cost to reach the top.

### **4. Why not use a greedy approach?**

Suppose:

```
cost = [1, 2, 100, 1]
```

Choosing the cheapest immediate stair does not necessarily produce the cheapest complete path.

Dynamic Programming compares the total costs of the possible paths.

### **5. Why is the answer not necessarily the sum of all costs?**

Because we can jump over stairs without paying for them.

## **GitHub-ready notes — DSA-Java**

**LC 746 — Min Cost Climbing Stairs**

**Pattern:** 1D Dynamic Programming / Fibonacci variation

**Problem:** Given an array of stair costs, find the minimum cost to reach the top by moving one or two steps at a time.

**Approach:** The minimum cost to reach position `i` comes from either position `i-1` or `i-2`, including the cost of the stair used to make the jump.

**State:**

`dp[i]` = minimum cost to reach position `i`.

**Recurrence:**

`dp[i] = min(dp[i-1] + cost[i-1], dp[i-2] + cost[i-2])`

**Base cases:**

`dp[0] = 0`, `dp[1] = 0`

**Time:** O(n)

**Space:** O(1) with two variables.

**Key insight:** Unlike LC 70, where we add the number of ways, here we choose the minimum total cost between two possible previous states.

## **Practice: Check your understanding**

**Q1. For cost = [10, 15, 20], what is the minimum cost to reach the top?**

10

15

25

**Q2. What does dp[i] represent in our tabulation solution?**

Cost of stair i

Minimum total cost to reach position i

Number of ways to reach i

**Q3. Why do we use Math.min() instead of addition as in LC 70?**

Because we need the cheapest path

Because there is only one path

Because every stair has the same cost

Check answers

**Next recommended problem:** LC 198 — House Robber. It uses the same idea of making a choice between two possibilities, but introduces the classic **take / not-take DP pattern**, which is especially useful for coding assessments.
