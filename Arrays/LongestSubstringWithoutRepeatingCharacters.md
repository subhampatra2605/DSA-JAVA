# **LeetCode 3 — Longest Substring Without Repeating Characters**



Medium

Sliding Window + HashSet / HashMap



This is one of the most important variable-size sliding-window problems.

The objective is to find the **length of the longest contiguous substring that contains no repeated characters**.

The key idea:

**Expand the window when the character is unique. If a duplicate appears, shrink the window from the left until the duplicate is removed.**

## **1. Understand the problem**

You are given a string `s`. Find the length of the longest substring without repeating characters.

Remember:

- A substring must be contiguous.
- Characters cannot repeat inside the substring.
- Return the length, not the substring itself.

**Example 1**

```
Input:  s = "abcabcbb"
Output: 3
```

Possible substrings without repeated characters:

```
"abc" → length 3
"bca" → length 3
"cab" → length 3
```

The answer is `3`.

**Example 2**

```
Input:  s = "bbbbb"
Output: 1
```

Every character is repeated, so the longest valid substring is just `"b"`.

**Example 3**

```
Input:  s = "pwwkew"
Output: 3
```

One longest substring is `"wke"`.

Notice that `"pwke"` is not a valid answer because it is not contiguous.

## **2. Brute-force approach**

Let's first generate every possible substring and check whether it contains duplicate characters.

For every starting index:

1. Create an empty `HashSet`.
2. Extend the substring one character at a time.
3. If the character is already in the set, stop extending.
4. Otherwise, add it and update the maximum length.

### **Java code — Brute force**

```
import java.util.HashSet;
import java.util.Set;

class Solution {
    public int lengthOfLongestSubstring(String s) {

        int maxLength = 0;

        // Try every possible starting position.
        for (int start = 0; start < s.length(); start++) {

            // Store characters in the current substring.
            Set<Character> seen = new HashSet<>();

            // Extend the substring from start.
            for (int end = start; end < s.length(); end++) {

                char current = s.charAt(end);

                // Stop if this character is already present.
                if (seen.contains(current)) {
                    break;
                }

                // Add the new unique character.
                seen.add(current);

                // Calculate current substring length.
                int length = end - start + 1;

                // Update the maximum length.
                maxLength = Math.max(maxLength, length);
            }
        }

        return maxLength;
    }
}
```

**Complexity:**

- Time: \\(O(n^2)\\) average — there are \\(O(n^2)\\) candidate substrings, with expected constant-time set operations.
- Space: \\(O(\min(n,\Sigma))\\), where \\(\Sigma\\) is the character-set size.

## **3. Identify the bottleneck**

The brute-force solution repeatedly checks characters that were already examined.

Consider:

```
s = "abcabcbb"
```

We examine:

```
Starting at index 0: "abc"
Starting at index 1: "bca"
Starting at index 2: "cab"
```

Notice that characters are repeatedly processed.

Can we avoid starting from scratch every time?

Yes. Instead of checking every substring independently, maintain a window and reuse the information from the previous window.

## **4. Recognize the pattern — Variable-size Sliding Window**

We need two pointers:

- `left`: beginning of the current window.
- `right`: end of the current window.

We also need a `HashSet` to track the characters currently inside the window.

Our condition is:

\\[ \text{Every character in the window must be unique} \\]

The window is valid when the current character is not already present in the set.

There are two possibilities:

| Situation                            | Action                                                  |
| ------------------------------------ | ------------------------------------------------------- |
| Current character is not present     | Add it and expand the window                            |
| Current character is already present | Remove characters from the left until it becomes unique |

Our objective is:

\\[ \text{maxLength}=\max(\text{right}-\text{left}+1) \\]

## **5. Derive the optimal approach**

Let's understand each operation.

### **Step 1 — Initialize variables**

```
int left = 0;
int maxLength = 0;

Set<Character> seen = new HashSet<>();
```

- `left` tracks the beginning of the window.
- `maxLength` stores the longest valid window.
- `seen` stores characters currently in the window.

### **Step 2 — Expand using the right pointer**

```
for (int right = 0; right < s.length(); right++)
```

Get the current character:

```
char current = s.charAt(right);
```

### **Step 3 — Check for duplicates**

```
while (seen.contains(current))
```

If the current character already exists, the window is invalid.

We need to remove characters from the left:

```
seen.remove(s.charAt(left));
left++;
```

Why do we use `while` instead of `if`?

Because the duplicate character might not be at the beginning of the window.

For example:

```
Window = "abc"
Current character = 'c'
```

We must remove:

```
'a' → window becomes "bc"
'b' → window becomes "c"
```

Only then can we safely add the new `'c'`.

### **Step 4 — Add the current character**

```
seen.add(current);
```

Now the window contains unique characters again.

### **Step 5 — Update maximum length**

```
int length = right - left + 1;

maxLength = Math.max(maxLength, length);
```

We calculate the length after restoring the window's validity.

## **6. Optimal Java solution — HashSet + Sliding Window**

```
import java.util.HashSet;
import java.util.Set;

class Solution {
    public int lengthOfLongestSubstring(String s) {

        int left = 0;
        int maxLength = 0;

        // Stores unique characters in the current window.
        Set<Character> seen = new HashSet<>();

        // Expand the window using the right pointer.
        for (int right = 0; right < s.length(); right++) {

            char current = s.charAt(right);

            // Shrink the window while the current
            // character is already present.
            while (seen.contains(current)) {

                // Remove the leftmost character.
                seen.remove(s.charAt(left));

                // Move the left boundary forward.
                left++;
            }

            // Add the current character to the window.
            seen.add(current);

            // Calculate the length of the valid window.
            int length = right - left + 1;

            // Update the maximum substring length.
            maxLength = Math.max(maxLength, length);
        }

        return maxLength;
    }
}
```

Optimal

- **Time:** \\(O(n)\\) average — each character enters and leaves the set at most once.
- **Space:** \\(O(\min(n,\Sigma))\\) — the set stores only characters in the current window.

### **Another optimal approach — HashMap**

There is an alternative that is especially useful for interviews.

Instead of removing characters one by one, store the **last index where each character appeared**.

The idea:

If the current character was previously seen at index `j`, move `left` directly to `j + 1`, provided that this does not move `left` backward.

```
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int lengthOfLongestSubstring(String s) {

        Map<Character, Integer> lastSeen = new HashMap<>();

        int left = 0;
        int maxLength = 0;

        for (int right = 0; right < s.length(); right++) {

            char current = s.charAt(right);

            // If character was seen inside the current window,
            // move left just beyond its previous occurrence.
            if (lastSeen.containsKey(current)) {
                left = Math.max(left, lastSeen.get(current) + 1);
            }

            // Store the latest index of this character.
            lastSeen.put(current, right);

            // Calculate the current window length.
            int length = right - left + 1;

            maxLength = Math.max(maxLength, length);
        }

        return maxLength;
    }
}
```

**Why is&#x20;**`Math.max()`**&#x20;important?**

Consider:

```
s = "abba"
```

When we reach the last `'a'`, its previous occurrence is at index 0. But `left` is already at index 2.

We must not move `left` backward to index 1.

```
left = Math.max(left, lastSeen.get(current) + 1);
```

This guarantees that the window's left boundary only moves forward.

Both approaches are \\(O(n)\\) average time. The HashSet approach is easier to understand initially; the HashMap approach can skip multiple unnecessary left-pointer movements.

## **7. Detailed dry run — HashSet approach**

Input:

```
s = "abcabcbb"
```

Initially:

```
left = 0
maxLength = 0
seen = {}
```

Let's trace every iteration.

| right | Character | Action                      | Current window | maxLength |
| ----- | --------- | --------------------------- | -------------- | --------- |
| 0     | a         | Add                         | `a`            | 1         |
| 1     | b         | Add                         | `ab`           | 2         |
| 2     | c         | Add                         | `abc`          | 3         |
| 3     | a         | Remove old `a`, add new `a` | `bca`          | 3         |
| 4     | b         | Remove old `b`, add new `b` | `cab`          | 3         |
| 5     | c         | Remove old `c`, add new `c` | `abc`          | 3         |
| 6     | b         | Shrink, then add `b`        | `cb`           | 3         |
| 7     | b         | Shrink, then add `b`        | `b`            | 3         |

Let's understand the important iterations.

### **Iteration 1 — right = 3, current = 'a'**

Before this iteration:

```
Window = "abc"
seen = {a, b, c}
left = 0
```

We encounter another `'a'`.

```
while (seen.contains('a'))
```

Since `'a'` already exists, remove characters from the left.

**First removal:**

```
Remove s[0] = 'a'
left = 1

Window = "bc"
seen = {b, c}
```

Now `'a'` is no longer present.

Add the new `'a'`:

```
Window = "bca"
seen = {a, b, c}
```

Calculate:

```
length = right - left + 1
       = 3 - 1 + 1
       = 3
```

`maxLength` remains 3.

### **Iteration 2 — right = 4, current = 'b'**

Current window:

```
"bca"
```

The new character `'b'` is already present.

Remove characters from the left:

```
Remove 'b'
left = 2

Window = "ca"
```

Now add the new `'b'`:

```
Window = "cab"
```

Length = 3.

### **Iteration 3 — right = 5, current = 'c'**

Current window:

```
"cab"
```

`'c'` is already present.

Remove `'c'` from the left:

```
Window = "ab"
left = 3
```

Add the new `'c'`:

```
Window = "abc"
```

Length = 3.

### **Iteration 4 — right = 6, current = 'b'**

Current window:

```
"abc"
```

`'b'` is already present.

Remove from the left:

```
Remove 'a' → window = "bc"
Remove 'b' → window = "c"
```

Now add the new `'b'`:

```
Window = "cb"
```

Length = 2.

### **Iteration 5 — right = 7, current = 'b'**

Current window:

```
"cb"
```

Another `'b'` appears.

Remove:

```
Remove 'c' → window = "b"
```

The old `'b'` is still in the set, so remove it:

```
Remove 'b' → window = ""
```

Add the new `'b'`:

```
Window = "b"
```

Length = 1.

**Final result**

# **3**

Longest substring length

Valid longest substrings include:

- `abc`
- `bca`
- `cab`

## **8. Tricky parts and interview insights**

### **A. Why is&#x20;**`while`**&#x20;necessary instead of&#x20;**`if`**?**

Consider:

```
s = "abca"
```

When the second `'a'` arrives, the current window is `"abc"`.

Removing just one character gives:

```
"bc"
```

The old `'a'` is gone, so in this example one removal is enough.

But consider:

```
s = "abcb"
```

When the second `'b'` arrives:

```
Current window = "abc"
```

We need to remove:

```
'a' → "bc"
'b' → "c"
```

Only then can we add the new `'b'`.

Therefore, `while` is necessary.

### **B. Why is the window length&#x20;**`right - left + 1`**?**

Both indices are inclusive.

For example:

```
left = 2
right = 5
```

The window contains indices:

```
2, 3, 4, 5
```

That's four characters.

Therefore:

\\[ 5-2+1=4 \\]

### **C. Why do we remove the leftmost character from the HashSet?**

Because the window must always contain unique characters.

When a duplicate appears, we remove characters from the left until the duplicate is gone.

The HashSet represents exactly the characters inside the current window.

### **D. Why is the HashMap approach's&#x20;**`Math.max()`**&#x20;essential?**

Consider:

```
s = "abba"
```

At the final `'a'`:

- Previous `'a'` index = 0.
- Current `left` = 2.
- Previous occurrence + 1 = 1.

If we blindly assign `left = 1`, we move backward and incorrectly include the old `'b'`.

Using:

```
left = Math.max(left, lastSeen.get(current) + 1);
```

keeps `left` at 2.

### **E. Why is the solution O(n)?**

Even though we have a `while` loop inside a `for` loop:

- `right` visits each character once.
- `left` only moves forward.
- Each character is inserted into and removed from the set at most once.

Total expected time is \\(O(n)\\).

## **9. Compare related problems**

| Problem | Pattern                 | Window condition        |
| ------- | ----------------------- | ----------------------- |
| LC 3    | Variable sliding window | No repeated characters  |
| LC 1004 | Variable sliding window | At most k zeros         |
| LC 424  | Variable sliding window | Replacements needed ≤ k |
| LC 209  | Variable sliding window | Sum ≥ target            |

The important difference is what makes a window valid.

For LC 3, the window is valid only when every character is unique.

## **GitHub-ready notes — LC 3**

**Problem:** Find the length of the longest substring without repeating characters.

**Approach:**

1. Initialize `left = 0`, `maxLength = 0`, and an empty HashSet.
2. Expand the window using `right`.
3. If the current character already exists, shrink from the left using a `while` loop.
4. Remove outgoing characters from the HashSet.
5. Add the current character.
6. Calculate `right - left + 1`.
7. Update `maxLength`.

**Pattern:** Variable-size Sliding Window + Two Pointers + HashSet

**Time Complexity:** \\(O(n)\\) average.

**Space Complexity:** \\(O(\min(n,\Sigma))\\).

**Key Insights:**

- A substring must be contiguous.
- HashSet tracks unique characters inside the current window.
- Use `while` to remove duplicates completely.
- Update the maximum only after the window becomes valid.
- HashMap with last-seen indices is an alternative that can jump `left` directly.

## **10. Test your understanding**

Try these without looking at the solution.

**Q1. For s = "abcabcbb", what is the answer?**

2

3

4



**Q2. Why do we use while (seen.contains(current))?**

To remove every character from the string

To ensure the duplicate is completely removed from the window

To increase the window size



**Q3. For s = "abba", why is Math.max(left, lastSeen.get(current) + 1) needed in the HashMap approach?**

To prevent left from moving backward

To sort the characters

To count duplicate characters



**Q4. What is the time complexity of the HashSet sliding-window approach?**

O(n²)

O(n log n)

O(n) average



Check answers

**Recommended next:** LC 76 — Minimum Window Substring. It uses a variable-size sliding window with a HashMap/frequency-count technique and is a very useful next challenge after LC 3.
