# Problem Solving Using Flowcharts — Concise Notes

The main focus is **solving problems logically without depending on a programming language**. If the logic/pseudocode is correct, the language is only a translator. Pasted text

## 1. N Marks → Average + Pass/Fail

### Problem
- Read `N` marks.
- Every mark must be in `[0,100]`; otherwise print **Invalid** and stop.
- Calculate average.
- **Pass only if every subject has marks ≥ 40.**

### Variables

```text
i = 1
sum = 0
pass = true
```

### Logic

```text
Repeat while i <= N:
    Read marks

    If marks < 0 OR marks > 100:
        Print "Invalid"
        Stop

    sum = sum + marks

    If marks < 40:
        pass = false

    i = i + 1

average = sum / N
Print average, pass
```

`pass` once changed to `false` stays false because **every subject** must have ≥40. Pasted text

Example:

```text
3 → 70 80 90 → Avg = 80, Pass
3 → 60 70 30 → Avg = 160/3, Fail
3 → 60 -4 70 → Invalid → Stop
```

---

# 2. Largest of 3 — Compound Conditions

For `A, B, C`:

```text
If A >= B AND A >= C:
    largest = A
Else if B >= C:
    largest = B
Else:
    largest = C
```

Why don't we compare `B` with `A` again?

If the first condition failed, we already know **A isn't the largest**, so only `B` and `C` need comparison. Pasted text

Works with ties when only the **largest value** is required:

```text
7 7 7 → 7
7 8 2 → 8
7 7 9 → 9
```

### Limitation

This doesn't scale well to hundreds/thousands of numbers → too many manual conditions.

---

# 3. Current Champion Method — Largest of N Numbers

> Maintain the **largest value seen so far**.

For each new number:

```text
If number > largest:
    largest = number
```

Example:

```text
7 → largest = 7
8 → largest = 8
2 → largest remains 8
```

### For N numbers

If constraint says every number is `>= 0`:

```text
largest = -1

For every number:
    If number > largest:
        largest = number

Print largest
```

`-1` works because it is guaranteed to be outside/below the valid range. If negatives are allowed, choose initialization according to the given constraints. Pasted text

### Invariant

> After each comparison, `largest` = greatest value seen **so far**.

This scales to 100, 1,000 or millions of values.

**Note:** It gives the largest **value**, not its position/index. Pasted text

---

# 4. Grade Classification

Example ranges:

```text
90–100 → A
75–89  → B
...
Outside 0–100 → Invalid
```

First validate:

```text
If marks < 0 OR marks > 100:
    Invalid
```

Then check grades **highest to lowest**:

```text
If marks >= 90:
    A
Else if marks >= 75:
    B
Else if marks >= 60:
    ...
```

### Why don't we check upper bounds repeatedly?

After `marks >= 90` fails, we already know:

```text
marks < 90
```

So the next condition only needs the lower boundary.

> **Order of conditions can eliminate redundant checks.** Pasted text

---

# 5. Sum from 1 to N

Mathematically:

```text
Sum = N × (N + 1) / 2
```

But using iteration:

```text
i = 1
sum = 0

while i <= N:
    sum = sum + i
    i = i + 1

Print sum
```

Example:

```text
N = 3

sum = 0
+1 → 1
+2 → 3
+3 → 6
```

### Important

Without:

```text
i = i + 1
```

the condition may never change → **infinite loop**. Pasted text

---

# 6. Sum of Digits

Example:

```text
472 → 4 + 7 + 2 = 13
```

Two crucial operations:

```text
N % 10  → gets last digit
N / 10  → removes last digit (integer division)
```

Example:

```text
472 % 10 = 2
472 / 10 = 47

47 % 10 = 7
47 / 10 = 4

4 % 10 = 4
4 / 10 = 0 → Stop
```

### Algorithm

```text
Read N

If N < 0:
    Invalid

sum = 0

while N > 0:
    digit = N % 10
    sum = sum + digit
    N = N / 10

Print sum
```

Essential pattern:

> **Extract → Process → Remove → Repeat**

Pasted text

### Special Case: `N = 0`

For **sum of digits**:

```text
N = 0
Loop executes 0 times
sum = 0
```

Correct.

But for **counting digits**, `0` requires special handling because `0` itself has **one digit**. Pasted text

---

# Quick Revision

```text
Problem Solving
│
├── Validate input before processing
├── Iteration → repeat for N inputs
├── Accumulator → sum = sum + value
├── Flag → pass = true/false
├── Compound condition → A>=B AND A>=C
├── Current champion → largest seen so far
├── Ordered conditions → avoid redundant checks
├── Loop update → required for termination
└── Digit manipulation
      ├── N % 10 → extract last digit
      └── N / 10 → remove last digit
```

### Core takeaway

> **Logic first → Flowchart/Pseudocode → Programming language last.**

The lecture deliberately solves these problems without a programming language to reinforce that **DSA/problem-solving is language independent**. Pasted text
