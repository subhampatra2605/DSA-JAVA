
/*
 * Problem: 1. Two Sum
 * Platform: LeetCode
 * Difficulty: Easy
 * Pattern: Array + HashMap (Complement Lookup)
 *
 * PROBLEM:
 * Given an integer array nums and an integer target,
 * return the indices of two distinct elements whose
 * sum equals target.
 *
 * Exactly one solution exists, and the same element
 * cannot be used twice.
 *
 * Example:
 * Input:  nums = [2, 7, 11, 15], target = 9
 * Output: [0, 1]
 *
 * Explanation: nums[0] + nums[1] = 2 + 7 = 9.
 *
 * -------------------------------------------------------
 * APPROACH 1: BRUTE FORCE
 * -------------------------------------------------------
 * Use two nested loops to check every possible pair.
 *
 * For each i:
 *   For each j = i + 1:
 *     If nums[i] + nums[j] == target:
 *       Return [i, j]
 *
 * Time Complexity:  O(n^2)
 * Space Complexity: O(1) auxiliary space
 *
 * -------------------------------------------------------
 * APPROACH 2: OPTIMAL - HASHMAP
 * -------------------------------------------------------
 * Instead of searching the remaining array for a pair,
 * use a HashMap to remember numbers already encountered
 * and their indices.
 *
 * For each nums[i]:
 *
 * 1. Calculate the required complement:
 *      complement = target - nums[i]
 *
 * 2. Check whether complement already exists in the map.
 *
 * 3. If it exists, return:
 *      [map.get(complement), i]
 *
 * 4. Otherwise, store:
 *      map.put(nums[i], i)
 *
 * Time Complexity:  O(n) expected
 * Space Complexity: O(n)
 *
 * -------------------------------------------------------
 * KEY INSIGHT:
 * -------------------------------------------------------
 * Instead of asking:
 * "Which two numbers add up to target?"
 *
 * Ask:
 * "For the current number, what other number do I need?"
 *
 * Required number = target - current number
 *
 * Store VALUE -> INDEX in the HashMap.
 *
 * -------------------------------------------------------
 * WHY CHECK BEFORE INSERTING?
 * -------------------------------------------------------
 * Consider nums = [3, 3], target = 6.
 *
 * At index 0:
 *   Complement = 3
 *   Not found, so store 3 -> 0.
 *
 * At index 1:
 *   Complement = 3
 *   Found at index 0, so return [0, 1].
 *
 * Checking first guarantees that the matching element
 * comes from an earlier index, not the current element.
 *
 * -------------------------------------------------------
 * EDGE CASES:
 * -------------------------------------------------------
 * [2, 7, 11, 15], target = 9 -> [0, 1]
 * [3, 2, 4],      target = 6 -> [1, 2]
 * [3, 3],         target = 6 -> [0, 1]
 * [-1, -2, -3, -4], target = -6 -> [1, 3]
 *
 * -------------------------------------------------------
 * REVISION TAKEAWAY:
 * -------------------------------------------------------
 * Pair-sum problem + need original indices
 * -> Think HashMap: value -> index.
 *
 * Complement = target - current value.
 *
 * Check the complement BEFORE storing the current value.
 */

import java.util.HashMap;
import java.util.Map;

class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];

            if (map.containsKey(complement)) {
                return new int[] {map.get(complement), i};
            }

            map.put(nums[i], i);
        }

        return new int[0];
    }
}
