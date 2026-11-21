# **LeetCode 518 — Coin Change II**



Medium

Unbounded Knapsack · DP · Counting Combinations



This is an important next step after **LC 416 (Partition Equal Subset Sum)** and **LC 494 (Target Sum)**.

The biggest new concept is:

**In LC 518, you can use the same coin denomination unlimited times.**

That changes the DP loop direction:

- 0/1 Knapsack → iterate backwards.
- Unbounded Knapsack → iterate forwards.

Let's understand why, rather than just memorizing it.

## **Step 1 — Understand the problem**

You are given:

- `amount`: the total amount of money you want to make.
- `coins`: different coin denominations.

Return the number of different combinations that make up the amount.

You can use each coin denomination as many times as you want.

**Example:**

```
amount = 5
coins = [1, 2, 5]
```

The valid combinations are:

| Combination         | Total |
| ------------------- | ----- |
| `1 + 1 + 1 + 1 + 1` | 5     |
| `1 + 1 + 1 + 2`     | 5     |
| `1 + 2 + 2`         | 5     |
| `5`                 | 5     |

Answer = 4

### **Important: combinations vs permutations**

These are considered the same combination:

```
1 + 2 + 2
2 + 1 + 2
2 + 2 + 1
```

They use the same coins, just in a different order.

We count that combination only once.

This distinction is extremely important in LC 518.

## **Step 2 — Brute-force recursion**

At every step, we decide whether to:

1. Take the current coin.
2. Skip the current coin and move to the next denomination.

But because coins can be reused, when we take a coin, we stay at the same index.

### **Define the recursive state**

```
countWays(index, remainingAmount)
```

Meaning:

> How many combinations can form `remainingAmount` using coins from `index` onwards?

### **The two choices**

**Choice 1 — Take the current coin**

```
countWays(index, remainingAmount - coins[index])
```

Why do we stay at the same index?

Because we can use the same denomination again.

**Choice 2 — Skip the current coin**

```
countWays(index + 1, remainingAmount)
```

Why move to the next index?

Because we don't want to use this denomination anymore in this branch.

### **Base cases**

```
if (remainingAmount == 0) return 1;
if (index == coins.length) return 0;
```

- If remaining amount becomes zero, we found one valid combination.
- If all denominations are exhausted and the amount is still positive, there is no valid combination.

### **Java — brute force**

```
class Solution {
    public int change(int amount, int[] coins) {
        return solve(0, amount, coins);
    }

    private int solve(int index, int remaining, int[] coins) {

        // We have formed the required amount.
        // This complete combination contributes one way.
        if (remaining == 0) {
            return 1;
        }

        // No coins are left, or the remaining amount is invalid.
        if (index == coins.length || remaining < 0) {
            return 0;
        }

        // Choice 1: Take the current coin.
        // Stay at the same index because this coin can be reused.
        int take = solve(index, remaining - coins[index], coins);

        // Choice 2: Skip the current coin.
        // Move to the next denomination.
        int skip = solve(index + 1, remaining, coins);

        // Count combinations from both choices.
        return take + skip;
    }
}
```

### **Dry run:&#x20;**`amount = 3`**,&#x20;**`coins = [1, 2]`

```
solve(0, 3)
       /       \
   Take 1     Skip 1
     /           \
solve(0, 2)    solve(1, 3)
```

Follow the left branch:

```
solve(0, 3)
  Take 1 → solve(0, 2)
    Take 1 → solve(0, 1)
      Take 1 → solve(0, 0) → returns 1
      Skip 1 → solve(1, 1)
        Take 2 → remaining -1 → returns 0
        Skip 2 → solve(2, 1) → returns 0
      solve(0, 1) returns 1

    Skip 1 → solve(1, 2)
      Take 2 → solve(1, 0) → returns 1
      Skip 2 → solve(2, 2) → returns 0
    solve(0, 2) returns 2

  Skip 1 → solve(1, 3)
    Take 2 → solve(1, 1)
      Take 2 → remaining -1 → returns 0
      Skip 2 → solve(2, 1) → returns 0
    Skip 2 → solve(2, 3) → returns 0
    solve(1, 3) returns 0

Final answer = 2
```

The two combinations are:

```
1 + 1 + 1
1 + 2
```

**Complexity:** The brute-force recursion can take exponential time because it explores many possible choices. Its maximum recursion depth is \\(O(amount)\\) for positive coin values.

## **Step 3 — Identify the bottleneck**

Consider:

```
amount = 5
coins = [1, 2, 5]
```

Different recursive paths can reach the same state, such as:

```
solve(index = 1, remaining = 3)
```

Whenever the same state is reached again, we calculate its answer repeatedly.

We can avoid this with memoization.

## **Step 4 — Memoization (Top-Down DP)**

Our state is:

```
dp[index][remaining]
```

It stores the number of combinations to form `remaining` using denominations from `index` onwards.

We initialize the table with `-1`, meaning not calculated yet.

### **Java — memoization**

```
import java.util.Arrays;

class Solution {
    public int change(int amount, int[] coins) {

        int[][] dp = new int[coins.length][amount + 1];

        // -1 means this state has not been calculated.
        for (int[] row : dp) {
            Arrays.fill(row, -1);
        }

        return solve(0, amount, coins, dp);
    }

    private int solve(int index, int remaining, int[] coins, int[][] dp) {

        if (remaining == 0) {
            return 1;
        }

        if (index == coins.length || remaining < 0) {
            return 0;
        }

        // Return the stored answer if this state was solved before.
        if (dp[index][remaining] != -1) {
            return dp[index][remaining];
        }

        // Take: stay at the same index because coins are reusable.
        int take = solve(
            index, remaining - coins[index], coins, dp
        );

        // Skip: move to the next denomination.
        int skip = solve(
            index + 1, remaining, coins, dp
        );

        // Store the total combinations from this state.
        dp[index][remaining] = take + skip;

        return dp[index][remaining];
    }
}
```

**Complexity:**

- Time: \\(O(n \times amount)\\)
- Space: \\(O(n \times amount)\\) for memoization, plus recursion stack.

Here, \\(n\\) is the number of coin denominations.

## **Step 5 — Tabulation (Bottom-Up DP)**

Now let's remove recursion.

### **5.1 Define the 2D DP state**

Let:

```
dp[i][a]
```

mean:

> Number of combinations to make amount `a` using the first `i` coin denominations.

For example:

```
coins = [1, 2]

dp[2][3]
```

means the number of combinations to form 3 using denominations 1 and 2.

### **5.2 Base cases**

There is exactly one way to make amount zero:

```
Choose no coins.
```

Therefore:

```
dp[i][0] = 1;
```

For zero denominations and a positive amount, there are no combinations:

```
dp[0][a] = 0;
```

### **5.3 Transition**

For each coin denomination, we have two choices.

**Choice 1: Don't take the current coin**

```
dp[i - 1][a]
```

We use the previous denominations only.

**Choice 2: Take the current coin**

```
dp[i][a - coins[i - 1]]
```

Notice that we stay in row `i`, not `i - 1`.

Why?

Because we can take the same denomination again.

Therefore:

\\[ \boxed{dp[i][a]=dp[i-1][a]+dp[i][a-coins[i-1]]} \\]

The second term is included only when `coins[i - 1] <= a`.

### **Java — 2D tabulation**

```
class Solution {
    public int change(int amount, int[] coins) {

        int n = coins.length;

        int[][] dp = new int[n + 1][amount + 1];

        // One way to form amount 0: choose no coins.
        for (int i = 0; i <= n; i++) {
            dp[i][0] = 1;
        }

        // Consider each coin denomination.
        for (int i = 1; i <= n; i++) {

            int coin = coins[i - 1];

            // Calculate combinations for every possible amount.
            for (int a = 1; a <= amount; a++) {

                // Don't take the current coin.
                dp[i][a] = dp[i - 1][a];

                // Take the current coin if it fits.
                // Stay in the same row because reuse is allowed.
                if (coin <= a) {
                    dp[i][a] += dp[i][a - coin];
                }
            }
        }

        return dp[n][amount];
    }
}
```

**Complexity:**

- Time: \\(O(n \times amount)\\)
- Space: \\(O(n \times amount)\\)

## **Step 6 — Space optimization (1D DP)**

We can reduce the 2D table to one dimension.

Remember our 2D transition:

\\[ dp[i][a]=dp[i-1][a]+dp[i][a-coin] \\]

We only need:

- The previous row's value at the same amount.
- The current row's value at a smaller amount.

We can maintain both in a single array by processing coins one at a time and amounts in ascending order.

### **The optimal code**

Recommended for LeetCode

```
class Solution {
    public int change(int amount, int[] coins) {

        // dp[a] = number of combinations to make amount a.
        int[] dp = new int[amount + 1];

        // One way to make amount 0: choose no coins.
        dp[0] = 1;

        // Process one denomination at a time.
        for (int coin : coins) {

            // Iterate forwards because each coin can be reused.
            for (int a = coin; a <= amount; a++) {

                // Existing combinations + combinations using this coin.
                dp[a] += dp[a - coin];
            }
        }

        return dp[amount];
    }
}
```

**Complexity:**

- Time: \\(O(n \times amount)\\)
- Space: \\(O(amount)\\)

## **Step 7 — Complete dry run**

Let's dry-run the optimal solution using:

```
amount = 5
coins = [1, 2, 5]
```

### **7.1 Initialization**

```
int[] dp = new int[6];
dp[0] = 1;
```

Index = amount; value = number of combinations.

| Amount     | 0 | 1 | 2 | 3 | 4 | 5 |
| ---------- | - | - | - | - | - | - |
| Initial DP | 1 | 0 | 0 | 0 | 0 | 0 |

### **7.2 Process coin = 1**

The inner loop:

```
for (int a = 1; a <= 5; a++)
```

| Current amount | Calculation              | Updated value |
| -------------- | ------------------------ | ------------- |
| 1              | `dp[1] += dp[0]` → 0 + 1 | 1             |
| 2              | `dp[2] += dp[1]` → 0 + 1 | 1             |
| 3              | `dp[3] += dp[2]` → 0 + 1 | 1             |
| 4              | `dp[4] += dp[3]` → 0 + 1 | 1             |
| 5              | `dp[5] += dp[4]` → 0 + 1 | 1             |

DP becomes:

```
[1, 1, 1, 1, 1, 1]
```

We can form every amount from 0 to 5 using only coin 1.

### **7.3 Process coin = 2**

Now:

```
for (int a = 2; a <= 5; a++)
```

| Current amount | Calculation              | Updated value |
| -------------- | ------------------------ | ------------- |
| 2              | `dp[2] += dp[0]` → 1 + 1 | 2             |
| 3              | `dp[3] += dp[1]` → 1 + 1 | 2             |
| 4              | `dp[4] += dp[2]` → 1 + 2 | 3             |
| 5              | `dp[5] += dp[3]` → 1 + 2 | 3             |

DP becomes:

```
[1, 1, 2, 2, 3, 3]
```

Let's understand the important calculation:

At amount 4:

```
dp[4] += dp[2];
```

Before this update:

```
dp[4] = 1
dp[2] = 2
```

Why are there two ways to form 2?

```
2
1 + 1
```

Now add one more coin of denomination 2 to each:

```
2 + 2
1 + 1 + 2
```

We already had one way to make 4 using only coin 1:

```
1 + 1 + 1 + 1
```

So:

```
dp[4] = 1 + 2 = 3
```

### **7.4 Process coin = 5**

Now:

```
for (int a = 5; a <= 5; a++)
```

Only one iteration:

```
dp[5] += dp[0];
```

Before the update:

```
dp[5] = 3
dp[0] = 1
```

After the update:

```
dp[5] = 3 + 1 = 4
```

**Final DP array**

| Amount | 0 | 1 | 2 | 3 | 4 | 5     |
| ------ | - | - | - | - | - | ----- |
| Ways   | 1 | 1 | 2 | 2 | 3 | **4** |

# **4 combinations**

- `1 + 1 + 1 + 1 + 1`
- `1 + 1 + 1 + 2`
- `1 + 2 + 2`
- `5`

## **Step 8 — The most important concepts and tricky parts**

### **8.1 Why is the coin loop outside?**

Consider these two structures.

**Correct — combinations:**

```
for (int coin : coins) {
    for (int a = coin; a <= amount; a++) {
        dp[a] += dp[a - coin];
    }
}
```

This processes one denomination at a time.

Once we process coin 2, we build combinations using 1 and 2. We never create a separate ordering for `1 + 2` and `2 + 1`.

**Different problem — permutations:**

```
for (int a = 1; a <= amount; a++) {
    for (int coin : coins) {
        if (a >= coin) {
            dp[a] += dp[a - coin];
        }
    }
}
```

Here, we process each amount and consider the last coin used. Different orders can be counted separately.

For example, with `amount = 3`, `coins = [1, 2]`:

| Combination | Permutations     |
| ----------- | ---------------- |
| `1 + 1 + 1` | `1 + 1 + 1`      |
| `1 + 2`     | `1 + 2`, `2 + 1` |

- LC 518 asks for combinations: answer = 2.
- The amount-outer approach counts permutations: answer = 3.

### **8.2 Why forward iteration?**

This is the biggest difference from LC 416 and LC 494.

| Problem                             | Can reuse element? | Inner loop |
| ----------------------------------- | ------------------ | ---------- |
| LC 416 — Partition Equal Subset Sum | No                 | Backward   |
| LC 494 — Target Sum                 | No                 | Backward   |
| LC 518 — Coin Change II             | Yes                | Forward    |

Let's understand with a tiny example:

```
amount = 4
coins = [2]
```

We should be able to form:

```
2 + 2 = 4
```

If we iterate forwards:

- At `a = 2`, `dp[2] += dp[0]` → `dp[2] = 1`.
- At `a = 4`, `dp[4] += dp[2]` → `dp[4] = 1`.

The newly updated `dp[2]` is available when we calculate `dp[4]`. This allows the same coin denomination to be reused.

If we iterated backwards, `dp[2]` would still be zero when we process amount 4. We would fail to count `2 + 2`.

**Remember this rule:**

- Backward = use each element at most once.
- Forward = allow unlimited reuse of the current denomination.

## **GitHub-ready notes — DSA-Java**

```

# 518. Coin Change II

## Problem
Given an amount and coin denominations, count the number of different combinations that make the amount. Each denomination can be used unlimited times.

## Approach
Unbounded Knapsack — Count Combinations.

Process each coin denomination separately. For every coin, update all amounts from coin to amount.

## Pattern
Unbounded Knapsack / Coin Change / Combination Counting

## DP State
dp[a] = number of combinations to form amount a using the processed denominations.

## Initialization
dp[0] = 1

There is exactly one way to form zero: choose no coins.

## Transition
dp[a] += dp[a - coin]

## Loop Direction
- Outer loop: coins
- Inner loop: amount from coin to amount (forward)
- Forward iteration allows unlimited reuse.
- Coin-outer ordering prevents counting different orders as separate combinations.

## Complexity
Time: O(n * amount)
Space: O(amount)

## Key Insight
Unlike 0/1 Knapsack, a denomination can be reused. Therefore, the amount loop runs forwards. Processing coins in the outer loop ensures combinations are counted rather than permutations.

## Java Code

```

## **Compare the three problems you've just learned**

| Feature      | LC 416          | LC 494       | LC 518            |
| ------------ | --------------- | ------------ | ----------------- |
| Problem      | Equal partition | Target signs | Coin combinations |
| DP goal      | Existence       | Count        | Count             |
| Reuse        | No              | No           | Yes               |
| DP operation | Boolean OR      | Addition     | Addition          |
| Inner loop   | Backward        | Backward     | Forward           |
| Outer loop   | Elements        | Elements     | Coins             |

## **Test your understanding**

**Quick quiz**

0/4 answered

**1. amount = 4, coins = [2]. What is the answer?**

0

1

2

**2. What does dp[a] represent?**

Minimum coins needed

Number of combinations to form amount a

Whether amount a is possible

**3. Why is the coin loop outside?**

To sort coins

To count combinations without treating different orders as separate

To avoid using a coin twice

**4. Why does the amount loop go forwards?**

To allow unlimited reuse of a denomination

To avoid duplicate combinations

Because the array is sorted

Check answers

**Next recommended problem: LC 322 — Coin Change.** It uses the same Unbounded Knapsack structure, but instead of counting combinations, we find the minimum number of coins needed.
