/*
 * Problem: Product of Array Except Self
 * Platform: LeetCode
 * Problem Number: 238
 * Difficulty: Medium
 * Topic: Arrays, Prefix Product, Suffix Product
 * Pattern: Prefix-Suffix Technique
 *
 * 1. PROBLEM STATEMENT
 * Given an integer array nums, return an array answer such that:
 * answer[i] = product of all elements of nums except nums[i].
 *
 * Requirements:
 * - Do not use division.
 * - Solve in O(n) time.
 * - Extra space complexity should be O(1), excluding the output array.
 *
 * Example:
 * Input:  nums = [1, 2, 3, 4]
 * Output: [24, 12, 8, 6]
 *
 * Explanation:
 * answer[0] = 2 * 3 * 4 = 24
 * answer[1] = 1 * 3 * 4 = 12
 * answer[2] = 1 * 2 * 4 = 8
 * answer[3] = 1 * 2 * 3 = 6
 *
 * 2. BRUTE-FORCE APPROACH
 * For each index i:
 * - Multiply every element except nums[i].
 * - Store the product in answer[i].
 *
 * Time Complexity: O(n^2)
 * Extra Space: O(1), excluding output.
 *
 * Limitation:
 * Repeatedly calculates products, resulting in unnecessary work.
 *
 * 3. OPTIMAL APPROACH — PREFIX AND SUFFIX PRODUCTS
 *
 * Key observation:
 * Product except self =
 * product of elements to the LEFT * product of elements to the RIGHT.
 *
 * answer[i] = prefixProduct[i] * suffixProduct[i]
 *
 * Instead of creating two extra arrays, use the output array and
 * two running variables: prefix and suffix.
 *
 * PASS 1: PREFIX PRODUCT
 * - prefix represents the product of elements strictly to the left
 *   of the current index.
 * - Store prefix in answer[i].
 * - Then multiply prefix by nums[i] for the next iteration.
 *
 * PASS 2: SUFFIX PRODUCT
 * - Traverse from right to left.
 * - suffix represents the product of elements strictly to the right
 *   of the current index.
 * - Multiply answer[i] by suffix.
 * - Then multiply suffix by nums[i] for the next iteration.
 *
 * 4. MOST IMPORTANT INSIGHT — ORDER OF OPERATIONS
 *
 * Prefix loop:
 *     answer[i] = prefix;
 *     prefix *= nums[i];
 *
 * Suffix loop:
 *     answer[i] *= suffix;
 *     suffix *= nums[i];
 *
 * WHY ANSWER FIRST, UPDATE SECOND?
 *
 * At the beginning of each iteration:
 * - prefix contains only elements before i.
 * - suffix contains only elements after i.
 *
 * The current element must NOT be included in its own prefix or
 * suffix product.
 *
 * Therefore:
 * 1. Use the accumulated product to calculate the current answer.
 * 2. Update the running product by including nums[i].
 * 3. The updated product will be used in the next iteration.
 *
 * General reasoning question:
 * "What should my running variable represent at the beginning of
 * this iteration, and should the current element be included in
 * the current answer or only in the next iteration?"
 *
 * 5. DRY RUN
 *
 * Input: nums = [1, 2, 3, 4]
 *
 * After the prefix pass:
 * answer = [1, 1, 2, 6]
 *
 * Suffix pass:
 * i = 3: answer[3] = 6 * 1  = 6;  suffix = 4
 * i = 2: answer[2] = 2 * 4  = 8;  suffix = 12
 * i = 1: answer[1] = 1 * 12 = 12; suffix = 24
 * i = 0: answer[0] = 1 * 24 = 24; suffix = 24
 *
 * Final output: [24, 12, 8, 6]
 *
 * 6. EDGE CASES
 * - Single element: [5] -> [1]
 * - Contains zero: [1, 2, 0, 4] -> [0, 0, 8, 0]
 * - Multiple zeros: [0, 2, 0] -> [0, 0, 0]
 * - Negative values: [-1, 2, 3] -> [6, -3, -2]
 *
 * 7. COMPLEXITY
 * Time Complexity: O(n)
 * Auxiliary Space: O(1), excluding the output array.
 *
 * 8. RECOGNITION CUES
 * Consider prefix-suffix when:
 * - Each answer depends on elements before and after an index.
 * - You need to exclude the current element.
 * - Division is prohibited.
 * - Repeated calculations can be avoided using running products.
 *
 * 9. KEY TAKEAWAY
 * Define what prefix and suffix mean before coding.
 * Use the accumulated value first, then update it for the next
 * iteration when the current element must be excluded.
 */

class Solution {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] answer = new int[n];

        // Pass 1: Product of elements strictly to the left.
        int prefix = 1;

        for (int i = 0; i < n; i++) {
            answer[i] = prefix;
            prefix *= nums[i];
        }

        // Pass 2: Product of elements strictly to the right.
        int suffix = 1;

        for (int i = n - 1; i >= 0; i--) {
            answer[i] *= suffix;
            suffix *= nums[i];
        }

        return answer;
    }
}
