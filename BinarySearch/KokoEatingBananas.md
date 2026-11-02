LeetCode 875 — Koko Eating Bananas

Medium
 Binary Search on Answer

Pattern: Binary Search on Answer

This problem is important because it introduces a different application of binary search.

In LC 704, 33 and 153, we performed binary search on array indices.

In LC 875, we perform binary search on the possible answer itself — Koko's eating speed.

1. Understand the Problem
Koko has n piles of bananas. The i-th pile contains piles[i] bananas.

She has h hours to eat all the bananas.

Rules:
1. Koko chooses an eating speed k bananas per hour.
2. Every hour, she chooses one pile and eats k bananas from it.
3. If the pile contains fewer than k bananas, she eats all of them and doesn't eat anything else during that hour.
4. She wants to eat all bananas within h hours.
5. We need to find the minimum possible eating speed k.
Example 1
Input:
piles = [3,6,7,11]
h = 8

Output: 4


Let's understand why.

If Koko eats 4 bananas per hour:
Pile	Bananas	Hours required
1	3	1
2	6	2
3	7	2
4	11	3
Total	27	8

She finishes in exactly 8 hours.

Therefore, the answer is 4.
Example 2
piles = [30,11,23,4,20]
h = 5

Output: 30


She has only 5 hours and there are 5 piles.

She must finish one pile per hour, so she needs to eat at the speed of the largest pile: 30.

2. Brute-Force Approach
Idea
What is the minimum possible eating speed?

At least 1 banana per hour.

What is the maximum possible eating speed?

The size of the largest pile.

Why?

If Koko eats at the speed of the largest pile, she can finish every pile in one hour per pile.

Therefore:
\[
1 \leq k \leq \max(piles)
\]

We can try every possible speed:
k = 1
k = 2
k = 3
k = 4
...
k = max(piles)


For each speed, calculate how many hours Koko needs.

The first speed that allows her to finish within h hours is our answer.
How do we calculate the hours?
Suppose:
pile = 11
k = 4


Koko eats:
Hour 1 → 4 bananas (7 remaining)
Hour 2 → 4 bananas (3 remaining)
Hour 3 → 3 bananas (finished)


Hours required:
\[
\left\lceil \frac{11}{4} \right\rceil = 3
\]

In Java, we can calculate ceiling division using:
int hours = (pile + k - 1) / k;


Java Code — Brute Force
class Solution {
    public int minEatingSpeed(int[] piles, int h) {

        int maxPile = 0;

        // Find the maximum pile size.
        for (int pile : piles) {
            maxPile = Math.max(maxPile, pile);
        }

        // Try every possible eating speed.
        for (int k = 1; k <= maxPile; k++) {

            int totalHours = 0;

            // Calculate hours required at speed k.
            for (int pile : piles) {
                totalHours += (pile + k - 1) / k;
            }

            // Return the first speed that finishes within h hours.
            if (totalHours <= h) {
                return k;
            }
        }

        return maxPile;
    }
}


Complexity
Let:
- n = number of piles
- M = maximum pile size
Complexity	Value
Time	O(n × M)
Space	O(1)

This is inefficient when pile sizes are very large.

3. Identify the Bottleneck
The brute-force solution checks every possible speed.

Suppose:
piles = [3,6,7,11]


Possible speeds:
1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11


But do we really need to check every speed?

Let's observe something important.

If Koko can finish all piles at speed 4, she can also finish them at speed 5, 6, 7, and so on.

Similarly, if she cannot finish within 8 hours at speed 3, she also cannot finish at speed 2 or 1.

This creates a monotonic pattern.

4. Recognize the Pattern — Binary Search on Answer
Let's examine the possible speeds for:
piles = [3,6,7,11]
h = 8


Possible eating speeds
Speed	Hours needed	Feasible?
1	27	No
2	15	No
3	10	No
4	8	Yes
5	7	Yes
6	6	Yes
7	5	Yes
8	5	Yes
9	5	Yes
10	5	Yes
11	4	Yes


Invalid: 1–3

Valid: 4–11


Once a speed becomes feasible, every higher speed is also feasible.

Our objective is to find the first feasible speed.

This is a classic Binary Search on Answer problem.

The search space is:
\[
[1,\max(piles)]
\]


5. Derive the Optimal Approach
We need three things:
1. Search boundaries.
2. A function to calculate the hours required for a given speed.
3. Binary search to find the minimum feasible speed.
Step 1: Initialize search boundaries
int left = 1;
int right = maxPile;


Why?
- Minimum speed = 1.
- Maximum useful speed = largest pile.
Step 2: Calculate middle speed
int mid = left + (right - left) / 2;


This is the speed we are testing.
Step 3: Calculate total hours
For each pile:
\[
hours = \left\lceil \frac{pile}{mid} \right\rceil
\]

Java implementation:
int totalHours = 0;

for (int pile : piles) {
    totalHours += (pile + mid - 1) / mid;
}


Let's understand why this formula works.

Suppose:
pile = 11
mid = 4


Then:
(11 + 4 - 1) / 4
= 14 / 4
= 3


Java integer division gives 3, which is the correct ceiling result.
Step 4: Decide which direction to search
There are two possibilities.

Case A: Koko finishes within the available hours
if (totalHours <= h)


The speed is feasible.

But we want the minimum feasible speed, so we try a smaller speed:
right = mid;


Why mid, not mid - 1?

Because mid itself might be the minimum answer. We must keep it as a candidate.

Case B: Koko needs more than h hours
else


The speed is too slow.

We must increase the speed:
left = mid + 1;


Because mid is definitely invalid, we can eliminate it.

6. Optimal Java Solution
Optimal
class Solution {
    public int minEatingSpeed(int[] piles, int h) {

        int left = 1;
        int right = 0;

        // Find the maximum pile.
        for (int pile : piles) {
            right = Math.max(right, pile);
        }

        // Binary search on possible eating speeds.
        while (left < right) {

            int mid = left + (right - left) / 2;

            int totalHours = 0;

            // Calculate total hours required at speed mid.
            for (int pile : piles) {

                // Ceiling division: ceil(pile / mid).
                totalHours += (pile + mid - 1) / mid;
            }

            // If speed is feasible, try a smaller speed.
            if (totalHours <= h) {

                right = mid;

            } else {

                // Speed is too slow, so increase it.
                left = mid + 1;
            }
        }

        // left == right: minimum feasible eating speed.
        return left;
    }
}


Complexity
Let n be the number of piles and M the maximum pile size.
- Time: O(n log M) — binary search performs O(log M) iterations, each checking n piles.
- Space: O(1) — only a few variables are used.

7. Detailed Dry Run
Let's dry-run:
piles = [3,6,7,11]
h = 8


Initial boundaries:
left = 1
right = 11


Iteration 1
Calculate:
mid = 1 + (11 - 1) / 2
mid = 6


Koko eats at speed 6.

Calculate hours:
Pile	Calculation	Hours
3	ceil(3/6)	1
6	ceil(6/6)	1
7	ceil(7/6)	2
11	ceil(11/6)	2
Total		6

totalHours = 6
h = 8


Since:
6 <= 8


Speed 6 is feasible.

But can we do better with a smaller speed?
right = mid;
right = 6;


New range:
left = 1
right = 6


Iteration 2
mid = 1 + (6 - 1) / 2
mid = 3


Calculate hours:
Pile	Calculation	Hours
3	ceil(3/3)	1
6	ceil(6/3)	2
7	ceil(7/3)	3
11	ceil(11/3)	4
Total		10

totalHours = 10
h = 8


Since:
10 > 8


Speed 3 is too slow.

We need a higher speed:
left = mid + 1;
left = 4;


New range:
left = 4
right = 6


Iteration 3
mid = 4 + (6 - 4) / 2
mid = 5


Calculate hours:
Pile	Calculation	Hours
3	ceil(3/5)	1
6	ceil(6/5)	2
7	ceil(7/5)	2
11	ceil(11/5)	3
Total		8

Since:
8 <= 8


Speed 5 is feasible.

Try a smaller speed:
right = mid;
right = 5;


New range:
left = 4
right = 5


Iteration 4
mid = 4 + (5 - 4) / 2
mid = 4


Calculate hours:
Pile	Calculation	Hours
3	ceil(3/4)	1
6	ceil(6/4)	2
7	ceil(7/4)	2
11	ceil(11/4)	3
Total		8

Since:
8 <= 8


Speed 4 is feasible.
right = mid;
right = 4;


Now:
left = 4
right = 4


Loop terminates.
Answer: 4
Complete Dry Run Table
Iteration	Left	Right	Mid	Hours	Decision
1	1	11	6	6	Feasible → right = 6
2	1	6	3	10	Invalid → left = 4
3	4	6	5	8	Feasible → right = 5
4	4	5	4	8	Feasible → right = 4

Final:
left = right = 4



8. Tricky Parts You Must Understand
A. Why right = maxPile?
Suppose:
piles = [3,6,7,11]


The largest pile contains 11 bananas.

If Koko eats 11 bananas per hour:
Pile 3  → 1 hour
Pile 6  → 1 hour
Pile 7  → 1 hour
Pile 11 → 1 hour


She needs only 4 hours.

So the largest pile is always a feasible upper bound when h >= piles.length, as guaranteed by the problem constraints.
B. Why right = mid and not mid - 1?
This is a minimum-answer binary search.

When a speed is feasible, it might be the smallest feasible speed.

Therefore, we must keep mid in our search range:
right = mid;


Compare:
Problem	When condition succeeds	Pointer update
LC 33 — Search target	Target belongs to left half	right = mid - 1
LC 153 — Find minimum	Minimum may be at mid	right = mid
LC 875 — Koko	Speed is feasible	right = mid

C. Why use left < right?
We are searching for the minimum feasible speed.

When:
left == right


Only one possible answer remains.

No further iteration is needed.
D. Why is this called Binary Search on Answer?
In LC 704:
Search space = array indices


In LC 875:
Search space = possible eating speeds


We are not searching through the array for a particular element.

We are searching through a range of possible answers and using a feasibility check to decide which half to eliminate.
E. Important Java overflow consideration
For the given LeetCode constraints, the maximum pile size is large enough that this expression can potentially overflow an int:
(pile + mid - 1) / mid


A safer implementation uses long:
long hours = 0;

for (int pile : piles) {
    hours += (pile + (long) mid - 1) / mid;
}


This is especially important when adapting the algorithm to larger constraints.

Compare LC 704, LC 153 and LC 875
Problem	Search space	What are we searching for?
LC 704	Array indices	Target index
LC 153	Array indices	Minimum element
LC 875	Eating speed	Minimum feasible speed

The common concept is binary search, but the search space and condition being checked are different.

GitHub-Ready Notes
Problem: LeetCode 875 — Koko Eating Bananas

Pattern:
Binary Search on Answer

Approach:
1. Find maximum pile size.
2. Set search range: left = 1, right = maxPile.
3. Calculate mid as the candidate eating speed.
4. Calculate total hours:
   sum of ceil(pile / mid) for every pile.
5. If totalHours <= h:
   Speed is feasible. Search smaller speeds using right = mid.
6. Otherwise:
   Speed is too slow. Search larger speeds using left = mid + 1.
7. Return left.

Ceiling Division:
(pile + mid - 1) / mid

Time Complexity:
O(n * log M), where M is maximum pile size.

Space Complexity:
O(1)

Key Insight:
Feasibility is monotonic:
If speed k works, every speed greater than k also works.
Find the first feasible speed using binary search.

Important:
right = mid because mid may be the minimum valid answer.
Use long for totalHours to avoid integer overflow.



Test Your Understanding
Try these questions before moving forward.
1. If Koko can finish at speed 5, what can we conclude about speed 6?

Speed 6 is also feasible

Speed 6 is not feasible

Nothing can be concluded



2. piles = [3,6,7,11], speed = 4. How many hours does the pile of 11 require?

2

3

4



3. If totalHours <= h, which pointer should change?

left = mid + 1

right = mid

right = mid - 1



4. What is the time complexity?

O(n)

O(M)

O(n log M)


Check answers
Next recommended problem: LC 1011 — Capacity To Ship Packages Within D Days.

It uses exactly the same Binary Search on Answer pattern, but instead of finding the minimum eating speed, we find the minimum ship capacity.
