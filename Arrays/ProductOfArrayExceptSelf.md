LeetCode 238 — Product of Array Except Self

Let's solve this using your 8-step problem-solving framework, focusing especially on how to identify the bottleneck, discover the optimal approach, and determine the correct order of the final Java code.

The goal is not just to memorize the solution. It's to understand how to derive it when you encounter a similar problem in the Infosys SP/DSE assessment.


---

Step 1: Understand the Problem

Problem statement

Given an integer array nums, return an array answer such that:

answer[i] equals the product of every element in nums except nums[i].

Rules:

- You cannot use the element at index i in its own product.
- You must not use division.
- The target solution should run in O(n) time.
- The answer array doesn't count as extra space for the usual space-complexity analysis.

Example

Input:  nums = [1, 2, 3, 4]
Output:       [24, 12, 8, 6]

Let's understand each output:

Index	Exclude	Calculation	Answer

0	1	2 × 3 × 4	24
1	2	1 × 3 × 4	12
2	3	1 × 2 × 4	8
3	4	1 × 2 × 3	6


First checkpoint: For any index i, we need the product of all elements to its left multiplied by the product of all elements to its right.

That observation will lead us to the optimal solution.


---

Step 2: Solve a Small Example Manually

Take:

nums = [2, 3, 4, 5]

Calculate each answer without writing code.

- Index 0: exclude 2 → 3 × 4 × 5 = 60
- Index 1: exclude 3 → 2 × 4 × 5 = 40
- Index 2: exclude 4 → 2 × 3 × 5 = 30
- Index 3: exclude 5 → 2 × 3 × 4 = 24

Therefore:

answer = [60, 40, 30, 24]

Now ask yourself:

What information would let me calculate each answer without repeatedly multiplying all the other elements?

Consider the products to the left and right of each index.

For [2, 3, 4, 5]:

Index	Left-side product	Right-side product	Final product

0	1	3 × 4 × 5 = 60	60
1	2	4 × 5 = 20	40
2	2 × 3 = 6	5	30
3	2 × 3 × 4 = 24	1	24


Why use 1 when there are no elements on one side?

Because 1 is the multiplicative identity: multiplying by 1 doesn't change the result.

Important insight: answer[i] = leftProduct[i] × rightProduct[i].

We now have a possible direction, but we still need to derive it efficiently.


---

Step 3: Derive the Brute-Force Approach

When you're stuck, don't immediately search for a clever algorithm. First, describe the most direct solution in plain English.

Brute-force logic

For every index i:

1. Set a product variable to 1.
2. Visit every index j.
3. If i != j, multiply the product by nums[j].
4. Store the product in answer[i].
5. Repeat for every index.

Translate the logic into Java

class Solution {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] answer = new int[n];

        for (int i = 0; i < n; i++) {
            int product = 1;

            for (int j = 0; j < n; j++) {
                if (i != j) {
                    product *= nums[j];
                }
            }

            answer[i] = product;
        }

        return answer;
    }
}

This is a useful first solution because it establishes the logic clearly.

However, it takes O(n²) time, so it doesn't meet the intended O(n) time requirement.

Let's understand exactly why.


---

Step 4: Dry-Run the Brute Force

Input:

nums = [2, 3, 4, 5]

For i = 0, we calculate:

product = 1

j = 0 → skip because i == j
j = 1 → product = 1 × 3 = 3
j = 2 → product = 3 × 4 = 12
j = 3 → product = 12 × 5 = 60

answer[0] = 60

For i = 1:

product = 1

j = 0 → product = 1 × 2 = 2
j = 1 → skip
j = 2 → product = 2 × 4 = 8
j = 3 → product = 8 × 5 = 40

answer[1] = 40

We repeat this process for every index.

The issue is that we keep recalculating products.

For example, the multiplication 4 × 5 is performed when calculating the answer at index 0, but we also calculate those elements again when processing other indices.

This repeated work is what we want to eliminate.


---

Step 5: Identify the Bottleneck

This is the most important thinking step.

You already know that the brute-force solution works. Now ask:

1. What work am I repeating?
2. What information have I already calculated?
3. Can I save that information and reuse it?

Let's apply these questions.

Question 1: What work am I repeating?

For every index, we're multiplying nearly all the elements again.

For n elements, we perform approximately:

n × (n - 1) multiplications

Therefore, the time complexity is O(n²).

Question 2: What information can I reuse?

Remember our observation from Step 2:

answer[i] = product of elements on the left
          × product of elements on the right

For this array:

nums = [2, 3, 4, 5]

The products of elements to the left of each index are:

leftProduct = [1, 2, 6, 24]

Why?

Index 0: no elements on the left → 1
Index 1: 2                   → 2
Index 2: 2 × 3               → 6
Index 3: 2 × 3 × 4           → 24

Notice something important:

leftProduct[1] = leftProduct[0] × nums[0]
leftProduct[2] = leftProduct[1] × nums[1]
leftProduct[3] = leftProduct[2] × nums[2]

We don't need to recalculate each left-side product from scratch.

We can build it using the previous result.

Similarly, the right-side products can be built from right to left:

rightProduct = [60, 20, 5, 1]

Then multiply the corresponding left and right products.

Question 3: Can we do this without storing two extra arrays?

Yes.

We can store the left-side product directly in answer, then traverse the input from right to left while maintaining a running right-side product.

This gives us an O(n) solution with O(1) auxiliary space, excluding the required output array.

The general lesson: When a problem asks you to calculate something for every index, check whether the information for one index can be built from information already calculated for a neighboring index.

This often leads to prefix/suffix techniques.


---

Step 6: Derive the Optimized Algorithm

We'll use two passes.

- Pass 1: Store the product of all elements to the left of each index.
- Pass 2: Multiply each stored result by the product of all elements to the right.

Pass 1: Build left products

Input:

nums = [2, 3, 4, 5]

Start with:

int prefix = 1;

Why 1?

Because before the first element, there are no elements on its left. We need an identity value that doesn't change the product.

Now, at each index:

1. Store prefix in answer[i].
2. Multiply prefix by nums[i] so it's ready for the next index.

Why store before multiplying?

Because answer[i] must exclude nums[i]. If we multiply first, the prefix would incorrectly include the current element.

Let's trace it:

Index i	prefix before storing	answer[i]	prefix after multiplying by nums[i]

0	1	1	2
1	2	2	6
2	6	6	24
3	24	24	120


After Pass 1:

answer = [1, 2, 6, 24]

These are the left-side products, not the final answers yet.

Pass 2: Multiply by right products

Now traverse from right to left.

Initialize:

int suffix = 1;

Again, 1 represents the absence of elements to the right of the last index.

At every index:

1. Multiply answer[i] by suffix.
2. Update suffix by multiplying it by nums[i].

Why multiply before updating suffix?

Because the current answer must exclude nums[i]. At the moment we calculate the answer, suffix must contain only elements strictly to the right.

Let's trace it:

Index i	answer[i] before	suffix before	Final answer[i]	suffix after multiplying by nums[i]

3	24	1	24	5
2	6	5	30	20
1	2	20	40	60
0	1	60	60	120


Final result:

answer = [60, 40, 30, 24]

The two rules to remember

Left-to-right pass:

Store current prefix → update prefix

Right-to-left pass:

Apply current suffix → update suffix

We store or apply the running product before incorporating the current element because the current element must be excluded from its own answer.


---

Step 7: Dry-Run, Validate, and Check Edge Cases

Before writing the final code, make sure you can explain what each variable means.

Variable	Meaning

nums[i]	Current input element
answer[i] during Pass 1	Product of elements strictly to the left
prefix	Product of all elements processed so far in the left-to-right pass
suffix	Product of elements strictly to the right during the second pass
answer[i] after Pass 2	Product of every element except nums[i]


Edge case 1: Two elements

Input:  [3, 7]
Output: [7, 3]

The method works because each element's product is simply the other element.

Edge case 2: Contains zero

Input:  [1, 2, 0, 4]
Output: [0, 0, 8, 0]

Why?

- Excluding 1: the remaining product contains zero.
- Excluding 2: the remaining product contains zero.
- Excluding 0: the product is 1 × 2 × 4 = 8.
- Excluding 4: the remaining product contains zero.

No special zero-handling logic is needed. The two-pass approach handles it naturally.

Edge case 3: Negative numbers

Input:  [-1, 2, -3, 4]
Output: [-24, 12, -8, 6]

Negative values work naturally because ordinary multiplication handles their signs.

Edge case 4: Single element

Input: [5]
Output: [1]

The product of all elements except the only element is the empty product, conventionally 1. The algorithm returns [1].

Complexity check

- Time: O(n), because we traverse the array twice.
- Auxiliary space: O(1), because we use only prefix and suffix besides the output array.
- Output space: O(n), for the required answer array.


---

Step 8: Write the Final Java Code in the Correct Order

Now we translate the algorithm into code.

The order follows the dependencies of the algorithm, not guesswork.

1. Method signature and output array

class Solution {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] answer = new int[n];

Why first?

We need a method that receives the input array and returns an array. We create the output array before storing any answers.

2. Initialize the prefix product

int prefix = 1;

Why here?

The first pass requires a running product. Before processing index 0, there are no elements on the left, so the initial product is 1.

3. Build the left-side products

for (int i = 0; i < n; i++) {
            answer[i] = prefix;
            prefix *= nums[i];
        }

Why in this order?

- answer[i] = prefix; stores the product excluding the current element.
- prefix *= nums[i]; incorporates the current element for the next iteration.

If you reverse these statements, the stored product would include the current element, violating the problem's rule.

4. Initialize the suffix product

int suffix = 1;

Why after the first pass?

The left products are already stored. Now we need a new running product for the right-to-left pass.

5. Apply the right-side products

for (int i = n - 1; i >= 0; i--) {
            answer[i] *= suffix;
            suffix *= nums[i];
        }

Why iterate backwards?

We need to accumulate the products of elements to the right. Traversing from right to left makes that possible.

Why multiply before updating?

The current suffix contains only the elements strictly to the right. After applying it, we can incorporate the current element for the next iteration.

6. Return the completed array

return answer;
    }
}

Why last?

Only after both passes does answer contain the final product for every index.

Complete solution

class Solution {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] answer = new int[n];

        // Pass 1: Store products of elements to the left.
        int prefix = 1;

        for (int i = 0; i < n; i++) {
            answer[i] = prefix;
            prefix *= nums[i];
        }

        // Pass 2: Multiply by products of elements to the right.
        int suffix = 1;

        for (int i = n - 1; i >= 0; i--) {
            answer[i] *= suffix;
            suffix *= nums[i];
        }

        return answer;
    }
}


---

How to Recognize This Pattern in an Assessment

When you see a problem like Product of Array Except Self, ask these questions in order:

1. What does each answer depend on?
   Here, each answer depends on all elements except the current one.

2. Can I divide the work into independent parts?
   Yes. Elements to the left and elements to the right.

3. Am I recalculating products repeatedly?
   Yes, in brute force.

4. Can I reuse a running result?
   Yes. A prefix product and a suffix product.

5. Can I store one part in the output array?
   Yes. Store prefix products in answer.

6. What order must I process the array in?
   Left to right for prefixes, then right to left for suffixes.

7. When must I update the running product?
   After storing or applying the product for the current index, because that index must be excluded.

Compare the optimization insight in your last three problems

Problem	Bottleneck in brute force	Optimized idea

Two Sum	Repeatedly searching pairs	HashMap remembers previously seen values
Best Time to Buy and Sell Stock	Repeatedly checking earlier buying prices	Track the minimum price seen so far
Product of Array Except Self	Repeatedly multiplying all other elements	Reuse prefix and suffix products


The common skill is finding what information from earlier work can be reused. The data structure or technique changes, but the reasoning process remains the same.

Your revision task

Without looking at the complete solution, write down these four things:

1. The brute-force algorithm and its time complexity.
2. The repeated work that makes brute force inefficient.
3. What prefix and suffix mean at the moment each answer is calculated.
4. Why each pass must store or apply the current product before updating the running product.

If you can explain those four points and reconstruct the code, you've understood the problem rather than merely memorized it.