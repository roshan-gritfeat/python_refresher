# FizzBuzz (Custom Rules)

**Difficulty:** Easy
**Tags:** loops, math

## Problem

Given an integer `n` and a dictionary of `{divisor: word}` pairs (e.g. `{3: "Fizz", 5: "Buzz"}`), return a list of strings for the numbers `1` to `n` where:

- For each number, check divisors in ascending order and concatenate the words for every divisor that evenly divides the number.
- If no divisor matches, use the string representation of the number itself.

This generalizes classic FizzBuzz to an arbitrary set of divisor/word pairs.

## Example

```
Input: n = 15, rules = {3: "Fizz", 5: "Buzz"}
Output: ["1", "2", "Fizz", "4", "Buzz", "Fizz", "7", "8", "Fizz", "Buzz",
         "11", "Fizz", "13", "14", "FizzBuzz"]
```

```
Input: n = 10, rules = {2: "Even", 5: "Buzz"}
Output: ["1", "Even", "3", "Even", "Buzz", "Even", "7", "Even", "9", "EvenBuzz"]
```

## Constraints

- `1 <= n <= 10^4`
- `1 <= len(rules) <= 10`
- Divisors are positive integers.

## Hints

1. Start with the plain FizzBuzz case (divisors 3 and 5 only) to make sure the basic loop is right.
2. Iterate divisors in a fixed (e.g. sorted) order so the concatenation order is deterministic.
3. Build up a string per number by checking `num % divisor == 0` for each rule; if the string stays empty, fall back to `str(num)`.
