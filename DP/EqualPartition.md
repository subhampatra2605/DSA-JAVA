# **LeetCode 416 — Partition Equal Subset Sum**

Medium

Dynamic Programming

0/1 Knapsack

This is one of the most important DP problems for learning **Subset Sum and the Take / Not-Take pattern**.

It builds on LC 198, but instead of maximizing money, we are trying to find whether a particular sum is achievable.

Let's derive it from the beginning using your 8-step framework.

## **Step 1. Understand the problem**

You are given an array of positive integers:

```
nums = [1, 5, 11, 5]
```

You need to determine whether the array can be divided into two subsets such that:

**The sum of both subsets is equal.**

We can rearrange the elements, but every element can be used only once.

For example:

**Original array**

# **1**

# **5**

# **11**

# **5**

Partition into two subsets

**Subset 1**

1

5

5

# **11**

**Subset 2**

11

# **11**

Equal partition possible → true

Another example:

```
nums = [1, 2, 3, 5]
```

Total sum:

\\[ 1+2+3+5=11 \\]

Since 11 is odd, it cannot be divided into two equal integer sums.

Answer:

```
false
```

### **What is the actual objective?**

We need to determine whether there is a subset whose sum equals half of the total array sum.

Let:

\\[ S=\sum nums[i] \\]

If `S` is odd:

\\[ return\ false \\]

Otherwise, our target becomes:

\\[ \boxed{target=\frac{S}{2}} \\]

Now the problem becomes:

**Can we select some elements from the array whose sum equals&#x20;**`target`**?**

This is the classic Subset Sum problem.

## **Step 2. Brute-force approach — Recursion**

Now that we know the target is half the total sum, let's think about how to find it.

For every element, we have two choices:

1. **Take:** Include the current element in our subset.
2. **Not Take:** Ignore the current element.

This is exactly the Take / Not-Take pattern from LC 198.

### **Define the recursive function**

Let:

```
solve(index, target)
```

Answer the question:

**Can I form the remaining target sum using elements from&#x20;**`index`**&#x20;onwards?**

For the current element `nums[index]`:

**Choice 1 — Take**

If we take it, the remaining target becomes:

\\[ target-nums[index] \\]

And we move to the next index.

```
solve(index + 1, target - nums[index])
```

**Choice 2 — Not Take**

If we skip it, the target remains unchanged.

```
solve(index + 1, target)
```

If either choice succeeds, the answer is `true`.

Therefore:

\\[ \boxed{ f(i,t)=f(i+1,t-nums[i])\ OR\ f(i+1,t) } \\]

### **Base cases**

There are two important base cases.

**Base case 1: Target becomes zero**

```
if (target == 0) return true;
```

We have successfully formed the required sum.

**Base case 2: We have no elements left**

```
if (index == nums.length) return false;
```

If the target is still not zero, we cannot form it.

### **Brute-force Java code**

```

class Solution {

    public boolean canPartition(int[] nums) {

        int total = 0;

        for (int num : nums) {
            total += num;
        }

        // Odd total cannot be divided equally.
        if (total % 2 != 0) {
            return false;
        }

        int target = total / 2;

        return solve(nums, 0, target);
    }

    private boolean solve(int[] nums, int index, int target) {

        // Successfully formed the target.
        if (target == 0) {
            return true;
        }

        // No elements left, but target is not zero.
        if (index == nums.length) {
            return false;
        }

        // Choice 1: Take the current element.
        // Only take it if it does not exceed target.
        boolean take = false;

        if (nums[index] <= target) {
            take = solve(
                nums,
                index + 1,
                target - nums[index]
            );
        }

        // Choice 2: Do not take the current element.
        boolean notTake = solve(
            nums,
            index + 1,
            target
        );

        // If either choice succeeds, return true.
        return take || notTake;
    }
}

```

### **Complete recursion dry run**

Input:

```
nums = [1, 5, 11, 5]
target = 11
```

Start:

```
solve(nums, 0, 11)
```

At index 0, value = 1.

We have two choices:

```
TAKE:
solve(1, 10)

NOT TAKE:
solve(1, 11)
```

Let's follow the successful path first.

**Successful recursive path**

1. **solve(0, 11)**

   Take 1 → remaining target = 10
2. **solve(1, 10)**

   Take 5 → remaining target = 5
3. **solve(2, 5)**

   Cannot take 11 because 11 is greater than 5. Skip it.
4. **solve(3, 5)**

   Take 5 → remaining target = 0.
5. solve(4, 0) returns true

   Target is zero. We found a valid subset.

The selected subset is:

\\[ 1+5+5=11 \\]

The recursive calls return `true` back to their callers.

Notice that when `solve(2, 5)` tries to take 11, that choice is not allowed, but the not-take choice can still succeed.

**Important:** We stop exploring a branch as soon as it returns `true`, because we only need to know whether a valid subset exists.

### **Complexity**

- Time: \\(O(2^n)\\) — each element can be taken or skipped.
- Space: \\(O(n)\\) — recursion stack.

## **Step 3. Identify the bottleneck**

The recursion can calculate the same subproblem repeatedly.

For example:

```
solve(2, 5)
```

may be reached through different combinations of earlier choices.

But if the `index` and `target` are the same, the answer will always be the same.

We should store the result instead of calculating it repeatedly.

There are two changing variables:

- `index`
- `target`

That means our DP needs two dimensions.

## **Step 4. Recognize the DP pattern**

This is the classic **0/1 Knapsack / Subset Sum** pattern.

Why 0/1?

Because each element has only two possibilities:

- Take it once.
- Don't take it.

We cannot take the same element repeatedly.

Our DP state is:

\\[ dp[i][t] \\]

Meaning:

**Can we form sum&#x20;**`t`**&#x20;using elements from index&#x20;**`i`**&#x20;onwards?**

Our recurrence is:

\\[ dp[i][t]=dp[i+1][t]\ OR\ dp[i+1][t-nums[i]] \\]

The two parts mean:

- `dp[i+1][t]` → not take.
- `dp[i+1][t-nums[i]]` → take.

Base cases:

- Target zero is always achievable: `true`.
- No elements remaining with a positive target: `false`.

## **Step 5. Derive the DP approaches**

### **Approach A — Memoization (Top-down DP)**

We use a 2D array to remember previously calculated states.

```
dp[index][target]
```

We use three values:

- `-1` = not calculated.
- `0` = false.
- `1` = true.

```

class Solution {

    public boolean canPartition(int[] nums) {

        int total = 0;

        for (int num : nums) {
            total += num;
        }

        if (total % 2 != 0) {
            return false;
        }

        int target = total / 2;
        int n = nums.length;

        // -1 = uncalculated, 0 = false, 1 = true.
        int[][] dp = new int[n][target + 1];

        for (int i = 0; i < n; i++) {
            java.util.Arrays.fill(dp[i], -1);
        }

        return solve(nums, 0, target, dp);
    }

    private boolean solve(
        int[] nums, int index, int target, int[][] dp
    ) {

        if (target == 0) {
            return true;
        }

        if (index == nums.length) {
            return false;
        }

        // Reuse a previously calculated state.
        if (dp[index][target] != -1) {
            return dp[index][target] == 1;
        }

        // Choice 1: Take current element.
        boolean take = false;

        if (nums[index] <= target) {
            take = solve(
                nums, index + 1,
                target - nums[index], dp
            );
        }

        // Choice 2: Skip current element.
        boolean notTake = solve(
            nums, index + 1, target, dp
        );

        boolean result = take || notTake;

        // Store the answer for this state.
        dp[index][target] = result ? 1 : 0;

        return result;
    }
}

```

Complexity:

- Time: \\(O(n \times target)\\)
- Space: \\(O(n \times target)\\) for the DP table, plus recursion stack.

### **Approach B — Tabulation (Bottom-up DP)**

Now let's remove recursion and build the answers iteratively.

We define:

\\[ dp[i][t]= \begin{cases} true & \text{if sum }t\text{ can be formed using first }i\text{ elements}\\\ false & \text{otherwise} \end{cases} \\]

This is a slightly different but equivalent state definition from memoization.

For this version:

- Rows represent how many elements we've considered.
- Columns represent the target sums.

Let's use:

```
nums = [1, 5, 11, 5]
target = 11
```

Our DP table has 5 rows (including the empty prefix) and 12 columns (sums 0 to 11).

**Base cases:**

1\. Sum 0 is always possible, even with no elements.

```
dp[i][0] = true;
```

2\. With zero elements, a positive sum is impossible.

```
dp[0][t] = false; // t > 0
```

**Transition:**

If the current element is `nums[i - 1]`:

- Not take: `dp[i - 1][t]`
- Take: `dp[i - 1][t - nums[i - 1]]`

Therefore:

\\[ \boxed{ dp[i][t]=dp[i-1][t]\ OR\ dp[i-1][t-nums[i-1]] } \\]

The take option is available only when `nums[i - 1] <= t`.

### **Java code**

```

class Solution {

    public boolean canPartition(int[] nums) {

        int total = 0;

        for (int num : nums) {
            total += num;
        }

        if (total % 2 != 0) {
            return false;
        }

        int target = total / 2;
        int n = nums.length;

        boolean[][] dp = new boolean[n + 1][target + 1];

        // Sum 0 is possible with any number of elements.
        for (int i = 0; i <= n; i++) {
            dp[i][0] = true;
        }

        // Build the table using each element once.
        for (int i = 1; i <= n; i++) {

            for (int t = 1; t <= target; t++) {

                // Choice 1: Do not take current element.
                boolean notTake = dp[i - 1][t];

                // Choice 2: Take current element if possible.
                boolean take = false;

                if (nums[i - 1] <= t) {
                    take = dp[i - 1][t - nums[i - 1]];
                }

                dp[i][t] = take || notTake;
            }
        }

        return dp[n][target];
    }
}

```

Complexity:

- Time: \\(O(n \times target)\\)
- Space: \\(O(n \times target)\\)

## **Step 6. Optimal approach — Space optimization**

Now comes the most important part of this problem.

Look at the tabulation transition:

\\[ dp[i][t]=dp[i-1][t]\ OR\ dp[i-1][t-nums[i-1]] \\]

Notice that we only use the **previous row** to calculate the current row.

Do we really need a 2D array?

No. We can use a single 1D array.

```
boolean[] dp = new boolean[target + 1];
```

Here:

```
dp[t]
```

means:

**Can we form sum&#x20;**`t`**&#x20;using the elements processed so far?**

Initially:

```
dp[0] = true;
```

Because we can always form sum zero by choosing no elements.

### **The most important trick: Iterate backwards**

For each number, we update the possible sums:

```
for (int t = target; t >= num; t--) {
    dp[t] = dp[t] || dp[t - num];
}
```

Why do we iterate backwards instead of forwards?

Because each element can only be used once.

Suppose:

```
nums = [2]
target = 4
```

If we iterate forwards:

- `dp[2]` becomes true using the number 2.
- Then `dp[4]` may become true using the newly updated `dp[2]`.

This incorrectly uses the same element twice: `2 + 2`.

But if we iterate backwards:

- We check sum 4 first, using the old `dp[2]`, which is false.
- Then we update sum 2 to true.

So sum 4 remains false.

## **Why backward iteration matters**

One element: nums = [2], target = 4

**Forward iteration — incorrect for 0/1 Knapsack**

0

T

1

F

2

T

3

F

4

T

It can incorrectly mark 4 as achievable by reusing the same 2.

**Backward iteration — correct**

0

T

1

F

2

T

3

F

4

F

Only sum 2 becomes achievable. The same element cannot be reused.

### **Optimal Java code**

Recommended solution

```

class Solution {

    public boolean canPartition(int[] nums) {

        int total = 0;

        // Calculate the total sum.
        for (int num : nums) {
            total += num;
        }

        // Odd sum cannot be split equally.
        if (total % 2 != 0) {
            return false;
        }

        int target = total / 2;

        // dp[t] tells whether sum t is achievable.
        boolean[] dp = new boolean[target + 1];

        // Sum zero is always achievable.
        dp[0] = true;

        // Process every element exactly once.
        for (int num : nums) {

            // Traverse backwards to avoid reusing
            // the current element more than once.
            for (int t = target; t >= num; t--) {

                // Either sum t was already possible,
                // or we can form it by taking num.
                dp[t] = dp[t] || dp[t - num];
            }
        }

        return dp[target];
    }
}

```

Complexity:

| Metric          | Complexity               |
| --------------- | ------------------------ |
| Time            | \\(O(n \times target)\\) |
| Auxiliary space | \\(O(target)\\)          |

Since `target = total / 2`, the time complexity can also be written as \\(O(n \times S)\\), where \\(S\\) is the total sum.

## **Step 7. Complete dry run of the optimal code**

Input:

```
nums = [1, 5, 11, 5]
```

First calculate the total:

\\[ 1+5+11+5=22 \\]

Since 22 is even:

\\[ target=22/2=11 \\]

We need to find whether we can form sum 11.

Initialize:

```
boolean[] dp = new boolean[12];
dp[0] = true;
```

Initially:

**Initial DP state**

0

**T**

1

**F**

2

**F**

3

**F**

4

**F**

5

**F**

6

**F**

7

**F**

8

**F**

9

**F**

10

**F**

11

**F**

T = achievable, F = not achievable

### **Iteration 1: num = 1**

Our loop:

```
for (int t = 11; t >= 1; t--)
```

We check whether each sum can be formed using the current element 1.

At `t = 1`:

```
dp[1] = dp[1] || dp[1 - 1];
dp[1] = false || dp[0];
dp[1] = true;
```

Why? Because `dp[0]` is true, and adding 1 gives us sum 1.

Reachable sums:

```
{0, 1}
```

### **Iteration 2: num = 5**

Now we process 5.

The loop runs:

```
for (int t = 11; t >= 5; t--)
```

Let's examine the important updates:

**At t = 6:**

```
dp[6] = dp[6] || dp[6 - 5];
dp[6] = false || dp[1];
dp[6] = true;
```

We can form 6 using:

\\[ 1+5=6 \\]

**At t = 5:**

```
dp[5] = dp[5] || dp[0];
dp[5] = true;
```

We can form 5 directly.

Reachable sums:

```
{0, 1, 5, 6}
```

### **Iteration 3: num = 11**

Now we process 11.

The loop runs:

```
for (int t = 11; t >= 11; t--)
```

Only one iteration is possible.

```
dp[11] = dp[11] || dp[11 - 11];

dp[11] = false || dp[0];

dp[11] = true;
```

We can form 11 using the element 11 itself.

Reachable sums:

```
{0, 1, 5, 6, 11}
```

### **Iteration 4: num = 5**

Now we process the final 5.

The loop runs from 11 down to 5.

Some sums were already achievable. This element may also create new sums, but `dp[11]` is already true.

Reachable sums now include:

```
{0, 1, 5, 6, 10, 11}
```

The important thing:

```
dp[11] == true
```

Therefore:

```
return true;
```

Final answer: true

### **Complete dry-run summary**

| Processed elements | Newly achievable sums | Is target 11 achievable? |
| ------------------ | --------------------- | ------------------------ |
| None               | `{0}`                 | No                       |
| `[1]`              | `{0,1}`               | No                       |
| `[1,5]`            | `{0,1,5,6}`           | No                       |
| `[1,5,11]`         | `{0,1,5,6,11}`        | Yes                      |
| `[1,5,11,5]`       | Also 10               | Yes                      |

Our subset is:

```
[11]
```

The remaining subset is:

```
[1, 5, 5]
```

Both have sum 11.

## **Step 8. Tricky parts and key insights**

### **1. Why do we check whether total sum is odd?**

Because two equal integer subset sums must add up to an even total.

For example:

```
nums = [1, 2, 4]
```

Total = 7.

We cannot divide 7 into two equal integer sums.

So immediately return `false`.

### **2. Why do we initialize&#x20;**`dp[0] = true`**?**

Because we can always make sum zero by selecting no elements.

This is the foundation for building all other achievable sums.

### **3. Why do we iterate backwards?**

Because this is a **0/1 Knapsack** problem.

Each element must be used at most once.

Backward iteration ensures that an element cannot use a state it has just created during the same iteration.

### **4. Why do we use OR instead of Math.max or Math.min?**

We are not maximizing money or minimizing cost.

We are answering a yes/no question:

- Can we form the target without the current element?
- Can we form the target using the current element?

If either is possible, the result is true.

### **5. What if the array contains one element?**

```
nums = [2]
```

Total = 2, target = 1.

We cannot form 1, so the answer is false.

### **6. What is the connection with previous DP problems?**

| Problem | DP pattern        | Decision                    |
| ------- | ----------------- | --------------------------- |
| LC 70   | Fibonacci DP      | Count ways                  |
| LC 746  | Minimum path cost | Choose minimum              |
| LC 198  | Take / Not-Take   | Choose maximum              |
| LC 416  | 0/1 Knapsack      | Check if target is possible |

## **GitHub-ready notes — DSA-Java**

**LC 416 — Partition Equal Subset Sum**

- **Pattern:** 0/1 Knapsack — Subset Sum
- **Problem:** Determine whether an array can be divided into two subsets with equal sums.
- **Observation:** If total sum is odd, return false.
- **Target:** `total / 2`
- **State:** `dp[t]` = whether sum `t` can be formed using processed elements.
- **Initialization:** `dp[0] = true`.
- **Transition:** `dp[t] = dp[t] || dp[t - num]`.
- **Iteration:** Process target sums in descending order.
- **Time:** O(n × target).
- **Space:** O(target).
- **Key insight:** Equal partition is equivalent to finding a subset whose sum equals half the total. Backward iteration ensures each element is used at most once.

## **Practice — Test your understanding**

These questions are important because I want you to understand the reasoning behind the 0/1 Knapsack pattern, not just memorize the code.

**Q1. nums = [1, 2, 3, 5]. What should the algorithm return?**

true

false

**Q2. What does dp[7] = true mean?**

The array contains 7

A subset with sum 7 can be formed

The total sum is 7

**Q3. Why do we iterate from target down to num?**

To sort the array

To ensure an element is used at most once

To find the smallest element

**Q4. For nums = [1, 5, 11, 5], what is the target?**

22

5

11

Check answers

**Next recommended problem: LC 494 — Target Sum.**

It builds directly on LC 416. You'll learn how to transform a problem involving `+` and `-` choices into a subset-sum counting problem.
