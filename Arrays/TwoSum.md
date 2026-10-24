Let's apply your exact 8-step problem-solving framework to LeetCode 1: Two Sum.

The goal isn't just to understand the solution. It's to learn how to move from reading a problem to discovering the algorithm and then writing the code in the correct order.

We'll derive the solution in two stages: brute force first, then optimized.

Problem: Two Sum

Problem statement: Given an integer array nums and an integer target, return the indices of two distinct elements whose sum equals target.

Assume exactly one valid answer exists, and you cannot use the same element twice.

Example:

nums = [2, 7, 11, 15]
target = 9

Output: [0, 1]

Because nums[0] + nums[1] = 2 + 7 = 9.


---

STEP 1 — Understand the problem

What should you write in your notebook?

Item	Answer

Input	Integer array nums, integer target
Output	Two indices
Rules	Indices must be distinct; the two values must sum to target
Important detail	Return indices, not the values themselves


What should you think?

Ask yourself:

1. Am I looking for values or indices? Indices.
2. Must the elements be next to each other? No.
3. Can I use the same element twice? No.
4. Does the order of the two indices matter? Usually not, unless the problem says otherwise.

Why do this first?

Because you need to know exactly what counts as a valid answer before designing an algorithm.

Move to Step 2 when: You can explain the problem in your own words.


---

STEP 2 — Solve a small example manually

Given:

nums = [2, 7, 11, 15]
target = 9

Without thinking about code, ask yourself:

Which two numbers add up to 9?

Try the possibilities:

First number	Second number	Sum	Correct?

2	7	9	Yes
2	11	13	No
2	15	17	No


We have found the answer: indices [0, 1].

Notice how you solved it manually: you chose one number and looked for another number that would complete the target.

This observation will be useful later.

What should you write?

Expected output: [0, 1]

Reason:
nums[0] + nums[1] = 2 + 7 = 9

Why do this second?

It makes the problem concrete and gives you a test case against which you can verify your algorithm.

Move to Step 3 when: You understand how a valid answer is found.


---

STEP 3 — Derive the brute-force approach

Now ask yourself:

"How can I guarantee that I find the correct pair without missing any possibility?"

You already know the answer from your manual attempt: try pairs of elements.

Think in terms of indices

Suppose you select the first element using index i.

Which other elements can you pair it with?

All elements after it, starting from index i + 1.

For each pair, check whether:

\[
nums[i]+nums[j]=target
\]

If the equation is true, return [i, j].

Write the algorithm in plain English

1. Choose the first index i.
2. Choose a second index j, starting from i + 1.
3. Check whether the two values sum to target.
4. If they do, return their indices.
5. If no pair works, continue checking other pairs.

Now convert the idea into loop structure

for (int i = 0; i < nums.length; i++) {
    for (int j = i + 1; j < nums.length; j++) {
        // Check whether nums[i] + nums[j] == target
    }
}

Don't worry about the complete code yet.

Why two loops?

Because you need to select two indices. The outer loop chooses the first index, and the inner loop explores possible second indices.

Why does j start at i + 1?

Because we must not use the same element twice, and we don't need to check a pair twice in reverse order.

Move to Step 4 when: You can explain what both loops do and why their boundaries are correct.


---

STEP 4 — Dry-run the brute force

This is where you physically trace the algorithm on paper rather than trying to visualize the entire program in your head.

Given:

nums = [2, 7, 11, 15]
target = 9

Start with i = 0.

i	j	nums[i]	nums[j]	Sum	Decision

0	1	2	7	9	Return [0, 1]


We found the answer immediately, so the algorithm stops.

But what if the answer weren't found immediately? The outer loop would select another i, and the inner loop would examine the remaining elements.

What should you check?

- Does j start at i + 1?
- Does j stay within the array?
- Are you comparing the sum against target?
- Are you returning indices rather than values?
- Does the algorithm stop when a valid pair is found?

Why do we dry-run?

Because it verifies that your loops actually perform the operations you intended.

Move to Step 5 when: You can trace the algorithm correctly for both successful and unsuccessful comparisons.


---

STEP 5 — Analyze complexity and find the bottleneck

Now we have a correct brute-force solution.

But can we make it faster?

First, analyze how much work it does.

For an array of length \(n\), we may check approximately:

\[
\frac{n(n-1)}{2}
\]

pairs.

Therefore:

- Time complexity: \(O(n^2)\)
- Extra space: \(O(1)\)

Now ask the optimization questions

Question 1: What work am I repeating?

For each element, we repeatedly examine other elements to find the required partner.

Question 2: What information would help me find the partner immediately?

Suppose the current number is 2, and the target is 9.

Instead of searching for every possible partner, calculate the number you need:

\[
needed=target-current
\]

So:

\[
needed=9-2=7
\]

You need the value 7.

Question 3: Can I store information that lets me look up this value quickly?

Yes! A HashMap can store values we've already visited, along with their indices.

That is the key optimization idea.

Why do this step?

Because we don't want to eliminate loops randomly. We want to identify the specific work that can be avoided.

Move to Step 6 when: You can explain what information you need to store and why it makes the search faster.


---

STEP 6 — Derive the optimized approach

Now let's derive the HashMap solution instead of memorizing it.

Start with the same example

nums = [2, 7, 11, 15]
target = 9

Create an initially empty HashMap:

map = {}

We will store:

number -> index

For example, after processing the first element, the map might look like:

{2=0}

Process the array from left to right

Iteration 1: i = 0

Current number:

nums[i] = 2

Calculate the required partner:

needed = target - nums[i]
needed = 9 - 2 = 7

Ask the map:

"Have I already seen the number 7?"

No.

So store the current number and its index:

map = {2=0}

Iteration 2: i = 1

Current number:

nums[i] = 7

Calculate:

needed = 9 - 7 = 2

Ask the map:

"Have I already seen the number 2?"

Yes! The map tells us that 2 is at index 0.

The current element is at index 1.

Therefore, the answer is:

[0, 1]

We have found the answer without checking every possible pair.

Write the optimized algorithm in plain English

1. Create an empty HashMap to store previously visited values and their indices.
2. Visit each element from left to right.
3. Calculate needed = target - nums[i].
4. Check whether needed exists in the map.
5. If it exists, return the stored index and the current index.
6. Otherwise, store the current value and its index.
7. Continue until a solution is found.

Why check the map before inserting the current element?

Because the problem requires two distinct elements.

Looking for the partner among previously visited elements ensures that we don't accidentally use the current element twice.

For example:

nums = [3]
target = 6

If you inserted 3 before checking, the map would contain 3, and the algorithm might incorrectly pair the element with itself.

Checking first avoids that mistake.

Move to Step 7 when: You understand what the map stores, why the complement is calculated, and why the lookup happens before insertion.


---

STEP 7 — Dry-run the optimized approach

Now trace the complete optimized logic.

Input:

nums = [2, 7, 11, 15]
target = 9

Initially:

map = {}

i	Current number	Needed	Is needed in map?	Action

0	2	7	No	Store 2 -> 0
1	7	2	Yes	Return [0, 1]


Notice the order of operations:

1. Read the current number.
2. Calculate the complement.
3. Search for the complement.
4. Return the answer if found.
5. Otherwise, store the current number.

That sequence is essential to the algorithm.

Verify edge cases

Case 1: Negative numbers

nums = [-3, 4, 3]
target = 0

The complement of -3 is 3. When the algorithm reaches 3, it finds the previously stored -3.

Case 2: Duplicate values

nums = [3, 3]
target = 6

At index 0, store 3 -> 0.

At index 1, the needed value is 3, which is already in the map. Return [0, 1].

Case 3: No solution

If the problem guarantees a solution, this case isn't required by the stated contract. Otherwise, follow the problem's specified behavior for an unsuccessful search.

Why do this step?

Because an optimization isn't complete until you've checked that its logic and variable-update order are correct.

Move to Step 8 when: You can trace the map's contents and explain every operation without looking at the solution.


---

STEP 8 — Write the Java code in the correct order

This is where we combine everything you derived.

First, decide the code structure

Before typing Java, map each algorithm step to a code component.

Algorithm step	Code component	Why it belongs here

Receive the array and target	Method parameters	The method needs its input
Create storage for visited values	Initialize HashMap	The map must exist before lookup or insertion
Process each element	for loop	Each element must be considered
Calculate the complement	Variable assignment inside loop	It depends on the current element
Look up the complement	containsKey() inside loop	The lookup must happen for each element
Return if found	return inside the condition	We can finish as soon as a valid pair is found
Store the current value	put() inside the loop, after lookup	It becomes available to future elements
Handle no solution if necessary	After the loop	Only reached if no pair was found


This table tells you not only what to write, but why it belongs in that position.

Now write the complete code

import java.util.HashMap;
import java.util.Map;

class Solution {
    public int[] twoSum(int[] nums, int target) {

        // 1. Prepare storage for previously visited numbers
        Map<Integer, Integer> map = new HashMap<>();

        // 2. Process each element
        for (int i = 0; i < nums.length; i++) {

            // 3. Calculate the required partner
            int needed = target - nums[i];

            // 4. Check whether the partner has already appeared
            if (map.containsKey(needed)) {
                return new int[] { map.get(needed), i };
            }

            // 5. Store the current number and its index
            map.put(nums[i], i);
        }

        // 6. Fallback if the problem does not guarantee a solution
        return new int[0];
    }
}

The empty-array fallback is appropriate for this example if you choose to handle missing solutions that way. LeetCode 1 guarantees exactly one solution, so that fallback isn't reached for valid inputs.

Final complexity

- Time: \(O(n)\) average, because each element requires a constant number of average-case HashMap operations.
- Extra space: \(O(n)\), because the map may store up to \(n\) elements.

Compared with brute force:

Approach	Time	Extra space

Brute force	\(O(n^2)\)	\(O(1)\)
HashMap	\(O(n)\) average	\(O(n)\)



---

Your final Two Sum revision card

Before you move on to another problem, close this explanation and try to answer these questions from memory.

Step	Question to ask yourself

1. Understand	What are the input, output, and rules?
2. Example	Can I solve a small example manually?
3. Brute force	How can I try every valid pair?
4. Dry-run	Do my loops examine the right pairs?
5. Bottleneck	What work am I repeating?
6. Optimize	Can I store previously seen values and look up the complement?
7. Validate	Does lookup-before-insertion prevent reusing the same element?
8. Code	What must be initialized first, what belongs inside the loop, and when should I return?


Your practice task: Close this answer, open LeetCode 1, and solve it again on paper before writing Java. First derive the nested-loop solution. Then independently explain why a HashMap can eliminate the repeated search.

Don't aim to remember eight paragraphs. Aim to remember the eight questions and learn to answer them yourself. That's how you'll build a repeatable problem-solving process for unfamiliar questions, too.