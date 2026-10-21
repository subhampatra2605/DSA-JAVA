
/*
 * Problem: 121. Best Time to Buy and Sell Stock
 * Platform: LeetCode
 * Difficulty: Easy
 * Pattern: Arrays + Greedy (Running Minimum)
 *
 * PROBLEM:
 * Given an array prices[], where prices[i] is the stock price
 * on day i, find the maximum profit possible by buying once
 * and selling once on a later day.
 *
 * If no profit is possible, return 0.
 *
 * Example:
 * Input:  [7, 1, 5, 3, 6, 4]
 * Output: 5
 * Explanation: Buy at 1 and sell at 6.
 *
 * -------------------------------------------------------
 * APPROACH 1: BRUTE FORCE
 * -------------------------------------------------------
 * Try every possible buying day i and selling day j > i.
 * Calculate prices[j] - prices[i] and track maximum profit.
 *
 * Time Complexity:  O(n^2)
 * Space Complexity: O(1)
 *
 * -------------------------------------------------------
 * APPROACH 2: OPTIMAL - RUNNING MINIMUM
 * -------------------------------------------------------
 * Instead of checking every buying day, maintain the
 * lowest price encountered so far.
 *
 * For each day:
 * 1. Update the minimum buying price if today's price
 *    is lower.
 * 2. Calculate the profit from selling today.
 * 3. Update the maximum profit.
 *
 * Time Complexity:  O(n)
 * Space Complexity: O(1)
 *
 * -------------------------------------------------------
 * KEY INSIGHT:
 * -------------------------------------------------------
 * To maximize profit when selling on a particular day,
 * we should buy at the minimum price seen so far.
 *
 * My implementation updates buyPrice before calculating
 * profit. This is valid because:
 *
 * - If today's price is a new minimum, currProfit = 0.
 * - Otherwise, currProfit = today's price - earlier minimum.
 * - profit starts at 0, so zero-profit transactions
 *   never reduce the answer.
 *
 * Alternative standard ordering:
 *
 *   currProfit = prices[i] - buyPrice;
 *   profit = Math.max(profit, currProfit);
 *   buyPrice = Math.min(buyPrice, prices[i]);
 *
 * Both orderings produce the same final answer.
 *
 * -------------------------------------------------------
 * EDGE CASES:
 * -------------------------------------------------------
 * [7, 6, 4, 3, 1] -> 0  (prices always decrease)
 * [2, 4, 1]       -> 2
 * [1, 2, 3, 4]    -> 3
 * [5, 5, 5]       -> 0
 *
 * -------------------------------------------------------
 * REVISION TAKEAWAY:
 * -------------------------------------------------------
 * When asked to maximize the difference between a
 * future value and an earlier value, consider maintaining
 * a running minimum while scanning from left to right.
 */

class Solution {
    public int maxProfit(int[] prices) {
        int buyPrice = prices[0];
        int profit = 0;

        for (int i = 1; i < prices.length; i++) {

            if (prices[i] < buyPrice) {
                buyPrice = prices[i];
            }

            int currProfit = prices[i] - buyPrice;
            profit = Math.max(profit, currProfit);
        }

        return profit;
    }
}
