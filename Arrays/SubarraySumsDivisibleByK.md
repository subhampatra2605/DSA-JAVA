# LeetCode 974 — Subarray Sums Divisible by K

Let's solve **Subarray Sums Divisible by K** using the same Step 1–8 framework. We'll start with brute force, identify the bottleneck, derive the prefix-sum and HashMap optimization, and then dry-run the Java code.

---

## Step 1: Understand the Problem

You are given an integer array `nums` and an integer `k`.

Find the **number of non-empty contiguous subarrays whose sums are divisible by `k`**.

A sum is divisible by `k` when:

```text
sum % k == 0
```

### Example

```java
Input: nums = [4, 5, 0, -2, -3, 1], k = 5

Output: 7
```

The seven valid subarrays are:

| Subarray | Sum | Divisible by 5? |
|---|---:|---|
| `[4, 5, 0, -2, -3, 1]` | 5 | Yes |
| `[5]` | 5 | Yes |
| `[5, 0]` | 5 | Yes |
| `[5, 0, -2]` | 3 | No |
| `[5, 0, -2, -3]` | 0 | Yes |
| `[0]` | 0 | Yes |
| `[0, -2, -3]` | -5 | Yes |

The table above illustrates some candidates, but it doesn't enumerate exactly seven valid subarrays. Let's list them accurately:

1. `[4, 5, 0, -2, -3, 1]` → sum `5`
2. `[5]` → sum `5`
3. `[5, 0]` → sum `5`
4. `[5, 0, -2, -3]` → sum `0`
5. `[0]` → sum `0`
6. `[0, -2, -3]` → sum `-5`
7. `[-2, -3]` → sum `-5`

Therefore, the answer is `7`.

**Important:** The subarray must be contiguous and non-empty. The elements may be negative or zero.

---

## Step 2: Think of the Brute-Force Approach

Before optimizing, think about how to examine every possible subarray.

### Logic

1. Choose every possible starting index.
2. Expand the subarray by moving the ending index.
3. Maintain the running sum.
4. Whenever the running sum is divisible by `k`, increment the count.
5. Return the total count.

### Brute-Force Code — Java

```java
class Solution {
    public int subarraysDivByK(int[] nums, int k) {
        int count = 0;

        // Try every possible starting index
        for (int start = 0; start < nums.length; start++) {
            int sum = 0;

            // Expand the subarray from the current starting index
            for (int end = start; end < nums.length; end++) {

                // Include the current element in the running sum
                sum += nums[end];

                // If the sum is divisible by k, count this subarray
                if (sum % k == 0) {
                    count++;
                }
            }
        }

        // Return the total number of valid subarrays
        return count;
    }
}
```

### Complexity

- **Time:** `O(n²)` — we examine every possible subarray.
- **Auxiliary space:** `O(1)` — only a few variables are used.

### Identify the Bottleneck

The nested loops examine all possible subarrays. Can we count valid subarrays without calculating every subarray sum separately?

To answer that, we need to understand **prefix sums and remainders**.

---

## Step 3: Find the Pattern

The key observation is that **two prefix sums with the same remainder when divided by `k` produce a subarray sum divisible by `k`.**

Let's derive this rather than memorize it.

Suppose we have:

```text
nums = [2, 3, 1, 6]
k = 5
```

Define the prefix sum as the sum of all elements from index `0` through the current index.

| Index | Element | Prefix sum | Remainder (`prefixSum % 5`) |
|---:|---:|---:|---:|
| 0 | 2 | 2 | 2 |
| 1 | 3 | 5 | 0 |
| 2 | 1 | 6 | 1 |
| 3 | 6 | 12 | 2 |

Notice that the remainder `2` occurs at indices `0` and `3`.

The prefix sums are:

```text
Prefix sum at index 0 = 2
Prefix sum at index 3 = 12
```

Subtract them:

```text
12 - 2 = 10
```

The subarray between these prefix boundaries is:

```text
[3, 1, 6]
```

Its sum is `10`, which is divisible by `5`.

### Why does this work?

Let:

- `prefixSum1` be an earlier prefix sum.
- `prefixSum2` be the current prefix sum.

The subarray sum between them is:

```text
subarraySum = prefixSum2 - prefixSum1
```

If both prefix sums have the same remainder when divided by `k`, then:

```text
prefixSum1 % k = prefixSum2 % k
```

Therefore, their difference is divisible by `k`:

```text
(prefixSum2 - prefixSum1) % k = 0
```

That difference is exactly the subarray sum.

**Pattern identified: Prefix Sum + HashMap of Remainder Frequencies.**

---

## Step 4: Derive the Optimal Approach

Now we know that repeated remainders help us find valid subarrays.

The next question is:

**How do we count all valid subarrays efficiently?**

We maintain a HashMap that stores how many times each remainder has appeared.

```java
Map<Integer, Integer> remainderFrequency = new HashMap<>();
```

The mapping is:

```text
Remainder → Number of times it has appeared
```

### Why Store Frequencies?

Suppose `k = 5` and the prefix remainders seen so far are:

```text
0, 2, 2, 2
```

The remainder `2` has appeared three times.

If the current prefix also has remainder `2`, it can form a divisible-sum subarray with **each of those three earlier prefixes**.

Therefore, we add the frequency of the current remainder to our answer.

### The Initial Mapping

We initialize:

```java
remainderFrequency.put(0, 1);
```

This represents the empty prefix, whose sum is zero before the array begins.

Why is this necessary?

Consider:

```text
nums = [5]
k = 5
```

The prefix sum is `5`, and its remainder is `0`.

The subarray `[5]` is divisible by `5`, even though there is no earlier array element to form a prefix with.

The initial mapping lets us count it:

```text
remainderFrequency[0] = 1
```

When we encounter remainder `0`, we add `1` to the count.

### Handling Negative Numbers

Java can return a negative remainder when the dividend is negative.

For example:

```java
-2 % 5 == -2
```

But mathematically, the normalized remainder is `3`, because:

```text
-2 = (-1 × 5) + 3
```

We normalize the remainder with:

```java
int remainder = ((prefixSum % k) + k) % k;
```

This ensures that the remainder is always in the range `0` to `k - 1`, assuming `k > 0`.

### The Correct Order of Operations

For each array element:

1. Add the element to `prefixSum`.
2. Calculate and normalize the remainder.
3. Add the existing frequency of that remainder to `count`.
4. Increase the frequency of that remainder in the HashMap.

In code:

```java
prefixSum += nums[i];

int remainder = ((prefixSum % k) + k) % k;

count += remainderFrequency.getOrDefault(remainder, 0);

remainderFrequency.put(
    remainder,
    remainderFrequency.getOrDefault(remainder, 0) + 1
);
```

**Why count before updating the map?**

We only want to count subarrays formed with earlier prefix sums. Updating the frequency first could count the current prefix against itself, effectively counting an empty subarray.

---

## Step 5: Write the Optimal Algorithm

### Optimal Code — Java

```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int subarraysDivByK(int[] nums, int k) {
        int prefixSum = 0;
        int count = 0;

        // Store each normalized remainder and its frequency
        Map<Integer, Integer> remainderFrequency = new HashMap<>();

        // An empty prefix has sum 0 and remainder 0
        remainderFrequency.put(0, 1);

        // Traverse the array and maintain the running prefix sum
        for (int i = 0; i < nums.length; i++) {

            // Add the current element to the prefix sum
            prefixSum += nums[i];

            // Normalize the remainder to handle negative prefix sums
            int remainder = ((prefixSum % k) + k) % k;

            // Every earlier matching remainder forms a valid subarray
            count += remainderFrequency.getOrDefault(remainder, 0);

            // Record the current remainder for future subarrays
            remainderFrequency.put(
                remainder,
                remainderFrequency.getOrDefault(remainder, 0) + 1
            );
        }

        // Return the total number of subarrays divisible by k
        return count;
    }
}
```

### Complexity

- **Time:** `O(n)` average — one traversal with average constant-time HashMap operations.
- **Auxiliary space:** `O(min(n, k))` — there are at most `k` distinct normalized remainders and at most `n + 1` prefix boundaries.

---

## Step 6: Dry Run the Optimal Code

Let's use the example from the problem:

```text
nums = [4, 5, 0, -2, -3, 1]
k = 5
```

Initially:

```text
prefixSum = 0
count = 0

remainderFrequency = {
    0: 1
}
```

The initial frequency of remainder `0` accounts for the empty prefix.

| Index | Element | Prefix sum | Remainder | Previous frequency | Count after processing | Frequency updated to |
|---:|---:|---:|---:|---:|---:|---:|
| 0 | 4 | 4 | 4 | 0 | 0 | 4 → 1 |
| 1 | 5 | 9 | 4 | 1 | 1 | 4 → 2 |
| 2 | 0 | 9 | 4 | 2 | 3 | 4 → 3 |
| 3 | -2 | 7 | 2 | 0 | 3 | 2 → 1 |
| 4 | -3 | 4 | 4 | 3 | 6 | 4 → 4 |
| 5 | 1 | 5 | 0 | 1 | 7 | 0 → 2 |

The final answer is:

```text
7
```

Let's examine the most important iterations.

### Iteration 1: Index 0

```text
nums[0] = 4

prefixSum = 4
remainder = 4
```

Remainder `4` has not appeared before.

```text
count = 0
```

Record its frequency:

```text
remainderFrequency = {
    0: 1,
    4: 1
}
```

### Iteration 2: Index 1

```text
nums[1] = 5

prefixSum = 9
remainder = 4
```

Remainder `4` has appeared once before.

That earlier prefix sum was `4`. The current prefix sum is `9`.

Subtract them:

```text
9 - 4 = 5
```

This gives the subarray:

```text
[5]
```

Its sum is divisible by `5`.

So:

```text
count = 0 + 1 = 1
```

Then increase the frequency of remainder `4` from `1` to `2`.

### Iteration 3: Index 2

```text
nums[2] = 0

prefixSum = 9
remainder = 4
```

Remainder `4` has appeared twice before.

Therefore, two earlier prefix boundaries form valid subarrays ending at this index.

The subarrays are:

```text
[5, 0]
[0]
```

Both have sums divisible by `5`.

So:

```text
count = 1 + 2 = 3
```

The frequency of remainder `4` becomes `3`.

### Iteration 4: Index 3

```text
nums[3] = -2

prefixSum = 7
remainder = 2
```

The normalized remainder is `2`.

It has not appeared before, so no new valid subarray is counted.

```text
count = 3
```

### Iteration 5: Index 4

```text
nums[4] = -3

prefixSum = 4
remainder = 4
```

Remainder `4` has appeared three times before.

Each earlier occurrence creates a valid subarray ending at index `4`.

The three new valid subarrays are:

```text
[4, 5, 0, -2, -3]  → sum = 4
[5, 0, -2, -3]      → sum = 0
[0, -2, -3]         → sum = -5
```

All three sums are divisible by `5`.

Therefore:

```text
count = 3 + 3 = 6
```

### Iteration 6: Index 5

```text
nums[5] = 1

prefixSum = 5
remainder = 0
```

Remainder `0` has appeared once before, from the initial empty prefix.

This creates one more valid subarray:

```text
[4, 5, 0, -2, -3, 1]
```

Its sum is `5`.

Therefore:

```text
count = 6 + 1 = 7
```

**Final output: `7`**

---

## Step 7: Understand the Tricky Parts

### 1. Why do equal remainders produce a divisible subarray?

Suppose two prefix sums have the same remainder when divided by `k`.

Their difference must be divisible by `k`.

Since that difference is the sum of the subarray between the two prefix boundaries, the subarray is valid.

### 2. Why store frequencies instead of just checking whether a remainder exists?

Because one remainder may appear many times.

If a remainder has appeared three times before the current index, there are three different earlier prefix boundaries that can form valid subarrays ending at the current index.

Therefore:

```java
count += remainderFrequency.getOrDefault(remainder, 0);
```

We add the frequency, not just `1`.

### 3. Why normalize negative remainders?

Java's `%` operator can produce negative results for negative prefix sums.

Using:

```java
int remainder = ((prefixSum % k) + k) % k;
```

ensures that mathematically equivalent remainders are represented by the same non-negative value.

For example:

```text
-2 % 5 = -2 in Java

Normalized remainder:
((-2 % 5) + 5) % 5
= (-2 + 5) % 5
= 3
```

### 4. Why initialize `remainderFrequency.put(0, 1)`?

It allows the algorithm to count subarrays whose sum is divisible by `k` and that begin at index `0`.

Without this initial mapping, those subarrays would be missed.

### 5. Why update the map after counting?

We need to count matching **earlier** prefix boundaries. Counting before inserting the current remainder avoids counting an empty subarray.

### 6. How is this different from LC 560?

Both use prefix sums and a HashMap of frequencies.

- **LC 560:** Find previous prefix sums equal to `currentPrefixSum - k`.
- **LC 974:** Find previous prefix sums whose remainder matches the current prefix sum's remainder when divided by `k`.

The key difference is the condition being checked.

---

## Step 8: Complexity Analysis and Final Revision

### Brute Force vs. Optimal

| Approach | Time Complexity | Auxiliary Space | Main Idea |
|---|---|---|---|
| Brute force | `O(n²)` | `O(1)` | Generate every subarray and check its sum |
| Optimal | `O(n)` average | `O(min(n, k))` | Use prefix-sum remainders and a HashMap of frequencies |

### The Complete Logic to Remember

When you encounter a problem asking you to **count subarrays whose sum is divisible by `k`**, think:

1. A subarray sum equals the difference between two prefix sums.
2. If two prefix sums have the same remainder modulo `k`, their difference is divisible by `k`.
3. Maintain a running prefix sum.
4. Normalize the remainder to handle negative numbers.
5. Store each remainder's frequency in a HashMap.
6. Add the existing frequency to the answer before updating the map.
7. Initialize the map with `0 → 1` to handle subarrays starting at index `0`.
8. Return the total count.

### Your Practice Questions

Before moving to the next problem, try answering these without looking at the explanation:

1. Why do two prefix sums with the same remainder produce a subarray sum divisible by `k`?
2. Why do we add the **frequency** of a remainder instead of simply incrementing `count` by one?
3. Why must we normalize negative remainders in Java?
4. What is the purpose of `remainderFrequency.put(0, 1)`?

If you can explain these four points and write the optimal solution from scratch, you've understood the core pattern behind **LeetCode 974 — Subarray Sums Divisible by K**.
