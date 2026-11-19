# **LeetCode 621 — Task Scheduler**



Medium

Greedy + Heap / Counting



This is an important greedy problem involving **frequency counting, scheduling, and priority queues**.

The key challenge is not simply executing all tasks. We must arrange them so that the same task is separated by at least `n` intervals.

**Core idea:** Execute the most frequent available task first, because delaying a high-frequency task can make it harder to fit all its remaining executions into the schedule.

## **1. Understand the problem**

You are given:

- An array `tasks`, where each task is represented by an uppercase English letter (`A` to `Z`).
- An integer `n`, representing the cooldown period between two executions of the same task.

Return the minimum number of intervals required to complete all tasks.

Each interval can contain:

- One task, or
- An idle period.

**Example 1**

```
Input:
tasks = ["A","A","A","B","B","B"]
n = 2

Output: 8
```

One valid schedule:

```
A → B → idle → A → B → idle → A → B
```

Total intervals:

```
A, B, idle, A, B, idle, A, B
             8 intervals
```

The same task must have at least two intervals between consecutive executions.

**Example 2**

```
Input:
tasks = ["A","A","A","B","B","B"]
n = 0

Output: 6
```

No cooldown is required, so all six tasks can execute consecutively.

**Example 3**

```
Input:
tasks = ["A","A","A","B","B","B"]
n = 1

Output: 6
```

A valid schedule:

```
A → B → A → B → A → B
```

No idle time is needed.

## **2. Brute-force approach — Backtracking simulation**

A direct brute-force approach would try different possible task orders and find the minimum valid schedule.

At every interval:

1. Choose a task whose remaining frequency is greater than zero.
2. Check whether its cooldown has finished.
3. Execute it and update its next available time.
4. If no task is available, execute an idle interval.
5. Explore possible choices and find the minimum completion time.

This becomes a scheduling/backtracking problem with many possible execution orders.

A simple recursive formulation would need to track:

- Remaining frequency of every task.
- Cooldown or next available time for each task.
- Current time.

**Complexity:** Exponential in the number of scheduling decisions in a straightforward backtracking implementation, with additional state for frequencies and cooldowns.

For this problem, a frequency-based greedy strategy gives us a much simpler solution.

## **3. Identify the bottleneck**

The brute-force approach tries many task orders.

But observe something important:

If one task appears much more frequently than all other tasks, it determines how much cooldown space we need.

Example:

```
tasks = [A,A,A,A, B,B, C]
n = 2
```

Task frequencies:

```
A = 4
B = 2
C = 1
```

We must execute `A` four times, with at least two intervals between consecutive `A` executions.

One valid schedule is:

```
A → B → C → A → B → idle → A → idle → idle → A
```

Total = 10 intervals.

The challenge is to arrange other tasks inside the cooldown gaps so that we minimize idle time.

This suggests that **task frequencies**, rather than the original task order, are what matter.

## **4. Recognize the pattern**

**Greedy + Frequency Counting + Priority Queue**

There are two important ways to solve LC 621:

1. **Max Heap simulation:** Repeatedly execute the currently most frequent available task.
2. **Mathematical formula:** Calculate the minimum schedule length directly from the maximum frequency.

We'll understand both.

## **5. Derive the optimal approaches**

### **Approach A — Max Heap simulation**

A Max Heap allows us to retrieve the task with the highest remaining frequency.

The strategy:

1. Count the frequency of each task.
2. Insert all non-zero frequencies into a Max Heap.
3. In each scheduling round, execute up to `n + 1` different tasks.
4. Temporarily remove executed tasks from the heap.
5. If a task still has remaining occurrences, put it back after the round.
6. If the heap is empty before the round ends, add idle intervals only when tasks still remain.

**Why&#x20;**`n + 1`**&#x20;tasks per round?**

Suppose `n = 2`.

If we execute `A`, we need two intervals before executing `A` again.

So one complete scheduling cycle can contain:

```
A → B → C
```

That's `n + 1 = 3` positions.

Within these three positions, each task can appear at most once.

### **Java code — Max Heap**

```
import java.util.*;

class Solution {
    public int leastInterval(char[] tasks, int n) {

        // Step 1: Count frequency of each task.
        int[] frequency = new int[26];

        for (char task : tasks) {
            frequency[task - 'A']++;
        }

        // Step 2: Create a Max Heap.
        // Highest remaining frequency comes first.
        PriorityQueue<Integer> maxHeap =
            new PriorityQueue<>(Collections.reverseOrder());

        // Add only tasks that actually exist.
        for (int count : frequency) {
            if (count > 0) {
                maxHeap.offer(count);
            }
        }

        int intervals = 0;

        // Continue until every task is completed.
        while (!maxHeap.isEmpty()) {

            // Store tasks temporarily during this round.
            List<Integer> usedTasks = new ArrayList<>();

            // One round can contain at most n + 1 tasks.
            int cycle = n + 1;

            // Execute up to n + 1 different tasks.
            for (int i = 0; i < cycle; i++) {

                // If a task is available, execute it.
                if (!maxHeap.isEmpty()) {

                    int remaining = maxHeap.poll();

                    // One execution is completed.
                    remaining--;

                    // Store it if more executions are needed.
                    if (remaining > 0) {
                        usedTasks.add(remaining);
                    }

                    // One interval was used.
                    intervals++;
                }

                // If no task is available, but tasks remain
                // in this round, count an idle interval.
                else if (!usedTasks.isEmpty()) {
                    intervals++;
                }

                // If no tasks remain anywhere, stop.
                else {
                    break;
                }
            }

            // Return unfinished tasks to the Max Heap.
            for (int remaining : usedTasks) {
                maxHeap.offer(remaining);
            }
        }

        return intervals;
    }
}
```

**Complexity:**

Let `T` be the total number of tasks and `U` be the number of distinct task types (at most 26).

- Time: \\(O(T\log U)\\), because each task execution updates the heap.
- Space: \\(O(U)\\), for the heap and temporary list.

Since there are only 26 possible uppercase task types, `U` is bounded by 26.

### **Approach B — Greedy mathematical formula (Most optimal)**

We can solve this problem without actually simulating the schedule.

We only need to know:

- `maxFreq`: Highest frequency of any task.
- `maxCount`: Number of task types having that highest frequency.
- `totalTasks`: Total number of tasks.

The minimum number of intervals is:

\\[ \boxed{\max\left(T,\\;(F-1)(n+1)+C\right)} \\]

Where:

- \\(T\\) = total number of tasks.
- \\(F\\) = maximum task frequency.
- \\(C\\) = number of task types with frequency \\(F\\).
- \\(n\\) = cooldown period.

Let's understand where this formula comes from.

#### **Step 1 — Find the most frequent task**

Consider:

```
tasks = [A,A,A,B,B,B]
n = 2
```

Frequency:

```
A = 3
B = 3
```

Therefore:

```
maxFreq = 3
maxCount = 2
```

Both `A` and `B` have the maximum frequency.

#### **Step 2 — Create the scheduling structure**

Imagine we arrange the most frequent task first:

```
A → _ → _ → A → _ → _ → A
```

Since `A` appears three times, there are two gaps between its executions.

Each gap must have two intervals because `n = 2`.

So:

```
A → _ → _ → A → _ → _ → A
```

This gives us:

\\[ (F-1)(n+1)+1 \\]

For this example:

\\[ (3-1)(2+1)+1=7 \\]

But there are two tasks tied for maximum frequency: `A` and `B`.

We need to accommodate both at the end:

```
A → B → _ → A → B → _ → A → B
```

Therefore, the final frame length is:

\\[ (F-1)(n+1)+C \\]

\\[ =(3-1)(2+1)+2=8 \\]

#### **Step 3 — Why do we take the maximum with total tasks?**

Consider:

```
tasks = [A,A,A,B,B,B,C,D,E]
n = 2
```

Here:

```
maxFreq = 3
maxCount = 2
totalTasks = 9
```

Formula based on the most frequent tasks:

\\[ (3-1)(2+1)+2=8 \\]

But we have nine actual tasks.

We cannot finish nine tasks in eight intervals, even if there are no idle intervals.

Therefore:

\\[ \text{Answer}=\max(9,8)=9 \\]

This gives us the complete formula:

\\[ \boxed{\text{Answer}=\max(T,(F-1)(n+1)+C)} \\]

## **6. Optimal Java solution — Frequency Counting**

```
class Solution {
    public int leastInterval(char[] tasks, int n) {

        // Step 1: Count the frequency of every task.
        int[] frequency = new int[26];

        for (char task : tasks) {
            frequency[task - 'A']++;
        }

        // Step 2: Find the maximum frequency.
        int maxFreq = 0;

        for (int count : frequency) {
            maxFreq = Math.max(maxFreq, count);
        }

        // Step 3: Count how many tasks have max frequency.
        int maxCount = 0;

        for (int count : frequency) {
            if (count == maxFreq) {
                maxCount++;
            }
        }

        // Step 4: Calculate the minimum intervals
        // required by the most frequent tasks.
        int intervals = (maxFreq - 1) * (n + 1) + maxCount;

        // Step 5: We cannot use fewer intervals than
        // the total number of actual tasks.
        return Math.max(tasks.length, intervals);
    }
}
```

Optimal

**Complexity:**

- **Time:** \\(O(T+26)=O(T)\\), where \\(T\\) is the number of tasks.
- **Space:** \\(O(26)=O(1)\\), because the frequency array has a fixed size.

This is the most efficient solution for LC 621.

## **7. Detailed dry run — Mathematical approach**

Input:

```
tasks = [A,A,A,B,B,B]
n = 2
```

### **Step 1 — Build the frequency array**

```
A = 3
B = 3
All other tasks = 0
```

### **Step 2 — Find maximum frequency**

```
maxFreq = 3;
```

### **Step 3 — Count tasks with maximum frequency**

Both `A` and `B` have frequency 3.

```
maxCount = 2;
```

### **Step 4 — Apply the formula**

```
intervals = (maxFreq - 1) * (n + 1) + maxCount

          = (3 - 1) * (2 + 1) + 2

          = 2 * 3 + 2

          = 8
```

### **Step 5 — Compare with total tasks**

```
tasks.length = 6

answer = max(6, 8)
       = 8
```

**One valid schedule**

# **A**

&#x20;1&#x20;

# **B**

&#x20;2&#x20;

# **–**

&#x20;3&#x20;

# **A**

&#x20;4&#x20;

# **B**

&#x20;5&#x20;

# **–**

&#x20;6&#x20;

# **A**

&#x20;7&#x20;

# **B**

&#x20;8&#x20;

**Total intervals**

# **8**

### **Another important dry run — No idle time required**

Input:

```
tasks = [A,A,A,B,B,C,C]
n = 2
```

Frequencies:

```
A = 3
B = 2
C = 2
```

Therefore:

```
maxFreq = 3
maxCount = 1
totalTasks = 7
```

Calculate:

```
intervals = (3 - 1) * (2 + 1) + 1
          = 7
```

Answer:

```
max(7, 7) = 7
```

One valid schedule:

```
A → B → C → A → B → C → A
```

No idle intervals are required.

## **8. Tricky parts and interview insights**

### **A. Why do we use&#x20;**`maxCount`**?**

Consider:

```
tasks = [A,A,A,B,B,B,C,C]
n = 2
```

Here:

```
A = 3
B = 3
C = 2

maxFreq = 3
maxCount = 2
```

Both `A` and `B` have the highest frequency.

A possible arrangement is:

```
A → B → C → A → B → C → A → B
```

We need to account for both maximum-frequency tasks in the final group. That's why we add `maxCount`, not just `1`.

### **B. Why is the formula&#x20;**`(maxFreq - 1) * (n + 1) + maxCount`**?**

- `maxFreq - 1`: Number of gaps between repeated executions of the most frequent tasks.
- `n + 1`: Minimum spacing cycle for a task and its cooldown.
- `maxCount`: Number of maximum-frequency tasks occupying the final group.

### **C. Why do we use&#x20;**`Math.max(tasks.length, intervals)`**?**

The formula gives the minimum schedule length imposed by the most frequent tasks.

But other tasks might fill all the idle positions, or even require more intervals.

The answer can never be less than the number of actual tasks.

### **D. Why does&#x20;**`n = 0`**&#x20;work?**

When there is no cooldown:

```
tasks = [A,A,A,B,B]
n = 0
```

Formula:

```
maxFreq = 3
maxCount = 1

intervals = (3 - 1) * (0 + 1) + 1
          = 3

answer = max(5, 3)
       = 5
```

All tasks execute consecutively. Correct.

### **E. Heap vs formula — which should you remember?**

| Feature            | Max Heap                          | Mathematical formula     |
| ------------------ | --------------------------------- | ------------------------ |
| Technique          | Greedy simulation                 | Frequency counting       |
| Time               | \\(O(T\log U)\\)                  | \\(O(T)\\)               |
| Space              | \\(O(U)\\)                        | \\(O(1)\\)               |
| Simulates schedule | Yes, approximately through rounds | No                       |
| Best use           | Understanding scheduling          | Final optimized solution |

## **GitHub-ready notes — LC 621**

**Problem:** Find the minimum intervals required to execute all tasks, ensuring that identical tasks have at least `n` intervals between executions.

**Pattern:** Greedy + Frequency Counting + Heap

**Approach 1 — Max Heap:**

1. Count the frequency of each task.
2. Insert frequencies into a Max Heap.
3. Execute up to `n + 1` different tasks per cycle.
4. Decrease their frequencies.
5. Reinsert unfinished tasks after the cycle.
6. Count idle intervals whenever no task is available but unfinished tasks remain.

**Approach 2 — Mathematical formula:**

\\[ \boxed{\max(T,(F-1)(n+1)+C)} \\]

Where:

- `T` = total tasks.
- `F` = maximum frequency.
- `C` = number of tasks having maximum frequency.
- `n` = cooldown period.

**Time Complexity:** \\(O(T)\\) for the frequency-counting solution.

**Space Complexity:** \\(O(1)\\), fixed 26-element array.

**Key Insights:**

- The most frequent task determines the minimum spacing requirement.
- `maxCount` accounts for multiple tasks tied at maximum frequency.
- Other tasks can fill idle positions.
- The answer cannot be smaller than the total number of tasks.
- The mathematical formula avoids explicit schedule simulation.

## **9. Test your understanding**

**Q1. tasks = [A,A,A,B,B,B], n = 2. What is the minimum interval count?**

6

7

8

9



**Q2. What does maxCount represent?**

Total number of tasks

Number of task types having the maximum frequency

Number of idle intervals



**Q3. Why do we take max(tasks.length, formula)?**

Because the schedule must execute every actual task

To remove duplicates

To make the formula linear



**Q4. What is the time complexity of the frequency-counting solution?**

O(T²)

O(T log T)

O(T)



**Q5. If n = 0, what is the answer?**

Always 0

Always tasks.length

Always maximum frequency



Check answers

**Recommended next:** LC 347 — Top K Frequent Elements. It combines HashMap frequency counting with the PriorityQueue concepts from LC 215.
