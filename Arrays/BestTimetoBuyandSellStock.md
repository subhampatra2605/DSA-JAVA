LeetCode 121: Best Time to Buy and Sell Stock

Let's solve this using your same 8-step framework, just as we did for Two Sum.

This time, pay special attention to how you move from brute force to the optimal approach, and how that reasoning determines the order of your Java code.

Step 1 — Understand the problem

You are given an array prices, where prices[i] is the stock price on day i.

You must choose:

- One day to buy the stock.
- A later day to sell the stock.

Your goal is to maximize your profit.

Important rules:

1. You must buy before you sell.
2. You can make only one buy and one sell.
3. If no positive profit is possible, return 0.

Example:

prices = [7, 1, 5, 3, 6, 4]

Output: 5

Buy at price 1 on day 2 and sell at price 6 on day 5.

Profit = 6 - 1 = 5.

Write in your notebook:

Item	Answer

Input	Integer array prices
Output	Maximum profit
Rule	Buy before selling
Objective	Maximize sellPrice - buyPrice
Special case	If no profit is possible, return 0


Move to Step 2 when: You understand what makes a transaction valid.


---

Step 2 — Solve a small example manually

Consider:

prices = [7, 2, 5, 1, 6]

Try buying and selling on different days.

Buy price	Sell price	Profit

7	2	Invalid: selling comes later at a lower price
2	5	3
2	6	4
1	6	5


The maximum profit is 5.

Notice something important: the selling day must come after the buying day.

You cannot buy at 6 and then pretend you bought earlier at 1.

Write down:

Expected output = 5
Reason = Buy at 1, sell later at 6.

Move to Step 3 when: You can explain how to find the maximum profit manually.


---

Step 3 — Derive the brute-force approach

Now ask yourself:

"How can I try every valid buying and selling combination?"

Think in terms of indices.

- i represents the buying day.
- j represents the selling day.
- Since you must buy first, j must be greater than i.

Calculate:

\[
profit=prices[j]-prices[i]
\]

Keep track of the maximum profit.

Write the algorithm in plain English

1. Choose a buying day i.
2. Choose every selling day j after i.
3. Calculate prices[j] - prices[i].
4. Update the maximum profit if this profit is greater.
5. Return the maximum profit.

Now translate that into loop structure

for (int i = 0; i < prices.length; i++) {
    for (int j = i + 1; j < prices.length; j++) {

        int profit = prices[j] - prices[i];

        // Update maximum profit
    }
}

Notice how the problem statement determines the code:

- We need to consider buying days, so we use the outer loop.
- For each buying day, we consider later selling days, so we use the inner loop.
- We start j at i + 1 because selling must happen after buying.

Why do this?

The brute-force approach gives you a correct baseline before optimization.

Move to Step 4 when: You understand what each loop represents.


---

Step 4 — Dry-run the brute force

Use this array:

prices = [7, 2, 5, 1, 6]

Initially:

maxProfit = 0

Let's choose i = 1, meaning the buying price is 2.

Now move j through the remaining days.

Buy index i	Sell index j	Buy price	Sell price	Profit	Updated maximum

1	2	2	5	3	3
1	3	2	1	-1	3
1	4	2	6	4	4


Then the outer loop continues with the next possible buying day.

When i = 3, we buy at 1 and can sell at 6, giving a profit of 5.

The final answer is 5.

Ask yourself

- Does j always come after i?
- Are we calculating selling price minus buying price?
- Do we update the maximum only when we find a larger profit?
- Does the algorithm work when prices keep decreasing?

For example:

prices = [7, 6, 4, 3, 1]

Every possible transaction loses money. Since we initialized maxProfit = 0 and only replace it with larger profits, the answer remains 0.

Move to Step 5 when: You can trace the loops and explain how the maximum is maintained.


---

Step 5 — Analyze complexity and find the bottleneck

Our brute-force approach works, but it checks every possible buying-and-selling combination.

For an array of length \(n\), there are approximately:

\[
\frac{n(n-1)}{2}
\]

possible pairs.

Therefore:

- Time complexity: \(O(n^2)\)
- Extra space: \(O(1)\)

Now ask the following questions.

Question 1: What work am I repeating?

For every selling day, the brute-force algorithm considers multiple earlier buying days.

Question 2: What information do I actually need to calculate the best profit on a particular selling day?

Suppose today's selling price is 6.

To maximize today's profit, would you rather have bought at 5, 2, or 1?

You would rather have bought at 1, because:

\[
6-1=5
\]

The important observation is that for a given selling day, you only need the lowest price encountered before that day.

You don't need to reconsider every earlier buying price individually.

Question 3: Can I remember that lowest price as I scan the array?

Yes!

Maintain a variable:

minPrice

It stores the minimum price seen so far.

Then calculate the profit available at each current price:

profit = prices[i] - minPrice;

This avoids the repeated search through previous prices.

Move to Step 6 when: You understand why remembering the minimum previous price is enough.


---

Step 6 — Derive the optimal approach

Let's process the prices from left to right.

prices = [7, 2, 5, 1, 6]

We need two variables:

- minPrice: the cheapest price encountered so far.
- maxProfit: the best valid profit found so far.

Think through the decisions

Day 1: Price = 7

We have encountered no earlier buying day.

Initialize:

minPrice = 7
maxProfit = 0

Day 2: Price = 2

The current price is lower than our previous minimum.

Update:

minPrice = 2

No positive profit has been found.

Day 3: Price = 5

The cheapest previous price is 2.

Today's possible profit is:

\[
5-2=3
\]

Update:

maxProfit = 3

Day 4: Price = 1

The current price is lower than the minimum we've seen.

Update:

minPrice = 1

The maximum profit remains 3.

Day 5: Price = 6

The cheapest previous price is 1.

Today's possible profit is:

\[
6-1=5
\]

Update:

maxProfit = 5

The answer is 5.

The optimized algorithm

1. Initialize minPrice to the first price.
2. Initialize maxProfit to 0.
3. Visit every remaining day.
4. Calculate the profit possible at the current price using the minimum earlier price.
5. Update maxProfit if this profit is better.
6. Update minPrice if the current price is lower.
7. Return maxProfit.

Why calculate profit before updating minPrice?

Because the current price represents today's selling price. The buying price must come from an earlier day.

If you update minPrice with today's price first, you could accidentally treat today's price as both the buying price and selling price.

Although this would produce zero profit for that particular comparison, the proper order reflects the problem's buy-before-sell requirement.

The current price is compared against the minimum from earlier days, then it becomes eligible as a buying price for future days.

Move to Step 7 when: You understand what both variables mean and why the calculation and updates occur in this order.


---

Step 7 — Dry-run the optimal approach

Let's create the final trace table.

Input:

prices = [7, 2, 5, 1, 6]

Initialize:

minPrice = 7
maxProfit = 0

Process the remaining prices.

Day index i	Current price	minPrice before update	Current profit	maxProfit after update	minPrice after update

1	2	7	-5	0	2
2	5	2	3	3	2
3	1	2	-1	3	1
4	6	1	5	5	1


Final answer:

5

Check the edge cases

Case 1: Prices always decrease

prices = [7, 6, 4, 3, 1]

No profitable transaction exists.

Answer: 0.

Case 2: Only one day

prices = [5]

There is no later day on which to sell.

Answer: 0.

Case 3: Prices increase

prices = [1, 2, 3, 4, 5]

Buy at 1, sell at 5.

Answer: 4.

Move to Step 8 when: Your trace produces the correct answer and you can explain why the minimum price always represents a valid earlier buying opportunity.


---

Step 8 — Write the Java code in the correct order

This is the most important part for developing your coding process.

Don't start by trying to remember the final code. Take your numbered algorithm and translate each step into Java.

First, decide what must happen and when

Order	What to write	Why it comes here

1	Method signature	Defines the input and output
2	Initialize minPrice	We need a starting minimum before processing later days
3	Initialize maxProfit	No profitable transaction has been found yet
4	Write the loop starting at index 1	The first price has already initialized our state
5	Calculate the current profit	Compare today's price against the previous minimum
6	Update maxProfit	Record today's profit if it is the best so far
7	Update minPrice	Make today's price available as a buying price for future days
8	Return maxProfit	The final answer is ready after processing all prices


Notice the dependency:

Calculate profit → update maximum profit → update minimum price.

The calculation uses the previous minimum; the update then prepares the state for the next day.

Complete Java solution

class Solution {
    public int maxProfit(int[] prices) {

        // 1. Initialize the minimum buying price
        int minPrice = prices[0];

        // 2. Initialize the maximum profit
        int maxProfit = 0;

        // 3. Process each remaining day
        for (int i = 1; i < prices.length; i++) {

            // 4. Calculate today's possible profit
            int profit = prices[i] - minPrice;

            // 5. Update the best profit
            maxProfit = Math.max(maxProfit, profit);

            // 6. Update the minimum price for future days
            minPrice = Math.min(minPrice, prices[i]);
        }

        // 7. Return the best profit
        return maxProfit;
    }
}

Why is the loop outside the initialization?

Because minPrice and maxProfit must have valid initial values before the first iteration.

Why is profit calculated inside the loop?

Because each day has a different selling price.

Why does maxProfit update after calculating profit?

Because we need the current day's profit before we can decide whether it improves our answer.

Why does minPrice update after calculating the profit?

Because today's price can be used for buying on a future day, but not for buying and selling on the same day.

Why does return come after the loop?

Because we need to consider every possible selling day before knowing the maximum profit.

Complexity

- Time: \(O(n)\), because we process each price once.
- Extra space: \(O(1)\), because we maintain only a constant number of variables.


---

Your revision card: Best Time to Buy and Sell Stock

Before moving to the next problem, close this explanation and answer these questions from memory.

Step	Question

1. Understand	What are the input, output, and transaction rules?
2. Example	Can I manually identify the best buying and selling days?
3. Brute force	How can I try every valid pair of days?
4. Dry-run	Do my indices ensure buying happens before selling?
5. Bottleneck	What repeated work can I eliminate?
6. Optimize	Can I remember the minimum price seen before today?
7. Validate	Do I calculate profit before updating the minimum price?
8. Code	What must be initialized, what belongs inside the loop, and when should I return?


One final thing to notice

Compare the two problems you've solved.

Problem	Brute-force idea	Key optimization

Two Sum	Try every pair	Store previous values in a HashMap
Best Time to Buy and Sell Stock	Try every buying/selling pair	Store the minimum earlier price


Both follow the same eight steps, but their optimization ideas differ.

Your task now: Solve Best Time to Buy and Sell Stock once without looking at this answer. Write the brute-force logic, explain why tracking the minimum price works, and then write the optimized Java code.

If you can derive the solution instead of merely recalling it, you're developing exactly the problem-solving skill you need for unfamiliar assessment questions.