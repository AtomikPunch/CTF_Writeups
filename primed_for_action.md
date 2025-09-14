---
title: "Primed for Action"
description: "A programming challenge involving prime number detection and multiplication to find the flag."
date: "2025-09-15"
difficulty: "Easy"
tags:
  - Programming
  - Mathematics
  - Prime Numbers
  - Python
  - CTF
platform: "HackTheBox"
tools:
  - Python
  - Basic Math
references:
  - https://en.wikipedia.org/wiki/Prime_number
  - https://en.wikipedia.org/wiki/Primality_test
slug: primed-for-action-hackthebox
---

# Primed for Action - HackTheBox CTF Writeup

This challenge appears to involve prime number calculations based on a given set of numbers.

## Analysis

Looking at the challenge, we need to:
1. Parse a list of numbers from input
2. Identify which numbers are prime
3. Perform some operation on the prime numbers to get the flag

## Solution Approach

The solution involves:
1. **Input Parsing**: Read a space-separated list of numbers
2. **Prime Detection**: Check each number to determine if it's prime
3. **Result Calculation**: Multiply the first two prime numbers found

### Prime Number Detection Algorithm

The code uses a basic primality test:
- For each number, check if it has any divisors between 2 and the number itself
- If no divisors are found, the number is prime
- Numbers less than 2 are handled implicitly (they won't be prime)

## Solution Code

```python
# take in the number
n = input()
int_list = []
prime_number = []

# Parse input into integers
for num in n.split():
    int_list.append(int(num))

# Find prime numbers
for i in range(len(int_list)):
    is_prime = True
    for j in range(2, int_list[i]):
        if (int_list[i] % j) == 0:
            is_prime = False
            break
    if is_prime:
        prime_number.append(int_list[i])

# Calculate result (multiply first two primes)
result = prime_number[0] * prime_number[1]

# Output the answer
print(result)
```

## Code Walkthrough

1. **Input Processing**: The code reads a line of space-separated numbers and converts them to integers
2. **Prime Checking**: For each number, it checks divisibility from 2 up to the number itself
3. **Prime Collection**: Prime numbers are stored in a separate list
4. **Final Calculation**: The product of the first two prime numbers is calculated and output