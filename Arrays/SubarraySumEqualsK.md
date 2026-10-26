# LeetCode 560 — Subarray Sum Equals K

Let's solve this using your **8-step problem-solving framework**, just as we did for Two Sum, Maximum Subarray, and Product of Array Except Self.

This is an especially important problem for your Infosys SP preparation because it teaches you how to combine **prefix sums and a HashMap** to count subarrays efficiently.

The goal is to understand why the optimized solution works, not just memorize the code.

---

# Step 1: Understand the Problem

### Problem statement

Given an integer array `nums` and an integer `k`, return the **total number of contiguous subarrays whose sum equals `k`**.

Important points:
- A subarray must be contiguous.
- Negative numbers and zeros are allowed.
- We must count all qualifying subarrays, not return their indices.
- Different subarrays can have the same sum.

### Example

```text
nums = [1, 2, 3]
k = 3
```

Which subarrays have a sum of `3`?

```text
[1, 2] → 1 + 2 = 3
[3]    → 3
```

Therefore:

```text
Output: 2
```

Notice that `[1, 2]` and `[3]` are two different subarrays.

**Your first checkpoint:** We need to count every contiguous subarray whose sum is exactly `k`.

---

# Step 2: Solve a Small Example Manually

Consider:

```text
nums = [1, 1, 1]
k = 2
```

Let's enumerate the subarrays.

Starting at index `0`:

```text
[1]       → sum = 1
[1, 1]    → sum = 2 ✓
[1, 1, 1] → sum = 3
```

Starting at index `1`:

```text
[1]       → sum = 1
[1, 1]    → sum = 2 ✓
```

Starting at index `2`:

```text
[1] → sum = 1
```

There are two valid subarrays:

```text
[1, 1] at indices 0–1
[1, 1] at indices 1–2
```

Therefore:

```text
Output: 2
```

Now ask yourself:

**If I know the sum of a larger range, can I subtract a previous sum to obtain the sum of a subarray?**

Yes. This is the key observation behind the optimized solution.

---

# Step 3: Derive the Brute-Force Approach

Start with the simplest correct solution.

### Brute-force logic

1. Choose a starting index `i`.
2. Initialize `currentSum = 0`.
3. Extend the subarray using an ending index `j`, starting at `i`.
4. Add `nums[j]` to `currentSum`.
5. If `currentSum == k`, increment the answer.
6. Repeat for every starting index.

### Java code

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int count = 0;

        for (int i = 0; i < nums.length; i++) {
            int currentSum = 0;

            for (int j = i; j < nums.length; j++) {
                currentSum += nums[j];

                if (currentSum == k) {
                    count++;
                }
            }
        }

        return count;
    }
}
```

This solution is correct and takes **O(n²) time** and O(1) auxiliary space.

Notice that we don't reset `currentSum` inside the inner loop. We keep extending the same subarray as `j` moves forward.

---

# Step 4: Dry-Run the Brute Force

Use:

```text
nums = [1, 2, 1]
k = 3
```

Initially:

```text
count = 0
```

### Starting index `i = 0`

| `j` | Subarray | `currentSum` | Equals `k`? | `count` |
|---:|---|---:|---|---:|
| 0 | `[1]` | 1 | No | 0 |
| 1 | `[1, 2]` | 3 | Yes | 1 |
| 2 | `[1, 2, 1]` | 4 | No | 1 |

### Starting index `i = 1`

Reset `currentSum = 0`.

| `j` | Subarray | `currentSum` | Equals `k`? | `count` |
|---:|---|---:|---|---:|
| 1 | `[2]` | 2 | No | 1 |
| 2 | `[2, 1]` | 3 | Yes | 2 |

### Starting index `i = 2`

Reset `currentSum = 0`.

| `j` | Subarray | `currentSum` | Equals `k`? | `count` |
|---:|---|---:|---|---:|
| 2 | `[1]` | 1 | No | 2 |

Final result:

```text
count = 2
```

The brute-force approach works. Now we need to understand why it can be improved.

---

# Step 5: Identify the Bottleneck

Ask yourself three questions.

### Question 1: What work am I repeating?

For every starting index, we examine all possible ending indices.

The total number of contiguous subarrays is:

\[
\frac{n(n+1)}{2}
\]

That gives us O(n²) time.

### Question 2: What information can I reuse?

Consider this array:

```text
nums = [1, 2, 1]
```

Define the prefix sum at index `i` as the sum of all elements from index `0` through `i`.

The prefix sums are:

| Index | Element | Prefix sum |
|---:|---:|---:|
| 0 | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 1 | 4 |

Let's use these sums to find the subarray from index `1` to index `2`.

Its sum is:

```text
nums[1] + nums[2] = 2 + 1 = 3
```

We can calculate that using:

```text
prefix[2] - prefix[0] = 4 - 1 = 3
```

The sum of a range can be obtained by subtracting the prefix sum before that range from the prefix sum at its ending index.

### Question 3: How do we find subarrays whose sum is `k`?

Suppose the current prefix sum is `currentPrefix`.

Let the prefix sum before a subarray be `previousPrefix`.

The sum of the subarray between those two positions is:

\[
\text{subarraySum} = \text{currentPrefix} - \text{previousPrefix}
\]

We want:

\[
\text{subarraySum} = k
\]

Therefore:

\[
\text{currentPrefix} - \text{previousPrefix} = k
\]

Rearrange:

\[
\boxed{\text{previousPrefix} = \text{currentPrefix} - k}
\]

**This equation is the central insight of LC 560.**

Instead of checking every possible starting index, we can check whether the prefix sum `currentPrefix - k` has appeared previously.

But how do we remember previously seen prefix sums and how often they occurred?

We use a HashMap.

---

# Step 6: Derive the Optimized Algorithm

We'll maintain:

- `prefixSum`: the cumulative sum up to the current index.
- `count`: the total number of valid subarrays found.
- `map`: a HashMap that stores each previously seen prefix sum and its frequency.

The map represents:

```text
prefix sum → number of times it has appeared
```

Let's derive the algorithm in the correct order.

## Part A: Initialize the HashMap

```java
Map<Integer, Integer> map = new HashMap<>();
map.put(0, 1);
```

Why do we put `0 → 1` in the map before processing the array?

Consider:

```text
nums = [3]
k = 3
```

The current prefix sum becomes `3`.

We calculate:

```text
currentPrefix - k = 3 - 3 = 0
```

There is a valid subarray `[3]`, starting at index `0`.

We need the map to recognize that the prefix sum `0` occurred once **before the array began**.

This is why we initialize:

```text
map = {0: 1}
```

It represents the empty prefix, whose sum is zero.

Without this initialization, subarrays starting at index `0` would not be counted correctly.

## Part B: Calculate the current prefix sum

For each element:

```java
prefixSum += nums[i];
```

This extends the prefix sum from the previous index by incorporating the current element.

## Part C: Calculate the prefix sum we need

From our equation:

```text
previousPrefix = currentPrefix - k
```

In Java:

```java
int needed = prefixSum - k;
```

Now we know which previous prefix sum would produce a subarray sum of `k`.

## Part D: Check whether the needed prefix sum exists

```java
if (map.containsKey(needed)) {
    count += map.get(needed);
}
```

**Why add the frequency instead of just incrementing `count` by one?**

Because the same prefix sum can occur multiple times, and each occurrence can represent a different valid subarray.

Consider:

```text
nums = [0, 0, 0]
k = 0
```

Every non-empty contiguous subarray has sum zero.

There are six such subarrays.

The prefix sum `0` repeatedly occurs. We need to count every earlier occurrence that can form a valid subarray ending at the current index.

If `map.get(needed)` is `3`, for example, there are three valid subarrays ending at the current index, not just one.

## Part E: Record the current prefix sum

```java
map.put(prefixSum, map.getOrDefault(prefixSum, 0) + 1);
```

This records the current prefix sum so future elements can use it.

Why update the map **after** checking for `needed`?

Because the subarray must be non-empty.

If we insert the current prefix sum first, then when `k = 0`, we could match the current prefix sum against itself and incorrectly count an empty subarray.

Checking first ensures that we only use prefix sums from earlier positions.

### The optimized algorithm in plain English

1. Initialize the prefix-sum frequency map with `0 → 1`.
2. Start with `prefixSum = 0` and `count = 0`.
3. Visit every element from left to right.
4. Add the current element to `prefixSum`.
5. Calculate `needed = prefixSum - k`.
6. Add the frequency of `needed` to `count`.
7. Record the current prefix sum in the map.
8. Return `count` after processing every element.

This gives us an average time complexity of **O(n)**.

---

# Step 7: Dry-Run the Optimized Algorithm

Consider:

```text
nums = [1, 2, 1]
k = 3
```

Initialize:

```text
prefixSum = 0
count = 0
map = {0: 1}
```

### Detailed trace

| Index | `nums[i]` | `prefixSum` | `needed = prefixSum - k` | Frequency of `needed` | `count` after check | Map after insertion |
|---:|---:|---:|---:|---:|---:|---|
| 0 | 1 | 1 | -2 | 0 | 0 | `{0:1, 1:1}` |
| 1 | 2 | 3 | 0 | 1 | 1 | `{0:1, 1:1, 3:1}` |
| 2 | 1 | 4 | 1 | 1 | 2 | `{0:1, 1:1, 3:1, 4:1}` |

At index `1`:

```text
prefixSum = 3
needed = 3 - 3 = 0
```

The map contains `0 → 1`, so we count one valid subarray:

```text
[1, 2]
```

At index `2`:

```text
prefixSum = 4
needed = 4 - 3 = 1
```

The map contains `1 → 1`, so we count another valid subarray:

```text
[2, 1]
```

Final result:

```text
count = 2
```

### Test an important edge case

```text
nums = [0, 0, 0]
k = 0
```

Expected answer:

```text
6
```

The algorithm counts:

- At index `0`: one valid subarray.
- At index `1`: two additional valid subarrays.
- At index `2`: three additional valid subarrays.

Total:

```text
1 + 2 + 3 = 6
```

This is why we store **frequencies**, not just whether a prefix sum exists.

---

# Step 8: Write the Final Java Code in the Correct Order

Let's translate the algorithm into Java.

### 1. Define the method and initialize variables

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int count = 0;
        int prefixSum = 0;
```

We initialize `count` to zero because we haven't found any valid subarrays, and `prefixSum` to zero because no elements have been processed yet.

### 2. Create the frequency map

```java
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, 1);
```

We need to remember previously seen prefix sums and their frequencies. The initial `0 → 1` handles valid subarrays that start at index `0`.

Remember to import the required classes.

### 3. Traverse the array

```java
        for (int i = 0; i < nums.length; i++) {
            prefixSum += nums[i];
```

We process every element once and update the running prefix sum.

### 4. Calculate the required previous prefix sum

```java
            int needed = prefixSum - k;
```

This comes directly from:

```text
currentPrefix - previousPrefix = k
```

Therefore:

```text
previousPrefix = currentPrefix - k
```

### 5. Count matching prefix sums

```java
            count += map.getOrDefault(needed, 0);
```

If `needed` exists, its frequency tells us how many valid subarrays end at the current index.

If it doesn't exist, the frequency is zero.

### 6. Record the current prefix sum

```java
            map.put(prefixSum, map.getOrDefault(prefixSum, 0) + 1);
```

This happens after counting so that the current prefix cannot match itself to create an empty subarray.

### 7. Return the answer

```java
        }

        return count;
    }
}
```

We return after processing the entire array because we need the total number of valid subarrays.

## Complete optimized Java solution

```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int subarraySum(int[] nums, int k) {
        int count = 0;
        int prefixSum = 0;

        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, 1);

        for (int i = 0; i < nums.length; i++) {
            prefixSum += nums[i];

            int needed = prefixSum - k;

            count += map.getOrDefault(needed, 0);

            map.put(prefixSum, map.getOrDefault(prefixSum, 0) + 1);
        }

        return count;
    }
}
```

### Complexity analysis

- **Time:** O(n) on average, because each element requires a constant number of HashMap operations.
- **Auxiliary space:** O(n) in the worst case, because the map can store up to `n + 1` distinct prefix sums.

---

# Your Revision Card: LC 560

When revising, try to answer these questions without looking at the code.

| Step | Question | Answer |
|---|---|---|
| 1 | What are we counting? | Contiguous subarrays with sum equal to `k` |
| 2 | What is the brute-force approach? | Enumerate all starting and ending indices |
| 3 | What is the bottleneck? | Checking O(n²) subarrays |
| 4 | What is the prefix-sum equation? | `currentPrefix - previousPrefix = k` |
| 5 | What previous sum do we need? | `currentPrefix - k` |
| 6 | Why use a HashMap? | To find previous prefix sums and their frequencies efficiently |
| 7 | Why initialize `map` with `0 → 1`? | To count subarrays beginning at index `0` |
| 8 | Why update the map after counting? | To avoid using the current prefix as its own previous prefix |

## Compare your prefix-sum problems

| Problem | What the prefix sum helps you do |
|---|---|
| LC 303 — Range Sum Query | Calculate the sum of a range using subtraction |
| LC 560 — Subarray Sum Equals K | Count previous prefix sums equal to `currentPrefix - k` |

The important difference is that **LC 303 uses a prefix array, while LC 560 uses a prefix-sum frequency map**.

For LC 560, remember this order:

```text
1. Update prefixSum
2. Calculate needed = prefixSum - k
3. Add the frequency of needed to count
4. Insert/increment the current prefixSum in the map
```

If you can derive that order from the equation rather than memorizing it, you've understood the central idea of the problem.
