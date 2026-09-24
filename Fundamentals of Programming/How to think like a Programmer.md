# How to Think Like a Programmer

## 1. Core Idea

> **Syntax expresses a solution; logical thinking creates one.**

Don't jump directly into code. Programming language knowledge doesn't guarantee problem-solving ability.

### Problem-Solving Flow

```text
Understand → Model → Algorithm → Test → Edge Cases → Pseudocode → Code
```

---

## 2. Read the Problem Like a Contract

Identify:

1. **Input** — What is given?
2. **Output** — What is expected?
3. **Conditions** — What determines the result?
4. **Constraints** — What inputs are valid/possible?
5. **Edge Cases** — Where can behavior change/fail?

---

## 3. Example: Larger of Two Numbers

Given `A, B`, find the larger.

### Algorithm

```text
if A > B → A is larger
else if B > A → B is larger
else → both are equal
```

Test with:

```text
8, 13       → 13
100, -4     → 100
7, 7        → Equal ← edge case
```

**Lesson:** Don't hardcode examples. Build a model that works for **every valid input**.

---

## 4. Example: Three Marks

Given `A, B, C`:

```text
Average = (A + B + C) / 3
```

Pass only if:

```text
A >= 40 && B >= 40 && C >= 40
```

Important edge case:

```text
40, 40, 40 → PASS
39, 50, 60 → FAIL
```

**Watch operators carefully:** `>= 40` ≠ `> 40`.

---

# 5. Four Pillars of Problem Solving

### 1. Decomposition
Break a large problem into smaller manageable problems.

Example:

```text
Food Delivery App
├── Account
├── Restaurant
├── Menu
├── Cart
├── Pricing
├── Payment
└── Delivery/Tracking
```

Even these can be decomposed further.

**Large problem → smaller solvable problems.**

### 2. Pattern Recognition
Look for reusable techniques/patterns from previously solved problems.

But:

> **Knowing a pattern doesn't guarantee the solution.**

The pattern may need modification according to the problem and constraints.

### 3. Abstraction
Use existing components/data structures and hide unnecessary implementation details.

Use abstraction **when useful**, not everywhere.

### 4. Algorithmic Thinking
Combine the above into a sequence of precise steps that solves the problem.

---

# 6. Multiple Algorithms Can Solve the Same Problem

Example: Find `"Aman"` among 1000 names.

### Linear Search

Check one by one.

- Works on unsorted data.
- Worst case → check all elements.

### Binary Search

If names are sorted:

- Check middle.
- Eliminate half.
- Repeat.

**Requirement:** Data must be sorted.

### Key Lesson

> Multiple correct algorithms can exist. Compare their efficiency and assumptions.

**Correctness ≠ Efficiency**

---

# 7. Always Question Assumptions

If an algorithm skips something, ask:

> **Why am I allowed to skip it?**

Binary search can eliminate half because the data is **sorted**.

State assumptions explicitly.

---

# 8. Common Beginner Mistakes

❌ Start coding immediately  
✅ Understand → think → design → test → code

❌ Test only provided examples  
✅ Create your own edge cases

❌ Think knowing a pattern guarantees the solution  
✅ Understand and adapt the pattern

❌ Assume there is only one solution  
✅ Multiple solutions may exist

❌ Assume the computer understands your intention  
✅ Computer executes only explicitly written logic

❌ Ignore constraints  
✅ Constraints determine valid inputs and possible edge cases

❌ Present solution immediately in an interview  
✅ Test it and try to break it first

---

# 9. Interview Approach

Before coding:

```text
Input?
Output?
Constraints?
Assumptions?
Approach?
Edge Cases?
Can I break my approach?
```

If you're unsure about an assumption, **ask the interviewer instead of assuming**.

---

# 10. Golden Rules

> **Think before coding.**

> **Test your algorithm before implementing it.**

> **Always try to break your solution.**

> **Don't rely only on given examples.**

> **Constraints determine edge cases.**

> **Patterns are templates, not guaranteed solutions.**

> **Correctness comes first; then optimize efficiency.**

> **Code is just the expression of your algorithm.**

### Final Mental Model

```text
Problem
  ↓
Understand Contract
  ↓
Build Model
  ↓
Decompose / Find Pattern
  ↓
Design Algorithm
  ↓
Test + Break It
  ↓
Code
```
