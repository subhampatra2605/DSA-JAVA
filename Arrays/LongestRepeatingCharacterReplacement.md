# LeetCode 424 — Longest Repeating Character Replacement

**Pattern:** Variable-Size Sliding Window + Frequency Array  
**Difficulty:** Medium  
**Goal:** Understand why the sliding window works, especially why we use `maxFreq` and why the condition is `windowLength - maxFreq > k`.

We'll follow your 8-step DSA framework.

---

## 1. Understand the Problem

**Problem:** You are given a string `s` containing uppercase English letters and an integer `k`.

You can replace at most `k` characters in the string with any uppercase English letter. Return the length of the longest substring that can contain the same character after these replacements.

### Example 1

```text
Input: s = "ABAB", k = 2
Output: 4
```

We can replace both `A`s with `B`s:

```text
Original:   A B A B
Replaced:   B B B B
```

We replaced two characters, so the entire string becomes identical.

Answer: `4`

### Example 2

```text
Input: s = "AABABBA", k = 1
Output: 4
```

One valid substring is:

```text
A A B A
```

Replace `B` with `A`:

```text
A A A A
```

Only one replacement is needed, so the length is `4`.

**Important:** The substring must be contiguous. We cannot select characters from different parts of the string.

---

## 2. Brute-Force Approach

### Idea

Consider every possible substring.

For each substring:
1. Count the frequency of every character.
2. Find the character that appears most frequently.
3. Calculate how many replacements are needed.
4. If replacements needed `<= k`, update the maximum length.

### How do we calculate replacements?

Suppose the current substring is:

```text
A A B A B
```

Its length is `5`.

Frequencies:

```text
A → 3
B → 2
```

To make every character identical, we should keep the most frequent character, `A`, and replace the other two characters.

Therefore:

```text
Replacements needed = window length - maximum frequency
                    = 5 - 3
                    = 2
```

If `k = 2`, this substring is valid.

### Java Code — Brute Force

```java
class Solution {
    public int characterReplacement(String s, int k) {
        int n = s.length();
        int maxLength = 0;

        // Try every possible starting position.
        for (int left = 0; left < n; left++) {

            // Frequency of uppercase English letters.
            int[] frequency = new int[26];
            int maxFreq = 0;

            // Extend the substring to every possible right position.
            for (int right = left; right < n; right++) {

                char current = s.charAt(right);

                // Update the frequency of the current character.
                frequency[current - 'A']++;

                // Track the most frequent character in this substring.
                maxFreq = Math.max(
                    maxFreq,
                    frequency[current - 'A']
                );

                // Minimum replacements needed to make this
                // substring contain only one distinct character.
                int windowLength = right - left + 1;
                int replacements = windowLength - maxFreq;

                // The substring is valid if we can replace
                // at most k characters.
                if (replacements <= k) {
                    maxLength = Math.max(maxLength, windowLength);
                }
            }
        }

        return maxLength;
    }
}
```

### Complexity

- **Time:** `O(n²)` — there are `O(n²)` substrings, with constant-time frequency updates.
- **Space:** `O(1)` — the frequency array always has 26 elements.

---

## 3. Identify the Bottleneck

The brute-force solution considers every possible substring.

For a string of length `n`, there are approximately:

\[
\frac{n(n+1)}{2}
\]

possible substrings.

For example, with `n = 10,000`, examining every substring is too expensive.

**Can we avoid restarting from every possible left position?**

Yes! We can use a **variable-size sliding window**.

Instead of repeatedly examining substrings from scratch, we expand the window and shrink it whenever it becomes invalid.

---

## 4. Recognize the Pattern

The key observations are:

- We need a **contiguous substring**.
- We need the **longest** valid substring.
- The window can expand when we add a character.
- If the window requires too many replacements, we shrink it.

These characteristics suggest:

**Variable-Size Sliding Window + Character Frequency Tracking**

The central challenge is deciding whether the current window is valid.

### The key formula

Let:

- `windowLength = right - left + 1`
- `maxFreq` = frequency of the most common character in the window
- `k` = maximum allowed replacements

Then:

\[
\boxed{\text{replacementsNeeded}=\text{windowLength}-\text{maxFreq}}
\]

Why?

We keep every occurrence of the most frequent character and replace all other characters.

For example:

```text
Window: A A B A C
Length: 5

A → 3
B → 1
C → 1
```

Keep the three `A`s and replace `B` and `C`.

```text
Replacements needed = 5 - 3 = 2
```

If `k = 2`, the window is valid.

If `k = 1`, the window is invalid.

Therefore, the validity condition is:

\[
\boxed{\text{windowLength}-\text{maxFreq}\leq k}
\]

And the invalidity condition is:

\[
\boxed{\text{windowLength}-\text{maxFreq}>k}
\]

---

## 5. Derive the Optimal Approach

We maintain four things:

```java
int left = 0;
int maxLength = 0;
int maxFreq = 0;
int[] frequency = new int[26];
```

### Step 1: Expand the window

Move `right` from the beginning to the end of the string.

For each character:

```java
char current = s.charAt(right);
frequency[current - 'A']++;
```

Why `current - 'A'`?

Because uppercase English letters can be mapped to array indices:

```text
A → 0
B → 1
C → 2
...
Z → 25
```

This lets us count character frequencies using a fixed-size array.

### Step 2: Update `maxFreq`

```java
maxFreq = Math.max(
    maxFreq,
    frequency[current - 'A']
);
```

This tracks the highest frequency encountered as the window expands.

For example:

```text
Window: A A B A

A frequency = 3
B frequency = 1

maxFreq = 3
```

### Step 3: Shrink the window if necessary

```java
while ((right - left + 1) - maxFreq > k) {
    frequency[s.charAt(left) - 'A']--;
    left++;
}
```

If the window needs more than `k` replacements, remove characters from the left until the condition becomes valid.

Notice that we use `while`, not `if`, because the window might need to shrink more than once.

### Step 4: Update the answer

```java
maxLength = Math.max(maxLength, right - left + 1);
```

After shrinking, the window satisfies the validity condition used by the algorithm, so we consider its length.

---

## 6. Optimal Java Solution

Here is the implementation to use for LeetCode 424.

```java
class Solution {
    public int characterReplacement(String s, int k) {
        int left = 0;
        int maxLength = 0;
        int maxFreq = 0;

        // Frequency of each uppercase English letter.
        int[] frequency = new int[26];

        // Expand the sliding window by moving right.
        for (int right = 0; right < s.length(); right++) {

            char current = s.charAt(right);

            // Add the current character to the window.
            frequency[current - 'A']++;

            // Track the highest character frequency seen.
            maxFreq = Math.max(
                maxFreq,
                frequency[current - 'A']
            );

            // If too many replacements are required,
            // shrink the window from the left.
            while ((right - left + 1) - maxFreq > k) {
                frequency[s.charAt(left) - 'A']--;
                left++;
            }

            // Record the longest valid window found.
            maxLength = Math.max(
                maxLength,
                right - left + 1
            );
        }

        return maxLength;
    }
}
```

### Complexity

- **Time:** `O(n)` — both pointers move only forward. Each character enters the window once, and each character leaves it at most once.
- **Auxiliary space:** `O(1)` — the frequency array has exactly 26 elements.

---

## 7. Detailed Dry Run

Let's trace:

```text
s = "AABABBA"
k = 1
```

Initially:

```text
left = 0
maxLength = 0
maxFreq = 0
```

We use:

\[
\text{replacementsNeeded}=\text{windowLength}-\text{maxFreq}
\]

### Iteration 1: `right = 0`

Character: `A`

```text
Window: A

A frequency = 1
maxFreq = 1
windowLength = 1
replacementsNeeded = 1 - 1 = 0
```

Valid because `0 <= 1`.

```text
maxLength = 1
```

### Iteration 2: `right = 1`

Character: `A`

```text
Window: A A

A frequency = 2
maxFreq = 2
windowLength = 2
replacementsNeeded = 2 - 2 = 0
```

Valid.

```text
maxLength = 2
```

### Iteration 3: `right = 2`

Character: `B`

```text
Window: A A B

A frequency = 2
B frequency = 1
maxFreq = 2

windowLength = 3
replacementsNeeded = 3 - 2 = 1
```

Valid because one replacement is allowed.

```text
A A B → A A A
```

```text
maxLength = 3
```

### Iteration 4: `right = 3`

Character: `A`

```text
Window: A A B A

A frequency = 3
B frequency = 1
maxFreq = 3

windowLength = 4
replacementsNeeded = 4 - 3 = 1
```

Valid.

```text
A A B A → A A A A
```

```text
maxLength = 4
```

### Iteration 5: `right = 4`

Character: `B`

The window initially becomes:

```text
A A B A B
```

Frequencies:

```text
A = 3
B = 2

maxFreq = 3
windowLength = 5
replacementsNeeded = 5 - 3 = 2
```

Invalid because `2 > k`, where `k = 1`.

We must shrink the window.

**First shrink:**

Remove the leftmost `A`.

```text
left = 1

Remaining window: A B A B
A = 2
B = 2
```

The algorithm's `maxFreq` remains `3`; it is not recalculated downward when a character leaves the window.

So the condition is evaluated as:

```text
windowLength = 4
maxFreq = 3
4 - 3 = 1
```

The condition is now false, so shrinking stops.

```text
maxLength = max(4, 4) = 4
```

### Iteration 6: `right = 5`

Character: `B`

The current window expands to:

```text
A B A B B
```

Actual frequencies:

```text
A = 2
B = 3
```

`maxFreq` becomes `3`.

```text
windowLength = 5
windowLength - maxFreq = 5 - 3 = 2
```

Invalid, so shrink.

Remove the leftmost `A`:

```text
left = 2

Remaining window: B A B B
```

The frequency of `A` decreases from `2` to `1`.

The condition becomes:

```text
windowLength = 4
maxFreq = 3
4 - 3 = 1
```

Valid.

```text
maxLength = max(4, 4) = 4
```

### Iteration 7: `right = 6`

Character: `A`

The window expands:

```text
B A B B A
```

Frequencies:

```text
A = 2
B = 3
maxFreq = 3
windowLength = 5
```

Required replacements:

```text
5 - 3 = 2
```

Invalid, so shrink.

Remove the leftmost `B`:

```text
left = 3

Remaining window: A B B A
```

The actual frequencies are now:

```text
A = 2
B = 2
```

However, `maxFreq` remains `3`.

The algorithm evaluates:

```text
windowLength = 4
4 - 3 = 1
```

The condition is now false, so it stops shrinking.

```text
maxLength = max(4, 4) = 4
```

### Final result

```text
Input:  s = "AABABBA", k = 1
Output: 4
```

---

## 8. The Tricky Part: Why Don't We Decrease `maxFreq`?

This is the most important concept in LC 424.

When we shrink the window, the character that had the highest frequency might leave the window. So why don't we recalculate `maxFreq`?

For example, suppose:

```text
Window: A A A B
maxFreq = 3
```

After removing an `A`, the window becomes:

```text
Window: A A B
```

Now the actual maximum frequency is `2`, but `maxFreq` still stores `3`.

Isn't that incorrect?

**It is a deliberate optimization.**

The stored `maxFreq` is the highest frequency observed during the window's expansion; it is not necessarily the exact maximum frequency of the current window.

This means the condition:

```java
windowLength - maxFreq > k
```

can sometimes consider a window valid even when its *actual* character frequencies would require more than `k` replacements.

Why is this still sufficient to find the correct maximum length?

1. `maxFreq` never decreases.
2. The algorithm only needs to discover a maximum achievable window length; it doesn't need to maintain the exact maximum frequency after every removal.
3. If a window is genuinely valid, the algorithm can discover it as it expands. The stale `maxFreq` helps avoid unnecessary shrinking, but it does not produce a larger final answer than is achievable by a valid substring.

The key intuition is that when a stale `maxFreq` makes the current window appear valid, the algorithm preserves its length rather than increasing the answer beyond that length. A previously observed frequency justifies that length in a window that was valid when the frequency was reached. This is why the standard algorithm still returns the correct maximum length.

**Do not confuse this optimization with maintaining exact window frequencies.** If you want the exact maximum frequency in the current window at every step, you would need to recompute it or use a different data structure. That is unnecessary for this solution.

---

## GitHub-Ready Notes

```text
Problem: LeetCode 424 — Longest Repeating Character Replacement

Pattern:
Variable-Size Sliding Window + Frequency Array

Approach:
Expand the right pointer and count character frequencies.
Track the maximum frequency seen.
If windowLength - maxFreq > k, shrink from the left.
Update the maximum window length.

Key Formula:
Replacements Needed = Window Length - Maximum Character Frequency

Validity Condition:
windowLength - maxFreq <= k

Time Complexity: O(n)
Auxiliary Space: O(1), because the frequency array has 26 entries.

Key Insight:
Keep the most frequent character and replace all other characters.
The stored maxFreq is allowed to remain stale when shrinking,
avoiding repeated frequency-max calculations.
```

## Quick Revision: LC 3 vs LC 209 vs LC 424

| Problem | Window condition | When to shrink |
|---|---|---|
| LC 3 — Longest Substring Without Repeating Characters | No duplicate characters | While a duplicate exists |
| LC 209 — Minimum Size Subarray Sum | Sum must be at least `target` | While `sum >= target`, updating the minimum length |
| LC 424 — Longest Repeating Character Replacement | `windowLength - maxFreq <= k` | While `windowLength - maxFreq > k` |

### Test your understanding

Answer these without looking back:

1. In `s = "AABABBA", k = 1`, why is the window `"AABA"` valid?
2. Why do we calculate replacements using `windowLength - maxFreq` instead of counting all distinct characters?
3. Why do we use a `while` loop instead of an `if` statement when shrinking?
4. Why can `maxFreq` remain unchanged when the left pointer moves?
5. What would the time complexity become if we recalculated the maximum character frequency by scanning all 26 entries on every iteration?
