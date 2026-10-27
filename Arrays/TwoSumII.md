# LeetCode 167 — Two Sum II: Input Array Is Sorted

We'll solve this using our usual **Step 1–8 framework**: understand the problem, develop brute force, identify the pattern, optimize, write Java code, dry-run it, understand tricky parts, and revise the complexity.

The key pattern here is **Two Pointers**.

---

## Step 1: Understand the Problem

You are given a **1-indexed, sorted array of integers** called `numbers` and an integer `target`.

Find two numbers such that:

```text
numbers[i] + numbers[j] = target
```

Return their indices as a 1-indexed array:

```text
[index1, index2]
```

Conditions:
- The array is sorted in non-decreasing order.
- Exactly one solution exists.
- You must use two different elements.
- You must use constant extra space.
- The returned indices must be 1-indexed.

### Example

```java
Input: numbers = [2, 7, 11, 15], target = 9

Output: [1, 2]
```

Explanation:

```text
numbers[1] + numbers[2] = 2 + 7 = 9
```

Remember that the problem uses **1-based indexing**, not Java's usual 0-based indexing.

---

## Step 2: Think of the Brute-Force Approach

Before optimizing, consider the simplest way to find the pair.

### Logic

1. Choose an element using the outer loop.
2. Choose another element after it using the inner loop.
3. Check whether their sum equals `target`.
4. If it does, return their 1-based indices.
5. If no pair is found, return an empty array (though the problem guarantees a solution).

### Brute-Force Code — Java

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {

        // Choose the first element
        for (int i = 0; i < numbers.length; i++) {

            // Choose the second element after the first
            for (int j = i + 1; j < numbers.length; j++) {

                // Check whether the two elements sum to the target
                if (numbers[i] + numbers[j] == target) {

                    // Convert the 0-based indices to 1-based indices
                    return new int[]{i + 1, j + 1};
                }
            }
        }

        // The problem guarantees a solution, so this is a fallback
        return new int[]{};
    }
}
```

### Complexity

- **Time:** `O(n²)` — two nested loops examine pairs of elements.
- **Auxiliary space:** `O(1)` — no additional data structure is required.

### Identify the Bottleneck

The brute-force approach repeatedly checks pairs.

But look carefully at the problem:

**The array is sorted. Can we use its order to eliminate unnecessary comparisons?**

Yes! We can use two pointers.

---

## Step 3: Find the Pattern

The array is sorted in ascending order, so:

- The smallest element is at the beginning.
- The largest element is at the end.

We can use two pointers:

```text
left  → first element
right → last element
```

For example:

```text
numbers = [2, 7, 11, 15]
target = 9
```

Initially:

```text
left                     right
 ↓                         ↓
[2, 7, 11, 15]
```

Calculate:

```text
sum = numbers[left] + numbers[right]
    = 2 + 15
    = 17
```

Since `17 > 9`, the sum is too large.

Because the array is sorted, moving `left` to the right would make the sum even larger.

Instead, we decrease `right` to select a smaller number.

This gives us the two-pointer rules.

### Rule 1: If the sum equals the target

```java
sum == target
```

We have found the answer.

### Rule 2: If the sum is greater than the target

```java
sum > target
```

Move `right` leftward:

```java
right--;
```

Why? We need a smaller sum.

### Rule 3: If the sum is less than the target

```java
sum < target
```

Move `left` rightward:

```java
left++;
```

Why? We need a larger sum.

**Pattern identified: Two Pointers on a Sorted Array.**

---

## Step 4: Derive the Optimal Approach

Let's derive the logic using an example.

```text
numbers = [2, 7, 11, 15]
target = 9
```

Initialize:

```java
int left = 0;
int right = numbers.length - 1;
```

Our pointers are:

```text
left = 0
right = 3
```

### Iteration 1

```text
[2, 7, 11, 15]
 ↑           ↑
left       right
```

Calculate:

```text
sum = 2 + 15 = 17
```

Since `17 > 9`, move `right` leftward.

```java
right--;
```

Now:

```text
left = 0
right = 2
```

### Iteration 2

```text
[2, 7, 11, 15]
 ↑       ↑
left   right
```

Calculate:

```text
sum = 2 + 11 = 13
```

Since `13 > 9`, move `right` leftward again.

```java
right--;
```

Now:

```text
left = 0
right = 1
```

### Iteration 3

```text
[2, 7, 11, 15]
 ↑   ↑
left right
```

Calculate:

```text
sum = 2 + 7 = 9
```

Since `9 == target`, return:

```java
return new int[]{left + 1, right + 1};
```

The result is:

```text
[1, 2]
```

### Why Does This Algorithm Work?

The sorted order lets us eliminate one candidate at a time.

If the sum is too large, the element at `right` is too large to pair with the current `left`. It cannot form the target with any element at or to the right of `left`, because those elements are at least as large as `numbers[left]`.

So we move `right` left.

Similarly, if the sum is too small, the element at `left` is too small to pair with the current `right`. It cannot form the target with any element at or to the left of `right`, because those elements are no larger than `numbers[right]`.

So we move `left` right.

Each move eliminates a candidate without needing to examine every pair.

---

## Step 5: Write the Optimal Algorithm

### Optimal Code — Java

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {

        // Start one pointer at the beginning
        int left = 0;

        // Start the other pointer at the end
        int right = numbers.length - 1;

        // Continue while the pointers refer to different elements
        while (left < right) {

            // Calculate the sum of the two selected elements
            int sum = numbers[left] + numbers[right];

            // If the sum matches the target, return 1-based indices
            if (sum == target) {
                return new int[]{left + 1, right + 1};
            }

            // If the sum is too large, select a smaller right element
            else if (sum > target) {
                right--;
            }

            // If the sum is too small, select a larger left element
            else {
                left++;
            }
        }

        // The problem guarantees exactly one solution
        return new int[]{};
    }
}
```

### Complexity

- **Time:** `O(n)` — each pointer moves only toward the other pointer, and each move eliminates a candidate.
- **Auxiliary space:** `O(1)` — only two pointers and a sum variable are used.

This satisfies the constant-extra-space requirement.

---

## Step 6: Dry Run the Optimal Code

Let's use another example.

```text
numbers = [1, 3, 4, 6, 8, 11]
target = 10
```

Initially:

```text
left = 0
right = 5
```

| Iteration | Left value | Right value | Sum | Decision |
|---:|---:|---:|---:|---|
| 1 | 1 | 11 | 12 | Sum too large → `right--` |
| 2 | 1 | 8 | 9 | Sum too small → `left++` |
| 3 | 3 | 8 | 11 | Sum too large → `right--` |
| 4 | 3 | 6 | 9 | Sum too small → `left++` |
| 5 | 4 | 6 | 10 | Target found |

At the final iteration:

```text
left = 2
right = 3
```

The values are:

```text
numbers[2] = 4
numbers[3] = 6
```

Their sum is:

```text
4 + 6 = 10
```

Convert the indices to 1-based indexing:

```text
[2 + 1, 3 + 1] = [3, 4]
```

**Final output:**

```text
[3, 4]
```

---

## Step 7: Understand the Tricky Parts

### 1. Why use two pointers instead of a HashMap?

A HashMap could find a pair in average `O(n)` time, but it requires `O(n)` additional space.

The problem explicitly requires constant extra space.

Because the array is sorted, two pointers give us:

- `O(n)` time
- `O(1)` auxiliary space

### 2. Why is the array's sorted order important?

Without sorting, moving a pointer wouldn't reliably tell us whether the sum would increase or decrease.

In a sorted array:

- Moving `left` right increases or preserves the left value.
- Moving `right` left decreases or preserves the right value.

This predictable behavior is what makes the algorithm work.

### 3. Why use `while (left < right)`?

We need two different elements.

If:

```java
left == right
```

both pointers refer to the same element, so we must stop.

### 4. Why return `left + 1` and `right + 1`?

Java arrays use zero-based indexing, but LeetCode 167 explicitly requests 1-based indices.

For example:

```text
Java index:       0  1  2  3
numbers:         [2, 7, 11, 15]

Problem index:    1  2  3  4
```

Therefore:

```java
return new int[]{left + 1, right + 1};
```

### 5. Can we use binary search instead?

Yes. For each element, we could binary-search for its complement (`target - numbers[i]`).

That approach takes `O(n log n)` time and typically `O(1)` auxiliary space when implemented iteratively.

However, **two pointers are more efficient here**, achieving `O(n)` time and `O(1)` auxiliary space.

---

## Step 8: Complexity Analysis and Final Revision

### Brute Force vs. Optimal

| Approach | Time Complexity | Auxiliary Space | Main Idea |
|---|---|---|---|
| Brute force | `O(n²)` | `O(1)` | Try every pair |
| Binary search | `O(n log n)` | `O(1)` | Search for the complement of each element |
| Two pointers | `O(n)` | `O(1)` | Use sorted order to move the left or right pointer |

### The Complete Logic to Remember

When you encounter a **two-sum problem with a sorted array**, think:

1. The array is sorted, so its order can help eliminate candidates.
2. Place one pointer at the beginning and the other at the end.
3. Calculate the sum of the two selected elements.
4. If the sum equals the target, return their indices.
5. If the sum is too large, move `right` left.
6. If the sum is too small, move `left` right.
7. Continue until the answer is found.
8. Return 1-based indices if the problem requires them.

### Connection to Your Previous Problems

| LeetCode | Pattern | Main Optimization |
|---|---|---|
| LC 525 — Contiguous Array | Prefix sum + HashMap | Store the earliest index of each balance |
| LC 974 — Subarray Sums Divisible by K | Prefix sum + HashMap | Store the frequency of each remainder |
| LC 167 — Two Sum II | Two pointers | Use sorted order to eliminate candidate pairs |

**The key lesson:** Don't automatically use a HashMap for every array problem. First check whether the array is sorted and whether two pointers can solve the problem with less space.

### Practice Before Moving On

Try to write the optimal solution without looking above, then answer:

1. Why do we move `right` when the sum is greater than the target?
2. Why do we move `left` when the sum is smaller than the target?
3. Why does the algorithm use `O(n)` time instead of `O(n²)`?
4. Why is `left + 1` returned instead of `left`?

If you can explain these four questions and implement the code independently, you've understood the two-pointer pattern behind **LeetCode 167**.
