---
title: "Evaluative"
description: "A mathematical challenge involving polynomial evaluation using Horner's method or direct calculation."
date: "2025-09-15"
difficulty: "Easy"
tags:
  - Programming
  - Mathematics
  - Polynomial
  - Python
  - CTF
platform: "HackTheBox"
tools:
  - Python
  - Mathematical Operations
references:
  - https://en.wikipedia.org/wiki/Polynomial
  - https://en.wikipedia.org/wiki/Horner%27s_method
slug: evaluative-hackthebox
---

# Evaluative - HackTheBox CTF Writeup

This challenge involves evaluating a polynomial function given a set of coefficients and a variable value.

## Analysis

The challenge requires us to:
1. Parse a list of coefficients from input
2. Read a variable value (x)
3. Evaluate a polynomial using these coefficients and the variable
4. Output the result

## Solution Approach

The solution implements polynomial evaluation:
1. **Input Parsing**: Read coefficients as space-separated numbers
2. **Variable Input**: Read the value of x
3. **Polynomial Evaluation**: Calculate the sum of each coefficient multiplied by x raised to its corresponding power
4. **Result Output**: Print the final calculated value

### Mathematical Background

The code evaluates a polynomial in the form:
```
f(x) = a₀ + a₁x¹ + a₂x² + a₃x³ + ... + a₈x⁸
```

Where `a₀, a₁, a₂, ..., a₈` are the coefficients provided in the input.

## Solution Code

```python
# take in the number
n = input()
int_list = []
for num in n.split():
    int_list.append(int(num))
x = (int(input()))

# calculate answer
result = 0
i = 0
for i in range(9):
    result = result + int_list[i] * pow(x,i) 

# print answer
print(result)
```

## Code Walkthrough

1. **Coefficient Input**: The code reads a line of space-separated coefficients and stores them in `int_list`
2. **Variable Input**: Reads the value of x as an integer
3. **Polynomial Evaluation**: 
   - Iterates through 9 terms (indices 0-8)
   - For each term i, calculates `coefficient[i] * x^i`
   - Adds each term to the running result
4. **Output**: Prints the final polynomial value


## Mathematical Example

If input coefficients are `[1, 2, 3, 4, 5, 6, 7, 8, 9]` and `x = 2`:
```
f(2) = 1×2⁰ + 2×2¹ + 3×2² + 4×2³ + 5×2⁴ + 6×2⁵ + 7×2⁶ + 8×2⁷ + 9×2⁸
     = 1×1 + 2×2 + 3×4 + 4×8 + 5×16 + 6×32 + 7×64 + 8×128 + 9×256
     = 1 + 4 + 12 + 32 + 80 + 192 + 448 + 1024 + 2304
     = 4097
```
