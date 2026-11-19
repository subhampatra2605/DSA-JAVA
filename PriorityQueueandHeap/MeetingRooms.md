# **LeetCode 253 — Meeting Rooms II**



Medium

Intervals + Sorting + Min Heap



This problem is a natural follow-up to **LC 56 (Merge Intervals)** and **LC 435 (Non-overlapping Intervals)**.

But here, instead of merging or removing intervals, we need to determine the **minimum number of meeting rooms required so that every meeting can take place**.

The key idea:

**If a meeting starts before an existing meeting finishes, we need another room. If a meeting starts at or after an existing meeting finishes, we can reuse that room.**

We'll cover:

1. Brute force with Java code.
2. Sorting + Min Heap.
3. Two-pointer approach.
4. Detailed dry run and interview insights.

## **1. Understand the problem**

You are given an array of meeting intervals:

```
intervals[i] = [start, end]
```

Each interval represents a meeting's start and end time.

Return the minimum number of conference rooms required to hold all meetings.

**Example 1**

```
Input: [[0,30], [5,10], [15,20]]

Output: 2
```

Explanation:

- Meeting `[0,30]` occupies Room 1.
- Meeting `[5,10]` overlaps with `[0,30]`, so it needs Room 2.
- Meeting `[15,20]` can reuse Room 2 because `[5,10]` has finished.

**Meeting schedule**

Time →

**Room 1**

[0,30]

**Room 2**

[5,10]

[15,20]

Minimum rooms required = 2

**Example 2**

```
Input: [[7,10], [2,4]]

Output: 1
```

The meetings do not overlap, so only one room is required.

**Important endpoint rule:** If one meeting ends at time `10` and another starts at time `10`, the same room can be reused.

```
[2,10] and [10,15] → 1 room
```

## **2. Brute-force approach — Check every meeting**

Let's first solve this without a heap.

The idea is to find the maximum number of meetings happening simultaneously.

For every meeting:

1. Consider its start time.
2. Count how many meetings are active at that time.
3. Track the maximum number of simultaneous meetings.

Why is this valid?

The minimum number of rooms needed is exactly the maximum number of meetings that overlap at any point in time.

### **Java code — Brute force**

```
class Solution {
    public int minMeetingRooms(int[][] intervals) {

        int maxRooms = 0;

        // Consider the start time of every meeting.
        for (int i = 0; i < intervals.length; i++) {

            int currentTime = intervals[i][0];
            int activeMeetings = 0;

            // Count how many meetings are active
            // at this meeting's start time.
            for (int j = 0; j < intervals.length; j++) {

                // A meeting is active if it has started
                // and has not finished at currentTime.
                if (intervals[j][0] <= currentTime &&
                    currentTime < intervals[j][1]) {

                    activeMeetings++;
                }
            }

            // Track the maximum simultaneous meetings.
            maxRooms = Math.max(maxRooms, activeMeetings);
        }

        return maxRooms;
    }
}
```

**Complexity:**

- Time: \\(O(n^2)\\) — for every meeting's start time, we scan all meetings.
- Space: \\(O(1)\\).

This works because the maximum overlap can be found at a meeting's start time. A meeting ending at that exact time is not considered active, since its room can be reused.

## **3. Identify the bottleneck**

The brute-force approach checks every meeting against all other meetings.

For example:

```
[[0,30], [5,10], [15,20]]
```

We repeatedly scan the entire array to determine how many meetings are active.

Can we avoid this?

Notice that we don't need to know every meeting's details. We mainly need to know:

**When will the currently occupied room become available again?**

If we always know the earliest finishing meeting, we can decide whether its room can be reused.

This leads us to a Min Heap.

## **4. Recognize the pattern**

**Sorting + Min Heap + Greedy**

We use:

- Sorting to process meetings in increasing order of start time.
- Min Heap to track meeting end times.
- Greedy to reuse the room that becomes available earliest.

Why a Min Heap?

Because we need quick access to the smallest ending time.

In Java:

```
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
```

- `peek()` → earliest ending meeting.
- `poll()` → remove the earliest ending meeting.
- `offer()` → add a new meeting's ending time.

Remember: Java's `PriorityQueue` is a Min Heap by default.

## **5. Derive the optimal approach**

Let's build the logic step by step.

**Step 1 — Sort by starting time**

```
Arrays.sort(intervals, (a, b) ->
    Integer.compare(a[0], b[0]));
```

This ensures we process meetings chronologically.

**Step 2 — Create a Min Heap**

```
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
```

The heap stores the ending times of meetings currently occupying rooms.

**Step 3 — Process each meeting**

Suppose the current meeting is:

```
current = [15,20]
```

Check the earliest ending meeting:

```
minHeap.peek()
```

There are two possibilities.

**Case A: A room is available**

If:

```
minHeap.peek() <= current[0]
```

The earliest meeting has already finished.

We can reuse its room:

```
minHeap.poll();
```

**Case B: No room is available**

If:

```
minHeap.peek() > current[0]
```

All currently occupied rooms are still in use.

We need another room.

**Step 4 — Add the current meeting's ending time**

Regardless of whether we reused a room or needed a new one:

```
minHeap.offer(current[1]);
```

**Step 5 — Track the maximum heap size**

The heap size represents the number of rooms currently occupied by meetings.

The maximum heap size encountered is the minimum number of rooms required.

## **6. Optimal Java solution — Min Heap**

```
import java.util.Arrays;
import java.util.PriorityQueue;

class Solution {
    public int minMeetingRooms(int[][] intervals) {

        // Handle empty input.
        if (intervals.length == 0) {
            return 0;
        }

        // Step 1: Sort meetings by starting time.
        Arrays.sort(intervals, (a, b) ->
            Integer.compare(a[0], b[0])
        );

        // Step 2: Create a Min Heap to store
        // the ending times of occupied rooms.
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();

        int maxRooms = 0;

        // Step 3: Process meetings in chronological order.
        for (int[] meeting : intervals) {

            // If the earliest meeting has finished,
            // reuse that room.
            if (!minHeap.isEmpty() &&
                minHeap.peek() <= meeting[0]) {

                minHeap.poll();
            }

            // Allocate the current meeting to a room.
            // Store its ending time.
            minHeap.offer(meeting[1]);

            // Track the maximum number of rooms occupied.
            maxRooms = Math.max(maxRooms, minHeap.size());
        }

        return maxRooms;
    }
}
```

Optimal standard approach

**Complexity:**

- **Time:** \\(O(n\log n)\\) — sorting takes \\(O(n\log n)\\), and heap operations take \\(O(\log n)\\) per meeting.
- **Space:** \\(O(n)\\) — in the worst case, all meetings overlap and the heap stores all ending times.

### **Why do we remove only one element using&#x20;**`poll()`**?**

This is an important interview question.

We process meetings in sorted starting-time order.

Suppose:

```
Heap = [10, 20, 30]
Current meeting = [15,25]
```

The earliest meeting ends at `10`, so one room is available.

We remove `10`:

```
Heap = [20,30]
```

But `20` and `30` are still future ending times. We must not remove them because those meetings are still occupying their rooms.

We then add the current meeting's end time:

```
Heap = [20,25,30]
```

The heap now tracks three occupied rooms.

In fact, if the heap's minimum is greater than the current meeting's start, all other ending times must also be greater, so no other room is available.

## **7. Detailed dry run**

Input:

```
intervals = [[0,30], [5,10], [15,20]]
```

### **Step 1 — Sort by starting time**

```
[[0,30], [5,10], [15,20]]
```

### **Step 2 — Initialize**

```
minHeap = []
maxRooms = 0
```

**Iteration 1 — Meeting [0,30]**

Heap is empty, so we need a room.

```
offer(30)

Heap = [30]
maxRooms = 1
```

1 room occupied

**Iteration 2 — Meeting [5,10]**

Check earliest ending time:

```
peek() = 30
current start = 5

30 <= 5 → false
```

The existing meeting hasn't finished. We need another room.

```
offer(10)

Heap = [10,30]
maxRooms = 2
```

2 rooms occupied

**Iteration 3 — Meeting [15,20]**

Check earliest ending time:

```
peek() = 10
current start = 15

10 <= 15 → true
```

The meeting ending at `10` has finished. Reuse its room.

First:

```
poll()

Heap = [30]
```

Then add the current meeting's ending time:

```
offer(20)

Heap = [20,30]
maxRooms = 2
```

2 rooms occupied

Final answer:

# **2**

### **Important dry run — Multiple rooms becoming available**

Consider:

```
intervals = [[0,5], [1,6], [2,7], [5,8]]
```

Sorted order remains the same.

| Current meeting | Heap before processing | Action          | Heap after processing |
| --------------- | ---------------------- | --------------- | --------------------- |
| `[0,5]`         | `[]`                   | Add 5           | `[5]`                 |
| `[1,6]`         | `[5]`                  | Add 6           | `[5,6]`               |
| `[2,7]`         | `[5,6]`                | Add 7           | `[5,6,7]`             |
| `[5,8]`         | `[5,6,7]`              | Remove 5, add 8 | `[6,7,8]`             |

At time `5`, the first meeting has finished, so we reuse its room.

Answer: `3`.

Notice that we remove only one ending time because only one room has become available at that point.

## **8. Alternative optimal approach — Two Pointers**

There is another way to solve this problem without using a PriorityQueue.

Instead of tracking complete meeting intervals, separate all start times and end times into two arrays.

### **Step 1 — Create separate arrays**

For:

```
intervals = [[0,30], [5,10], [15,20]]
```

We get:

```
starts = [0,5,15]
ends   = [30,10,20]
```

Sort both independently:

```
starts = [0,5,15]
ends   = [10,20,30]
```

### **Step 2 — Use two pointers**

Maintain:

- `startPointer`: points to the next meeting starting.
- `endPointer`: points to the next meeting ending.
- `rooms`: currently occupied rooms.
- `maxRooms`: maximum rooms required.

At every step, compare:

```
starts[startPointer]
ends[endPointer]
```

Two possibilities:

**Case A: A meeting starts before the earliest meeting ends**

```
if (starts[startPointer] < ends[endPointer])
```

We need a new room.

```
rooms++;
startPointer++;
```

**Case B: A meeting ends before or exactly when another starts**

```
else
```

A room becomes available.

```
rooms--;
endPointer++;
```

Notice the strict `<` in Case A.

If start time equals end time, we process the ending first because the room can be reused.

### **Java code — Two Pointers**

```
import java.util.Arrays;

class Solution {
    public int minMeetingRooms(int[][] intervals) {

        int n = intervals.length;

        if (n == 0) {
            return 0;
        }

        // Separate starting and ending times.
        int[] starts = new int[n];
        int[] ends = new int[n];

        for (int i = 0; i < n; i++) {
            starts[i] = intervals[i][0];
            ends[i] = intervals[i][1];
        }

        // Sort both arrays independently.
        Arrays.sort(starts);
        Arrays.sort(ends);

        int startPointer = 0;
        int endPointer = 0;

        int rooms = 0;
        int maxRooms = 0;

        // Process all meeting start times.
        while (startPointer < n) {

            // A new meeting starts before the
            // earliest ongoing meeting finishes.
            if (starts[startPointer] < ends[endPointer]) {

                // Allocate another room.
                rooms++;

                // Track maximum simultaneous rooms.
                maxRooms = Math.max(maxRooms, rooms);

                startPointer++;
            }

            // A meeting has ended, so its room is available.
            else {

                rooms--;
                endPointer++;
            }
        }

        return maxRooms;
    }
}
```

**Complexity:**

- **Time:** \\(O(n\log n)\\) — sorting dominates; pointer traversal is \\(O(n)\\).
- **Space:** \\(O(n)\\) — two separate arrays.

### **Min Heap vs Two Pointers**

| Feature        | Min Heap                      | Two Pointers                   |
| -------------- | ----------------------------- | ------------------------------ |
| Sorting        | By start time                 | Start and end times separately |
| Tracks         | Active meeting end times      | Next start and next end        |
| Main operation | `peek()`, `poll()`, `offer()` | Pointer movement               |
| Time           | \\(O(n\log n)\\)              | \\(O(n\log n)\\)               |
| Space          | \\(O(n)\\)                    | \\(O(n)\\)                     |

Both are optimal. The Min Heap approach is especially useful when you need to track which room becomes available, while the two-pointer approach is simpler when you only need the number of rooms.

## **9. Tricky parts and interview insights**

### **A. Why&#x20;**`peek() <= meeting[0]`**&#x20;in the Min Heap approach?**

Because a meeting ending at the exact moment another starts can reuse the same room.

```
[1,5] and [5,10]
```

Only one room is required.

### **B. Why&#x20;**`starts[startPointer] < ends[endPointer]`**&#x20;in Two Pointers?**

The strict comparison ensures that when start and end times are equal, we process the ending first.

Using `<=` here would incorrectly allocate another room for meetings that can reuse the same room.

### **C. Why do we sort meetings by start time in the Min Heap approach?**

We need to know which meeting starts next, so we process them chronologically.

The heap independently tells us which occupied room becomes available earliest.

### **D. Why is the answer the maximum heap size?**

The heap stores one ending time for each room currently occupied.

Whenever we reuse a room, one ending time is removed before the new meeting is inserted.

Therefore, the maximum heap size represents the maximum number of rooms needed simultaneously.

### **E. Difference between LC 56, LC 435 and LC 253**

| Problem | Objective                                     | Pattern            |
| ------- | --------------------------------------------- | ------------------ |
| LC 56   | Merge overlapping intervals                   | Sorting + Greedy   |
| LC 435  | Remove minimum intervals to eliminate overlap | Greedy + Sorting   |
| LC 253  | Find minimum rooms for all meetings           | Sorting + Min Heap |

## **GitHub-ready notes — LC 253**

**Problem:** Find the minimum number of meeting rooms required to accommodate all meetings.

**Approach 1 — Brute Force:**

1. Consider every meeting's starting time.
2. Count meetings satisfying `start <= currentTime < end`.
3. Track the maximum simultaneous meetings.
4. Return the maximum count.

**Time:** \\(O(n^2)\\)

**Space:** \\(O(1)\\)

**Approach 2 — Min Heap:**

1. Sort meetings by starting time.
2. Create a Min Heap to store occupied room ending times.
3. For every meeting, check the earliest ending time.
4. If `peek() <= currentStart`, remove it and reuse that room.
5. Insert the current meeting's ending time.
6. Track the maximum heap size.

**Approach 3 — Two Pointers:**

1. Extract and independently sort start and end times.
2. Compare the next start and end.
3. If start is strictly smaller, allocate a room.
4. Otherwise, release a room.
5. Track the maximum active rooms.

**Pattern:** Intervals + Sorting + Min Heap / Two Pointers

**Time Complexity:** \\(O(n\log n)\\) for both optimal approaches.

**Space Complexity:** \\(O(n)\\).

**Key Insights:**

- Minimum rooms = maximum number of simultaneous meetings.
- Min Heap stores the ending times of occupied rooms.
- `peek()` gives the earliest room availability.
- `poll()` releases a room.
- `offer()` allocates the current meeting.
- Endpoint equality means a room can be reused.

## **10. Test your understanding**

**Q1. Meetings: [1,5] and [5,10]. How many rooms are required?**

1

2

3



**Q2. What does the Min Heap store?**

Meeting starting times

Meeting ending times of occupied rooms

Meeting durations



**Q3. Why do we use peek() before poll()?**

To check whether the earliest meeting has finished before releasing its room

To sort the heap

To remove every meeting



**Q4. In the two-pointer approach, why do we use starts[i] < ends[j] instead of <=?**

To handle equal start and end times by releasing the room first

To count duplicate meetings

To make sorting faster



**Q5. What does the maximum heap size represent?**

Total number of meetings

Maximum number of simultaneous meetings

Number of meetings without overlap



Check answers

**Recommended next:** LC 2406 — Divide Intervals Into Minimum Number of Groups. It is extremely similar to LC 253 and will reinforce the Min Heap and interval-scheduling concepts.
