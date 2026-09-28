# Flowcharts & Pseudocode — DSA Notes

## 1. Problem-Solving Flow

After understanding the problem/contract:

```text
Problem → Contract → Algorithm → Flowchart → Test → Pseudocode → Code
```

* **Contract:** input, output, conditions, assumptions, constraints, corner cases.
* Don't directly code the first solution.
* Test with **multiple inputs + edge cases** before coding.

---

## 2. Flowchart

> **Flowchart = diagrammatic representation of an algorithm/control flow.**

Shows:

* Inputs
* Processing/state changes
* Decisions
* Repetition
* Start/end
* Direction of execution

### Why use it?

* Makes the algorithm easy to visualize.
* Helps expose missing/incorrect logic **before coding**.
* Especially useful for **large/complex systems** involving multiple teams/components.
* For basic DSA problems, you may eventually visualize it mentally; beginners can draw it on paper.

### What a good flowchart should answer

1. Where does execution start/end?
2. What enters and leaves? → input/output
3. What changes?
4. Which decisions choose different paths?
5. What repeats?
6. How does repetition terminate?

> An algorithm is a **finite sequence of steps** → it must eventually terminate.

---

# 3. Flowchart Symbols

| Symbol                       | Meaning                                         |
| ---------------------------- | ----------------------------------------------- |
| **Oval / Rounded rectangle** | Start / End                                     |
| **Parallelogram**            | Input / Output                                  |
| **Rectangle**                | Processing / State change                       |
| **Diamond**                  | Decision (`true/false`)                         |
| **Arrow**                    | Direction/control flow                          |
| **Connector**                | Connects separated parts/pages of a large chart |

### Important

Always **label diamonds with the actual condition**.

❌ Don't make the reader guess what the diamond means.

✅ `Age >= 18?`

---

# 4. Control-Flow Token

Think of a **main control-flow token** travelling through the flowchart.

* Starts at **Start**.
* Follows arrows.
* A process box performs its operation.
* A diamond chooses a path.
* A backward arrow represents repetition.
* It follows **one path at a time**, not multiple paths simultaneously.
* Not every visible box executes in every test case.
* Only the **chosen route** executes.
* Background/concurrent tasks are separate from this main control-flow idea.

---

# 5. Example — Rectangle Area

Given `length` and `width`:

```text
Start
 ↓
Read length
 ↓
Read width
 ↓
area = length × width
 ↓
Display area
 ↓
End
```

Example:

```text
length = 20
width = 30
area = 600
```

Output → `600`

---

# 6. Example — Voting Eligibility

Given `age`:

```text
Start
 ↓
Read age
 ↓
Age >= 18?
 ↙       ↘
Yes       No
 ↓         ↓
Eligible  Not Eligible
  ↘       ↙
     End
```

Examples:

```text
20 → Eligible
17 → Not Eligible
```

A decision creates **branches**, but only one branch is followed for a particular input.

---

# 7. Multi-Way Decision — Number Sign

Given a number:

```text
number > 0 ?
   ↓ Yes → Positive
   ↓ No
number < 0 ?
   ↓ Yes → Negative
   ↓ No  → Zero
```

Examples:

```text
4   → Positive
-2  → Negative
0   → Zero
```

This demonstrates **multiple decisions/branches**.

---

# 8. Iteration

### Problem

Print `1` to `N`.

A hardcoded flowchart like:

```text
Print 1 → Print 2 → Print 3 → End
```

only works for `N = 3`.

### General solution

Use a variable `i`:

```text
Start
 ↓
Read N
 ↓
i = 1
 ↓
i <= N?
 ↙       ↘
Yes       No
 ↓         ↓
Print i   End
 ↓
i = i + 1
 ↓
↖── back to decision
```

### Example: N = 3

```text
i = 1 → print 1 → i = 2
i = 2 → print 2 → i = 3
i = 3 → print 3 → i = 4
i = 4 → 4 <= 3? No → End
```

### Key idea

> **Iteration = repeat a set of steps until a termination condition becomes false.**

The same logic works for **any valid N**.

---

# 9. Iteration + Decision

### Problem

Given `N` students, read each student's marks and print:

* `Pass` if marks `>= 40`
* `Fail` otherwise.

Algorithm:

```text
Read N
i = 1

while i <= N:
    Read marks

    if marks >= 40:
        Pass
    else:
        Fail

    i = i + 1
```

Example:

```text
N = 3

42 → Pass
39 → Fail
40 → Pass
```

Important:

* Outer logic = iteration through students.
* Inner logic = decision for each student's marks.
* `i` must be updated; otherwise the loop may never terminate.

---

# 10. Nested Control

> **Nested logic = one familiar control structure placed inside another.**

Examples:

```text
if
   └── if
```

or:

```text
loop
   └── if
```

or:

```text
loop
   └── loop
```

Can have multiple levels of nesting.

Detailed nested patterns will be covered later.

---

# 11. Pseudocode

> **Pseudocode = structured, language-independent description of an algorithm.**

It is:

* More precise than casual English.
* Less strict than actual programming syntax.
* Not tied to Java, C++, Python, etc.
* Used to inspect **order, decisions, repetition and termination** before coding.

### No universal pseudocode grammar

Different people may write:

```text
Input age
```

or:

```text
Read age
```

Both can be valid if the meaning is clear and consistent.

### Common action words

```text
READ / INPUT  → take input
DISPLAY / PRINT → produce output
SET            → assign a value
IF             → decision
ELSE           → alternative branch
WHILE          → repetition
FOR            → repetition
RETURN         → return result
STOP           → terminate
```

---

# 12. Flowchart vs Pseudocode

| Flowchart                  | Pseudocode                   |
| -------------------------- | ---------------------------- |
| Visual                     | Text-based                   |
| Uses symbols/arrows        | Uses structured instructions |
| Shows paths visually       | Shows logic sequentially     |
| Useful for complex systems | Useful before actual coding  |
| Language independent       | Language independent         |

---

# 13. Overall Workflow to Remember

```text
1. Understand problem
2. Extract contract
3. Design algorithm
4. Visualize with flowchart
5. Test normal + corner cases
6. Write pseudocode
7. Implement in any language
```

### Golden Principle

> **First make the logic correct; programming language comes at the end.**

### Quick Revision

```text
Flowchart = Visual algorithm
Pseudocode = Textual algorithm
Diamond = Decision
Rectangle = Processing
Parallelogram = Input/Output
Oval = Start/End
Arrow = Flow
Backward arrow = Repetition
Connector = Separate chart sections
```
