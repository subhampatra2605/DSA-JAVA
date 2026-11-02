# LeetCode 455 — Assign Cookies



Easy

Greedy + Sorting + Two Pointers



Pattern: Greedy Algorithm + Two Pointers

This problem introduces an important greedy concept: satisfy the least demanding requirement with the smallest resource that can satisfy it.

Let's follow your 8-step DSA framework.

## 1. Understand the Problem

You are given two integer arrays:

- `g[i]` = greed factor of the `i-th` child. This is the minimum cookie size that child needs.
- `s[j]` = size of the `j-th` cookie.

Each child can receive at most one cookie, and each cookie can be assigned to at most one child.

A child is satisfied if:

\\[ s[j] \geq g[i] \\]

Return the maximum number of children that can be satisfied.

### Example 1

```
Input:
g = [1,2,3]
s = [1,1]

Output: 1
```

We have:

- Child 1 needs at least 1.
- Child 2 needs at least 2.
- Child 3 needs at least 3.

Both cookies have size 1.

We can satisfy only one child.

### Example 2

```
Input:
g = [1,2]
s = [1,2,3]

Output: 2
```

We can distribute:

```
Child 1 (greed 1) ← Cookie 1
Child 2 (greed 2) ← Cookie 2
```

Both children are satisfied.

## 2. Brute-Force Approach

### Idea

For every child, search for an unused cookie that is large enough.

We maintain a boolean array to track which cookies have already been assigned.

For each child:

1. Check all available cookies.
2. Find a cookie with size greater than or equal to the child's greed factor.
3. Assign it and mark it as used.
4. Increment the satisfied count.

### Java Code — Brute Force

```
class Solution {
    public int findContentChildren(int[] g, int[] s) {

        boolean[] used = new boolean[s.length];
        int satisfied = 0;

        // Consider every child.
        for (int i = 0; i < g.length; i++) {

            // Find an unused cookie that satisfies this child.
            for (int j = 0; j < s.length; j++) {

                if (!used[j] && s[j] >= g[i]) {

                    // Assign the cookie.
                    used[j] = true;
                    satisfied++;

                    // Move to the next child.
                    break;
                }
            }
        }

        return satisfied;
    }
}
```

### Complexity

- Time: `O(n × m)`, where `n` is the number of children and `m` is the number of cookies.
- Space: `O(m)` for the used array.

This approach works, but the assignment order can affect how many children we satisfy.

We need a better strategy.

## 3. Identify the Bottleneck

Consider:

```
g = [1,2,3]
s = [3,1,2]
```

Suppose we give the cookie of size `3` to the first child.

```
Child 1 (greed 1) ← Cookie 3
```

Now the remaining cookies are:

```
[1,2]
```

We can satisfy the second child with cookie `2`, but the third child cannot be satisfied.

Total satisfied = 2.

But we wasted a large cookie on a child who could have been satisfied with a smaller one.

The bottleneck is inefficient cookie assignment.

We need to distribute cookies intelligently.

## 4. Recognize the Pattern — Greedy

The key observation:

Give the smallest available cookie that can satisfy the least demanding unsatisfied child.

Why?

- A child with lower greed has fewer requirements.
- A larger cookie can potentially satisfy a child with higher greed.
- We should preserve larger cookies for children who need them.

This is a Greedy Algorithm.

To make this easier, sort both arrays.

```
g = [1,2,3]
s = [1,2,3]
```

Now we can use two pointers.

- `i` tracks the current child.
- `j` tracks the current cookie.

[Assign Cookies - Tutorial](https://images.openai.com/static-rsc-4/i4ypCoHNAtIlPfHXPWaZsAsiKt5sT9f0OO8NoGSsMicKeIHCYsjmpjZ0nGv3rCTaQUeGwLYJlVI3RIZE76E4nX1t871LrAs3-tv_oH2qQImwok4SrjxjY7Yy7NhtBh0FPIMwXMdKxxyMVKY6rFoHBNTqU7LZtXz4W4luXJ8IEdc?purpose=inline)

[takeuforward.org](https://takeuforward.org/Greedy/assign-cookies?utm_source=chatgpt.com)

## 5. Derive the Optimal Approach

First, sort both arrays:

```
Arrays.sort(g);
Arrays.sort(s);
```

Initialize:

```
int i = 0;
int j = 0;
int satisfied = 0;
```

### Case 1: Cookie can satisfy the current child

Condition:

```
if (s[j] >= g[i])
```

Assign the cookie.

```
satisfied++;
i++;
j++;
```

Why move both pointers?

- The current child has been satisfied.
- The current cookie has been used.

Move to the next child and cookie.

### Case 2: Cookie cannot satisfy the current child

Condition:

```
else
```

The current cookie is too small.

```
j++;
```

Why don't we move the child pointer?

Because this cookie cannot satisfy the current child, but a larger cookie might.

We therefore try the next cookie.

### Important example

```
g = [1,2,3]
s = [1,1,2]
```

Sorted arrays:

```
g = [1,2,3]
s = [1,1,2]
```

| Step | Child greed | Cookie size     | Decision               |
| ---- | ----------- | --------------- | ---------------------- |
| 1    | 1           | 1               | Satisfied              |
| 2    | 2           | 1               | Too small, skip cookie |
| 3    | 2           | 2               | Satisfied              |
| 4    | 3           | No cookies left | Cannot satisfy         |

Answer = `2`.

## 6. Optimal Java Solution

Optimal Greedy Solution

```
import java.util.Arrays;

class Solution {
    public int findContentChildren(int[] g, int[] s) {

        // Sort greed factors and cookie sizes.
        Arrays.sort(g);
        Arrays.sort(s);

        int i = 0; // Child pointer
        int j = 0; // Cookie pointer

        int satisfied = 0;

        // Continue while children and cookies are available.
        while (i < g.length && j < s.length) {

            // If the cookie can satisfy the current child.
            if (s[j] >= g[i]) {

                // Child is satisfied.
                satisfied++;

                // Move to the next child.
                i++;

                // Cookie has been used.
                j++;

            } else {

                // Cookie is too small.
                // Try the next larger cookie.
                j++;
            }
        }

        return satisfied;
    }
}
```

### Complexity

Let:

- `n` = number of children.
- `m` = number of cookies.

| Operation             | Complexity   |
| --------------------- | ------------ |
| Sort children         | `O(n log n)` |
| Sort cookies          | `O(m log m)` |
| Two-pointer traversal | `O(n + m)`   |

Overall Time: `O(n log n + m log m)`

Auxiliary Space: Depends on the sorting implementation; the two-pointer traversal itself uses `O(1)` extra space.

## 7. Detailed Dry Run

Let's trace:

```
g = [1,2,3]
s = [1,1,2]
```

After sorting:

```
g = [1,2,3]
s = [1,1,2]
```

Initially:

```
i = 0
j = 0
satisfied = 0
```

### Iteration 1

```
Child: 1
Cookie: 1
```

Check:

```
s[j] >= g[i]
1 >= 1 → true
```

Child is satisfied.

```
satisfied++;
i++;
j++;
```

New state:

```
i = 1
j = 1
satisfied = 1
```

### Iteration 2

```
Child: 2
Cookie: 1
```

Check:

```
1 >= 2 → false
```

Cookie is too small.

Move only the cookie pointer:

```
j++;
```

New state:

```
i = 1
j = 2
satisfied = 1
```

Notice: We did not move `i`, because child 2 is still waiting for a suitable cookie.

### Iteration 3

```
Child: 2
Cookie: 2
```

Check:

```
2 >= 2 → true
```

Child is satisfied.

Move both pointers:

```
satisfied++;
i++;
j++;
```

New state:

```
i = 2
j = 3
satisfied = 2
```

### Loop terminates

Because:

```
j == s.length
```

No cookies remain.

The third child cannot be satisfied.

Final Answer: 2

### Complete Dry Run Table

| Iteration | `i` | `j` | `g[i]` | `s[j]` | Action                     |
| --------- | --- | --- | ------ | ------ | -------------------------- |
| 1         | 0   | 0   | 1      | 1      | Assign, move both          |
| 2         | 1   | 1   | 2      | 1      | Cookie too small, move `j` |
| 3         | 1   | 2   | 2      | 2      | Assign, move both          |
| End       | 2   | 3   | —      | —      | No cookies left            |

## 8. Tricky Parts You Must Understand

### A. Why sort both arrays?

Sorting allows us to process the least demanding child and smallest available cookie first.

Without sorting, we might waste a large cookie on a child who could have been satisfied with a smaller one.

### B. Why move only `j` when the cookie is too small?

Suppose:

```
g = [2,3,4]
s = [1,2,3]
```

Current comparison:

```
Child greed = 2
Cookie size = 1
```

The cookie cannot satisfy the child.

But perhaps the next cookie can.

So we move:

```
j++;
```

We keep `i` unchanged.

### C. Why move both pointers when a child is satisfied?

Because:

- One child has received a cookie.
- One cookie has been consumed.

Neither can be used again.

### D. Why is this greedy strategy optimal?

Suppose the smallest remaining cookie can satisfy the least demanding remaining child.

Giving that cookie to that child is safe:

- The child could not benefit from a larger cookie in terms of satisfaction count.
- Saving the smallest sufficient cookie for a more demanding child cannot help, because that child needs at least as much as the current child.
- The larger cookies remain available for children with higher greed factors.

If the smallest cookie cannot satisfy the least demanding child, it cannot satisfy any remaining child either, so skipping it is also safe.

This is why the greedy strategy gives the maximum number of satisfied children.

## GitHub-Ready Notes

```
Problem: LeetCode 455 — Assign Cookies

Pattern:
Greedy + Sorting + Two Pointers

Approach:
1. Sort greed factors and cookie sizes.
2. Initialize i = 0 (child pointer).
3. Initialize j = 0 (cookie pointer).
4. If s[j] >= g[i]:
   - Satisfy the child.
   - Increment both pointers.
5. Otherwise:
   - Cookie is too small.
   - Increment only j.
6. Stop when children or cookies are exhausted.
7. Return satisfied count.

Time Complexity:
O(n log n + m log m)

Auxiliary Space:
O(1) for the two-pointer traversal,
excluding sorting implementation space.

Key Insight:
Satisfy the least demanding child using the smallest
available cookie that can satisfy them.

Important:
If cookie is too small, move only j.
If child is satisfied, move both i and j.
```

## Quick Revision: Greedy vs Two Pointers

| Problem                           | Main pattern           | Greedy decision                                   |
| --------------------------------- | ---------------------- | ------------------------------------------------- |
| LC 11 — Container With Most Water | Two Pointers           | Move the shorter boundary                         |
| LC 15 — 3Sum                      | Sorting + Two Pointers | Move pointers based on sum                        |
| LC 455 — Assign Cookies           | Greedy + Two Pointers  | Smallest sufficient cookie for least greedy child |

## Test Your Understanding

1\. g = [1,2,3], s = [1,2]. What is the maximum number of satisfied children?

1

2

3

2\. If s[j] < g[i], which pointer should move?

Only i

Only j

Both i and j

3\. If s[j] >= g[i], which pointers should move?

Only i

Only j

Both i and j

4\. Why do we sort both arrays?

To use binary search

To match small cookies with less demanding children first

To remove duplicate values

Check answers

Next recommended problem: LC 881 — Boats to Save People. It builds on sorting, greedy decisions, and two pointers, and is a useful follow-up for your Infosys SP preparation.
