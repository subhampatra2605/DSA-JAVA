# LeetCode 303 — Range Sum Query: Immutable

Let's solve LC 303 using your **8-step problem-solving framework**, focusing on how to identify the repeated work, derive the prefix-sum approach, and write the Java code in the correct order.

This is an important problem because **prefix sums appear frequently in coding assessments**, including problems involving range queries and subarray sums.

---

# Step 1: Understand the Problem

You are given an integer array `nums`. You need to implement a class called `NumArray` that supports multiple queries.

Each query provides two indices, `left` and `right`, and asks you to return the sum of all elements between those indices, **including both endpoints**.

The array is immutable, meaning its elements do not change between queries.

### Example

```text
Input:
nums = [-2, 0, 3, -5, 2, -1]

Queries:
sumRange(0, 2)
sumRange(2, 5)
sumRange(0, 5)

Output:
1
-1
-3
```

Let's verify each query.

**Query 1: `sumRange(0, 2)`**

```text
-2 + 0 + 3 = 1
```

**Query 2: `sumRange(2, 5)`**

```text
3 + (-5) + 2 + (-1) = -1
```

**Query 3: `sumRange(0, 5)`**

```text
-2 + 0 + 3 + (-5) + 2 + (-1) = -3
```

### What are we trying to optimize?

The task is not just to calculate one range sum. We need to support **multiple range-sum queries efficiently**.

That's the clue that will help us derive the optimal approach.

---

# Step 2: Solve a Small Example Manually

Consider:

```text
nums = [2, 4, 1, 5, 3]
```

Suppose we receive these queries:

```text
sumRange(1, 3)
sumRange(2, 4)
sumRange(0, 2)
```

Calculate them manually:

| Query | Calculation | Result |
|---|---|---:|
| `sumRange(1, 3)` | `4 + 1 + 5` | 10 |
| `sumRange(2, 4)` | `1 + 5 + 3` | 9 |
| `sumRange(0, 2)` | `2 + 4 + 1` | 7 |

Notice that some elements participate in multiple queries.

For example, the elements at indices `2` and `3` are used in both the first and second queries.

**Ask yourself: Can we calculate some information once and reuse it for every query?**

Yes. We can precompute cumulative sums.

---

# Step 3: Derive the Brute-Force Approach

Before optimizing, let's describe the simplest correct approach.

### Brute-force logic

For every query:

1. Initialize `sum = 0`.
2. Start at index `left`.
3. Continue through index `right`.
4. Add each element to `sum`.
5. Return the sum.

### Java implementation

```java
class NumArray {
    private int[] nums;

    public NumArray(int[] nums) {
        this.nums = nums;
    }

    public int sumRange(int left, int right) {
        int sum = 0;

        for (int i = left; i <= right; i++) {
            sum += nums[i];
        }

        return sum;
    }
}
```

This solution is correct.

However, each query requires us to visit every element in the requested range.

If the array has `n` elements and a query covers most of them, one query can take O(n) time.

If there are `q` queries, the worst-case total time is **O(nq)**.

Let's identify the bottleneck.

---

# Step 4: Dry-Run the Brute Force

Use:

```text
nums = [2, 4, 1, 5, 3]
```

Suppose the query is:

```text
sumRange(1, 3)
```

Initially:

```text
sum = 0
```

| Index `i` | `nums[i]` | Calculation | New sum |
|---:|---:|---|---:|
| 1 | 4 | `0 + 4` | 4 |
| 2 | 1 | `4 + 1` | 5 |
| 3 | 5 | `5 + 5` | 10 |

Return `10`.

Now suppose we receive another query:

```text
sumRange(2, 4)
```

We calculate:

```text
1 + 5 + 3 = 9
```

We have to process elements again, even though some of the work overlaps with the previous query.

**The problem is repeated calculation across queries.**

---

# Step 5: Identify the Bottleneck

Ask three questions.

### Question 1: What work am I repeating?

For every query, we traverse the requested range and calculate its sum from scratch.

For example:

```text
Query A: sumRange(1, 3) → 4 + 1 + 5
Query B: sumRange(2, 4) →     1 + 5 + 3
```

The values `1` and `5` are added again.

This repeated work becomes expensive when the number of queries increases.

### Question 2: What information could I calculate once?

Consider the cumulative sum from index `0` through each index.

For:

```text
nums = [2, 4, 1, 5, 3]
```

The cumulative sums are:

```text
Index 0: 2
Index 1: 2 + 4 = 6
Index 2: 2 + 4 + 1 = 7
Index 3: 2 + 4 + 1 + 5 = 12
Index 4: 2 + 4 + 1 + 5 + 3 = 15
```

So:

```text
prefix = [2, 6, 7, 12, 15]
```

Let's define:

**`prefix[i]` = sum of all elements from index `0` through index `i`.**

Now we have information we can reuse.

### Question 3: Can we calculate a range sum using two cumulative sums?

Suppose we want:

```text
sumRange(1, 3)
```

The desired sum is:

```text
nums[1] + nums[2] + nums[3] = 10
```

But:

```text
prefix[3] = nums[0] + nums[1] + nums[2] + nums[3]
          = 2 + 4 + 1 + 5
          = 12
```

This includes the unwanted element at index `0`.

We can remove it by subtracting:

```text
prefix[0] = 2
```

Therefore:

```text
sumRange(1, 3) = prefix[3] - prefix[0]
               = 12 - 2
               = 10
```

This reveals the key formula.

---

# Step 6: Derive the Optimized Prefix-Sum Algorithm

We need to handle two situations.

### Case 1: The query starts at index `0`

For example:

```text
sumRange(0, 2)
```

The answer is simply:

```text
prefix[2]
```

There is no preceding section to remove.

### Case 2: The query starts after index `0`

For example:

```text
sumRange(2, 4)
```

We know:

```text
prefix[4] = nums[0] + nums[1] + nums[2] + nums[3] + nums[4]
prefix[1] = nums[0] + nums[1]
```

Subtracting removes the elements before the requested range:

```text
prefix[4] - prefix[1] = nums[2] + nums[3] + nums[4]
```

Thus:

\[
\boxed{\text{sumRange}(left,right)=prefix[right]-prefix[left-1]}
\]

This formula applies when `left > 0`.

When `left == 0`, simply return `prefix[right]`.

### The complete algorithm

**During construction:**

1. Create an array called `prefix`.
2. Store the first element in `prefix[0]`.
3. For each remaining index, add the current element to the previous cumulative sum.
4. Store that result in `prefix[i]`.

**During each query:**

1. If `left == 0`, return `prefix[right]`.
2. Otherwise, return `prefix[right] - prefix[left - 1]`.

Now each query takes constant time!

---

# Step 7: Dry-Run the Optimized Approach

Let's use the original example:

```text
nums = [-2, 0, 3, -5, 2, -1]
```

### Part A: Build the prefix array

The first value is:

```text
prefix[0] = nums[0] = -2
```

For every subsequent index:

```text
prefix[i] = prefix[i - 1] + nums[i]
```

| Index `i` | `nums[i]` | Calculation | `prefix[i]` |
|---:|---:|---|---:|
| 0 | -2 | `-2` | -2 |
| 1 | 0 | `-2 + 0` | -2 |
| 2 | 3 | `-2 + 3` | 1 |
| 3 | -5 | `1 + (-5)` | -4 |
| 4 | 2 | `-4 + 2` | -2 |
| 5 | -1 | `-2 + (-1)` | -3 |

Therefore:

```text
prefix = [-2, -2, 1, -4, -2, -3]
```

### Part B: Answer the queries

**Query 1: `sumRange(0, 2)`**

Since `left == 0`:

```text
answer = prefix[2]
       = 1
```

**Query 2: `sumRange(2, 5)`**

Since `left > 0`:

```text
answer = prefix[5] - prefix[1]
       = -3 - (-2)
       = -1
```

**Query 3: `sumRange(0, 5)`**

Since `left == 0`:

```text
answer = prefix[5]
       = -3
```

The results are:

```text
[1, -1, -3]
```

These match the expected output.

---

# Step 8: Write the Final Java Code in the Correct Order

Now let's translate the algorithm into Java and understand why each part belongs where it does.

### 1. Declare the class and prefix array

```java
class NumArray {
    private int[] prefix;
```

**Why?**

The prefix sums must remain available between method calls. Therefore, `prefix` is an instance field rather than a local variable inside `sumRange()`.

### 2. Create the constructor

```java
    public NumArray(int[] nums) {
        int n = nums.length;
        prefix = new int[n];
```

**Why here?**

LeetCode creates a `NumArray` object before making queries. The constructor is the appropriate place to precompute information needed by those queries.

### 3. Initialize the first prefix sum

```java
        prefix[0] = nums[0];
```

**Why?**

There is no previous prefix sum for index `0`. Its cumulative sum is simply the first element.

### 4. Calculate the remaining prefix sums

```java
        for (int i = 1; i < n; i++) {
            prefix[i] = prefix[i - 1] + nums[i];
        }
    }
```

**Why this order?**

We calculate each prefix sum from the previous one, so the previous prefix must already exist.

Starting at index `1` also avoids accessing `prefix[-1]`.

### 5. Implement the range query

```java
    public int sumRange(int left, int right) {
        if (left == 0) {
            return prefix[right];
        }

        return prefix[right] - prefix[left - 1];
    }
}
```

**Why does this method come after the constructor?**

The constructor prepares the prefix array. The query method uses that prepared data.

The query method doesn't need to loop through the input because the cumulative sums already contain the necessary information.

## Complete Java solution

```java
class NumArray {
    private int[] prefix;

    public NumArray(int[] nums) {
        int n = nums.length;
        prefix = new int[n];

        prefix[0] = nums[0];

        for (int i = 1; i < n; i++) {
            prefix[i] = prefix[i - 1] + nums[i];
        }
    }

    public int sumRange(int left, int right) {
        if (left == 0) {
            return prefix[right];
        }

        return prefix[right] - prefix[left - 1];
    }
}
```

### Complexity analysis

Let `n` be the array length and `q` be the number of queries.

| Operation | Time complexity |
|---|---|
| Constructor / prefix preprocessing | O(n) |
| Each `sumRange()` query | O(1) |
| All `q` queries | O(q) |
| Extra space for the prefix array | O(n) |

**Total time:** O(n + q), including preprocessing and all queries.

**Extra space:** O(n), for the prefix array.

The trade-off is straightforward: we spend O(n) time and space preparing the data so that every query is answered in O(1) time.

---

# Your Revision Card: LC 303

When revising, try to reconstruct these points without looking at the code.

| Step | Question | Answer |
|---|---|---|
| 1 | What is the problem asking? | Return the sum between two inclusive indices |
| 2 | What is the straightforward approach? | Loop from `left` through `right` |
| 3 | What is the bottleneck? | Repeatedly summing overlapping ranges |
| 4 | What information can be reused? | Cumulative sums from index `0` |
| 5 | How do I build that information? | `prefix[i] = prefix[i - 1] + nums[i]` |
| 6 | How do I answer a range query? | `prefix[right] - prefix[left - 1]` |
| 7 | What if `left == 0`? | Return `prefix[right]` |
| 8 | Why is each query O(1)? | Only one conditional and at most one subtraction |

## The most important lesson

Compare this with your previous problems:

| Problem | Reusable information | Technique |
|---|---|---|
| Two Sum | Previously seen values and their indices | HashMap |
| Best Time to Buy and Sell Stock | Minimum price seen so far | Running minimum |
| Product of Array Except Self | Products to the left and right | Prefix and suffix products |
| Maximum Subarray | Best subarray ending at the previous index | Kadane's algorithm |
| LC 303 — Range Sum Query | Cumulative sums | Prefix sums |

In each problem, you begin with a correct straightforward approach, identify repeated work, and ask what information can be stored and reused.

**Your practice task:** Close this explanation and implement `NumArray` from scratch. Before writing code, explain aloud why subtracting `prefix[left - 1]` removes exactly the elements before the requested range. If you can explain that clearly, you've understood the central idea behind LC 303.
