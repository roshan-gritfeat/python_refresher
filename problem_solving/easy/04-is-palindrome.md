# Is Palindrome

**Difficulty:** Easy
**Tags:** string, two-pointer

## Problem

Given a string, determine if it is a palindrome, considering only alphanumeric characters and ignoring case.

## Example

```
Input: "A man, a plan, a canal: Panama"
Output: True

Input: "race a car"
Output: False

Input: " "
Output: True
```

## Constraints

- `0 <= len(s) <= 2 * 10^5`
- `s` consists of printable ASCII characters.

## Hints

1. First figure out how to filter out non-alphanumeric characters and normalize case.
2. Once you have a "cleaned" string, comparing it to its reverse is the simplest approach.
3. For an O(1) extra space approach, use two pointers from both ends and skip non-alphanumeric characters as you go, instead of building a new string.
