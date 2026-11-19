# **LeetCode 215 — Kth Largest Element in an Array**



Medium

Heap / Quickselect



This problem introduces two important techniques for finding the largest or smallest elements without necessarily sorting the entire array.

The key idea:

**Find the element that would appear at position&#x20;**`n - k`**&#x20;if the array were sorted in ascending order.**

We'll understand three approaches:

1. Sorting — simplest approach.
2. Min Heap — efficient and intuitive.
3. Quickselect — average \\(O(n)\\), the optimal expected-time approach.

## **1. Understand the problem**

You are given an integer array `nums` and an integer `k`.

Return the **k-th largest element** in the array.

Important: The answer is based on sorted order, not distinct values. Duplicate elements count separately.

**Example 1**

```
Input: nums = [3,2,1,5,6,4], k = 2
Output: 5
```

Sorted in descending order:

```
[6, 5, 4, 3, 2, 1]
```

The second largest element is `5`.

**Example 2**

```
Input: nums = [3,2,3,1,2,4,5,5,6], k = 4
Output: 4
```

Sorted in descending order:

```
[6, 5, 5, 4, 3, 3, 2, 2, 1]
```

The fourth largest element is `4`.

## **2. Brute-force approach — Sorting**

The simplest way is to sort the entire array in ascending order.

After sorting:

- Smallest element is at index `0`.
- Largest element is at index `n - 1`.
- Second largest is at index `n - 2`.
- K-th largest is at index `n - k`.

### **Java code**

```
import java.util.Arrays;

class Solution {
    public int findKthLargest(int[] nums, int k) {

        // Sort the array in ascending order.
        Arrays.sort(nums);

        // K-th largest element is at index n-k.
        return nums[nums.length - k];
    }
}
```

**Dry run:**

```
nums = [3,2,1,5,6,4]
k = 2
```

After sorting:

```
[1,2,3,4,5,6]
```

Calculate:

```
index = n - k
      = 6 - 2
      = 4
```

```
nums[4] = 5
```

Answer = `5`.

**Complexity:**

- Time: \\(O(n\log n)\\)
- Auxiliary space: Depends on the sorting implementation; Java's primitive-array sort uses an implementation-specific sorting strategy.

This works, but sorting everything is unnecessary if we only need one element.

## **3. Identify the bottleneck**

Sorting gives us the answer, but it processes the entire array even though we only need one element.

For example:

```
nums = [3,2,1,5,6,4]
k = 2
```

We don't need to know the exact order of every element.

We only need to identify the element that has exactly one element larger than it.

This leads to two useful approaches:

- **Min Heap:** Keep only the `k` largest elements.
- **Quickselect:** Partition the array until the target index is correctly positioned.

## **4. Recognize the patterns**

| Approach    | Pattern                        | Main idea                                         |
| ----------- | ------------------------------ | ------------------------------------------------- |
| Sorting     | Sorting                        | Sort everything                                   |
| Min Heap    | Heap / Priority Queue          | Maintain the k largest elements                   |
| Quickselect | Divide and Conquer / Partition | Position the target element without fully sorting |

Let's understand the heap approach first, then Quickselect.

## **5. Derive the optimal approaches**

### **Approach A — Min Heap (PriorityQueue)**

A Min Heap always keeps the smallest element at its root.

Java's `PriorityQueue` is a Min Heap by default.

Our strategy:

1. Insert each number into the heap.
2. If heap size becomes greater than `k`, remove the smallest element.
3. At the end, the heap contains the `k` largest elements.
4. The root is the smallest among those `k` elements — exactly the k-th largest.

**Why does this work?**

Suppose:

```
nums = [3,2,1,5,6,4]
k = 2
```

We only want to retain the two largest elements.

The heap evolves as follows:

| Insert | Heap contents (conceptually) | Action   |
| ------ | ---------------------------- | -------- |
| 3      | `[3]`                        | Keep     |
| 2      | `[2,3]`                      | Keep     |
| 1      | `[2,3,1]`                    | Remove 1 |
| 5      | `[3,5,2]`                    | Remove 2 |
| 6      | `[5,6,3]`                    | Remove 3 |
| 4      | `[5,6,4]`                    | Remove 4 |

The final heap contains:

```
[5,6]
```

The root is `5`, which is the second largest element.

### **Java code — Min Heap**

```
import java.util.PriorityQueue;

class Solution {
    public int findKthLargest(int[] nums, int k) {

        // Create a Min Heap.
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();

        // Process every element.
        for (int num : nums) {

            // Insert the current number.
            minHeap.offer(num);

            // Keep only the k largest elements.
            if (minHeap.size() > k) {
                minHeap.poll();
            }
        }

        // Root is the k-th largest element.
        return minHeap.peek();
    }
}
```

**Complexity:**

- Time: \\(O(n\log k)\\) — each insertion/removal costs \\(O(\log k)\\), and heap size never exceeds `k + 1`.
- Space: \\(O(k)\\) — heap stores at most `k` elements.

This is an excellent interview solution, especially when `k` is much smaller than `n`.

### **Approach B — Quickselect**

Quickselect is based on the partition operation used in Quick Sort.

But there is one major difference:

- Quick Sort recursively sorts both sides of the pivot.
- Quickselect only continues searching in the side containing the target index.

That is why Quickselect has **average&#x20;**\\(O(n)\\) time.

#### **Step 1: Convert k-th largest into an array index**

Imagine sorting the array in ascending order:

```
[1,2,3,4,5,6]
```

The k-th largest element is at:

\\[ targetIndex = n-k \\]

For our example:

```
n = 6
k = 2

targetIndex = 6 - 2 = 4
```

We need to position the element that belongs at index `4`.

#### **Step 2: Understand partitioning**

Choose a pivot and rearrange the array so that:

- Elements smaller than or equal to the pivot are on its left.
- Elements greater than the pivot are on its right.
- The pivot is placed at its correct sorted position.

For example:

```
[3,2,1,5,6,4]
```

Choose pivot `4`.

After partitioning, one possible arrangement is:

```
[3,2,1,4,6,5]
         ↑
      pivot
```

The pivot is now at index `3`.

If our target index is `4`, we only need to search the right side.

We don't need to sort the left side.

#### **Step 3: Repeat only on the required side**

- If `pivotIndex == targetIndex`, return the pivot.
- If `pivotIndex < targetIndex`, search the right side.
- If `pivotIndex > targetIndex`, search the left side.

This eliminates unnecessary work.

## **6. Quickselect — Java solution**

We'll use a randomized pivot to reduce the chance of repeatedly getting highly unbalanced partitions.

```
import java.util.concurrent.ThreadLocalRandom;

class Solution {

    public int findKthLargest(int[] nums, int k) {

        // Convert k-th largest into ascending-order index.
        int targetIndex = nums.length - k;

        int left = 0;
        int right = nums.length - 1;

        // Continue partitioning until target is positioned.
        while (left <= right) {

            int pivotIndex = partition(nums, left, right);

            // Target is found at its correct sorted position.
            if (pivotIndex == targetIndex) {
                return nums[pivotIndex];
            }

            // Target is on the right side.
            else if (pivotIndex < targetIndex) {
                left = pivotIndex + 1;
            }

            // Target is on the left side.
            else {
                right = pivotIndex - 1;
            }
        }

        return -1;
    }

    private int partition(int[] nums, int left, int right) {

        // Select a random pivot index.
        int randomIndex = ThreadLocalRandom.current()
                            .nextInt(left, right + 1);

        // Move the chosen pivot to the end.
        swap(nums, randomIndex, right);

        int pivot = nums[right];

        // Boundary for elements <= pivot.
        int smaller = left;

        // Place all elements <= pivot before the boundary.
        for (int i = left; i < right; i++) {

            if (nums[i] <= pivot) {
                swap(nums, smaller, i);
                smaller++;
            }
        }

        // Put pivot in its correct partition position.
        swap(nums, smaller, right);

        return smaller;
    }

    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

Expected optimal time

- **Average time:** \\(O(n)\\)
- **Worst-case time:** \\(O(n^2)\\)
- **Auxiliary space:** \\(O(1)\\) — iterative Quickselect, excluding the small constant amount of pivot-selection overhead.

Randomization makes consistently bad partitions unlikely, but it does not eliminate the theoretical worst case.

## **7. Detailed dry run**

Let's trace Quickselect using:

```
nums = [3,2,1,5,6,4]
k = 2
```

First calculate:

```
targetIndex = n - k
            = 6 - 2
            = 4
```

We want the element that belongs at index `4` in ascending order.

### **Partition 1 — Choose pivot = 4**

Initially:

```
[3, 2, 1, 5, 6, 4]
```

The pivot is `4`.

We move elements smaller than or equal to `4` to the left.

| Element | Comparison | Action         |
| ------- | ---------- | -------------- |
| 3       | `3 <= 4`   | Keep on left   |
| 2       | `2 <= 4`   | Keep on left   |
| 1       | `1 <= 4`   | Keep on left   |
| 5       | `5 > 4`    | Leave on right |
| 6       | `6 > 4`    | Leave on right |

After placing the pivot:

```
[3, 2, 1, 4, 6, 5]
          ↑
      pivotIndex = 3
```

Now compare:

```
pivotIndex = 3
targetIndex = 4
```

Since:

```
3 < 4
```

Our target is on the right side.

Discard the left partition, including the pivot:

```
Search range: [6, 5]
Indices:       4  5
```

### **Partition 2 — Choose pivot = 5**

Current search range:

```
[6, 5]
```

Choose pivot `5`.

Compare:

```
6 > 5
```

So `6` stays on the right.

After partitioning:

```
[5, 6]
 ↑
pivotIndex = 4
```

Now:

```
pivotIndex = 4
targetIndex = 4
```

Target found

Answer:

# **5**

Notice that we never needed to sort the entire array.

### **Understanding the three partition outcomes**

| Condition                   | What it means                        | Next action  |
| --------------------------- | ------------------------------------ | ------------ |
| `pivotIndex == targetIndex` | Found the answer                     | Return pivot |
| `pivotIndex < targetIndex`  | Target is larger in sorted position  | Search right |
| `pivotIndex > targetIndex`  | Target is smaller in sorted position | Search left  |

This is the most important logic to remember in Quickselect.

## **8. Compare all three approaches**

| Approach    | Time complexity                        | Space complexity |
| ----------- | -------------------------------------- | ---------------- |
| Sorting     | \\(O(n\log n)\\)                       | Depends on sort  |
| Min Heap    | \\(O(n\log k)\\)                       | \\(O(k)\\)       |
| Quickselect | Average \\(O(n)\\), worst \\(O(n^2)\\) | \\(O(1)\\)       |

**Which should you use in interviews?**

- Sorting: easiest to implement and explain.
- Min Heap: great when `k` is small or you need a streaming solution.
- Quickselect: excellent when you need the k-th element efficiently from an array.

## **9. Tricky parts and interview insights**

### **A. Why&#x20;**`n - k`**?**

Because Quickselect uses ascending-order indexing.

| k-th largest        | Ascending index |
| ------------------- | --------------- |
| Largest (`k=1`)     | `n-1`           |
| 2nd largest (`k=2`) | `n-2`           |
| 3rd largest (`k=3`) | `n-3`           |
| Smallest (`k=n`)    | `0`             |

Therefore:

```
int targetIndex = nums.length - k;
```

### **B. Why does the Min Heap remove the smallest element?**

We want to retain only the largest `k` elements.

Whenever the heap contains more than `k` elements, removing its smallest element ensures that the weaker candidate is discarded.

At the end, the smallest element among the retained `k` elements is the k-th largest overall.

### **C. Why doesn't Quickselect sort the whole array?**

After partitioning, the pivot is at its correct sorted position.

We only need to continue on the side containing our target index. The other side is irrelevant to the answer.

### **D. Do duplicates cause a problem?**

No. The problem counts duplicate elements separately.

For example:

```
nums = [3,2,3,1,2,4,5,5,6]
k = 4
```

Sorted descending:

```
[6,5,5,4,3,3,2,2,1]
```

The fourth largest element is `4`.

Our partition implementation uses `<= pivot`, so duplicates are handled correctly.

### **E. Why is randomized pivot selection useful?**

Suppose we repeatedly choose the largest element as the pivot in an already sorted array.

The partitions may become extremely unbalanced, resulting in \\(O(n^2)\\) time.

Random pivot selection reduces the likelihood of this happening repeatedly, giving expected \\(O(n)\\) time.

## **GitHub-ready notes — LC 215**

**Problem:** Find the k-th largest element in an unsorted array. Duplicates count as separate elements.

**Pattern:** Heap / Quickselect / Sorting

**Approach 1 — Min Heap:**

1. Create a Min Heap.
2. Insert every array element.
3. If heap size exceeds `k`, remove the smallest.
4. Return the heap root.

**Approach 2 — Quickselect:**

1. Convert k-th largest to ascending index `n-k`.
2. Choose a randomized pivot.
3. Partition the array around the pivot.
4. Compare the pivot index with the target index.
5. Continue only on the relevant side.
6. Return the pivot when its index equals the target.

**Complexity:**

| Approach    | Time               | Space          |
| ----------- | ------------------ | -------------- |
| Sorting     | \\(O(n\log n)\\)   | Sort-dependent |
| Min Heap    | \\(O(n\log k)\\)   | \\(O(k)\\)     |
| Quickselect | Average \\(O(n)\\) | \\(O(1)\\)     |

**Key Insights:**

- K-th largest corresponds to index `n-k` in ascending order.
- A Min Heap of size `k` retains the k largest elements.
- Quickselect positions the target without sorting the entire array.
- Randomized pivot selection improves expected performance.
- Duplicates count separately.

## **10. Test your understanding**

**Q1. nums = [3,2,1,5,6,4], k = 2. What is the target index for Quickselect?**

2

3

4

5



**Q2. Why do we remove the smallest element when Min Heap size exceeds k?**

To retain the k largest elements

To sort the array

To remove duplicates



**Q3. If pivotIndex < targetIndex, which side should Quickselect search?**

Left side

Right side

Both sides



**Q4. What is Quickselect's average time complexity?**

O(n²)

O(n log n)

O(n)



**Q5. In [5,5,5,5], what is the 3rd largest element?**

5

3

There is no answer



Check answers

**Recommended next problems:**

1. LC 347 — Top K Frequent Elements (HashMap + Heap / Bucket Sort).
2. LC 973 — K Closest Points to Origin (Heap / Quickselect).
3. LC 703 — Kth Largest Element in a Stream (Min Heap).

I recommend **LC 347 next**, because it builds directly on the Min Heap concept you learned here.
