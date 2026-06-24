# Big O Notation — Study Notes

---

## What is Big O?

Big O describes **how fast an algorithm slows down as the input grows**. It is not the exact runtime — it is the **rate of growth**.

```
n = input size
```

---

## Common Complexities — Fastest to Slowest

| Complexity | Name | Example |
|---|---|---|
| O(1) | Constant | `arr[0]` — always 1 operation |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Single loop |
| O(n²) | Quadratic | Two nested loops |
| O(2ⁿ) | Exponential | Recursive fibonacci |

**Intuition with n = 1000:**
- O(1) → **1** operation
- O(log n) → **10** operations
- O(n) → **1,000** operations
- O(n²) → **1,000,000** operations
- O(2ⁿ) → 💀

---

## Log n

Logarithm answers: **"How many times do we multiply 2 to reach n?"**

```
log₂(8)    = 3   →  2³ = 8
log₂(16)   = 4   →  2⁴ = 16
log₂(1024) = 10  →  2¹⁰ = 1024
```

**In an algorithm:** if you are **halving the problem at every step** → O(log n)

```
Binary search on n=16:

[1..16] → [9..16] → [13..16] → [15..16] → [16] ✓

16 elements → 4 steps
log₂(16) = 4 ✓
```

**Non-integer example:**
```
log₂(25) ≈ 4.64  →  ~4-5 steps in practice
```

### Important notes on log n

> **The base of the logarithm does not matter in Big O.**
> Big O measures approximate growth rate, so log₂(n) and log₁₀(n) are both written as O(log n).
> The base is a constant — and constants are dropped in Big O.

---

## Rules

### 1. Constants and small terms are dropped

```
O(2n)     → O(n)
O(n + 5)  → O(n)
O(3n²)    → O(n²)
O(n²/2 + n/2) → O(n²)
```

### 2. The dominant term absorbs smaller terms

```
O(n) + O(n²) = O(n²)
```

> O(n²) is so dominant next to O(n) that O(n) is simply discarded.

### 3. Addition rule → Sequential operations

```python
for i in range(n):   # O(n)
    print(i)

for i in range(n):   # O(n)
    print(i)
```

```
O(n) + O(n) = 2·O(n) = O(n)
```

### 4. Multiplication rule → Nested operations

```python
for i in range(n):       # O(n)
    for j in range(n):   # O(n)
        print(i, j)
```

```
O(n) × O(n) = O(n²)
```

**The key question:**
> "Is the inner operation dependent on the outer one?"
> - Yes → multiply → O(n²)
> - No  → add → O(n)

---

## O(log n) vs O(n)

```
O(log n) → problem is HALVED at every step
O(n)     → problem DECREASES by one (or closes from both sides)
```

**O(log n) example:**
```python
while n > 1:
    n = n // 2  # halved every step
```

**O(n) example — two pointers closing in:**
```python
left, right = 0, len(arr) - 1
while left < right:
    left += 1
    right -= 1
# n=8 → 4 steps = n/2 → still O(n), not O(log n)
```

---

## Watch Out — Hidden O(n) Operations

These operations are **O(n)** and are easy to miss inside loops:

| Operation | Why O(n) |
|---|---|
| `s[i:j]` | Copies j-i characters |
| `"".join(arr)` | Iterates over all elements |
| `s.find(x)` | Scans the string |

---

## Exercises

### Exercise 1
```python
def foo(n):
    for i in range(n):        # O(n)
        print(i)

    for i in range(n):        # O(n)
        for j in range(n):    # O(n)
            print(i, j)
```
```
O(n) + O(n²) = O(n²)   ← dominant term wins
```

---

### Exercise 2
```python
def foo(s):
    for i in range(len(s)):
        for j in range(len(s)):
            if s[i] == s[j]:
                print(s[i:j])   # ← hidden O(n)!
```
```
O(n) × O(n) × O(n) = O(n³)
```

---

### Exercise 3
```python
left, right = 0, len(arr) - 1
while left < right:
    left += 1
    right -= 1
```
```
n=8 → 4 steps → n/2 steps → O(n)
Not O(log n) — the problem shrinks by 1 each side, it is not halved.
```

---

### Exercise 4 — Expand Around Center (Longest Palindrome)
```python
for i in range(len(s)):         # O(n)
    pal1 = self.expand(s, i, i)      # O(n)
    pal2 = self.expand(s, i, i + 1)  # O(n)
```
```
pal1 and pal2 are sequential (not nested) → O(n) + O(n) = O(n)
Combined with the outer loop → O(n) × O(n) = O(n²)
```

---

### Exercise 5 — Triangle loop
```python
for i in range(n):
    for j in range(i, n):
        print(i, j)
```
```
Total steps: n + (n-1) + (n-2) + ... + 1 = n(n+1)/2 = n²/2 + n/2
Drop constants and small terms → O(n²)
```

---

---

## Best / Average / Worst Case

```python
def find(arr, target):
    for i in range(len(arr)):
        if arr[i] == target:
            return i
    return -1
```

**Best Case — O(1)**
Target is at the **first index.**
```
[3, 7, 1, 9, 4]  target=3
 ↑ found! → 1 step
```

**Worst Case — O(n)**
Target is at the **end or not in the array.**
```
[3, 7, 1, 9, 4]  target=4
          ↑ → 5 steps
```

**Average Case — O(n)**
On average we expect the element to be in the **middle:**
```
n/2 steps → drop constant → O(n)
```

---

### Which case matters in practice?

| Case | When to use |
|---|---|
| **Worst case** | Almost always — guaranteed upper bound |
| Average case | Realistic performance estimate |
| Best case | Usually misleading — often ignored |

> When we say "this algorithm is O(n)" we mean **worst case O(n)**.

---

### Binary Search — Best / Average / Worst Case

```
n = 8:

Best case:    O(1)      → found at the middle on first look
Average case: O(log n)  → found halfway through
Worst case:   O(log n)  → log₂(8) = 3 steps
```

| Case | Complexity |
|---|---|
| Best | O(1) |
| Average | O(log n) |
| Worst | O(log n) |

---

### Linear Search vs Binary Search

| | Linear Search | Binary Search |
|---|---|---|
| Best | O(1) | O(1) |
| Average | O(n) | O(log n) |
| Worst | O(n) | O(log n) |

**n = 1,000,000 worst case:**
```
Linear search  → 1,000,000 steps
Binary search  → log₂(1,000,000) ≈ 20 steps
```

---

---

## Space Complexity

Space complexity measures **how much extra memory** an algorithm uses as input grows. Same Big O notation, different question:

> "As input grows, how much **extra memory** do we use?"

**Important rule:**
> The input itself is NOT counted. Only the **extra** memory you allocate counts.

---

### O(1) — Constant Space

```python
def sum_array(arr):
    total = 0        # one variable, always
    for x in arr:
        total += x
    return total
```
```
Space: O(1)  — no matter how large arr is, only one extra variable
```

---

### O(n) — Linear Space

```python
def double_array(arr):
    result = []          # grows with input
    for x in arr:
        result.append(x * 2)
    return result
```
```
Space: O(n)  — result grows proportionally with input
```

---

### Recursive Functions — Call Stack

```python
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```

Each recursive call takes up space on the **call stack.**
At any moment, only **one branch** is active:

```
fib(5)
  fib(4)
    fib(3)
      fib(2)
        fib(1)  ← deepest point, max n levels
```

```
Time complexity:  O(2ⁿ)  ← total number of calls
Space complexity: O(n)   ← max call stack depth
```

---

### Example — Two Sum (Hash Map)

```python
def two_sum(nums, target):
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
```

```
seen        → O(n)  ← extra memory, counted ✓
target      → O(1)  ← dropped
complement  → O(1)  ← dropped
nums        → not counted, it's the input itself
```

```
Space complexity: O(n)
```

---

## Amortized Analysis

Some operations are **occasionally expensive** but cheap on average.

```python
arr = []
arr.append(1)  # O(1)
arr.append(2)  # O(1)
...            # at some point, array resizes → O(n)
arr.append(n)  # O(1) again
```

Python lists double in size when full. The resize is O(n), but it happens so rarely that averaged across all appends, each one is still **O(1)**.

> `list.append()` → **Amortized O(1)**

---

## O(n log n)

Sits between O(n) and O(n²). Most efficient sorting algorithms land here.

| Algorithm | Time |
|---|---|
| Merge sort | O(n log n) |
| Heap sort | O(n log n) |
| Quick sort | O(n log n) avg |
| Bubble sort | O(n²) |

**Why n log n?**
In merge sort, you split the array log n times (halving each time), and each level does O(n) work merging:
```
O(log n) levels × O(n) work per level = O(n log n)
```

---

## Space-Time Tradeoff

Improving time often costs space, and vice versa.

```
Two Sum:
  Brute force → O(n²) time, O(1) space   ← slow but memory efficient
  Hash map    → O(n) time,  O(n) space   ← fast but uses extra memory
```

> There is no free lunch — you are almost always trading one for the other.

This is why knowing both time and space complexity matters when choosing an algorithm.
