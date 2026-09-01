# Valid Parentheses

**Difficulty:** Easy
**Tags:** stack, string

## Problem

Given a string containing just the characters `(`, `)`, `{`, `}`, `[`, `]`, determine if the input string is valid. A string is valid if brackets close in the correct order and every opening bracket has a matching closing bracket of the same type.

## Example

```
Input: "()[]{}"
Output: True

Input: "(]"
Output: False

Input: "([)]"
Output: False

Input: "{[]}"
Output: True
```

## Constraints

- `1 <= len(s) <= 10^4`
- `s` consists only of bracket characters.

## Hints

1. Think about what data structure naturally handles "most recently opened, first closed" ordering.
2. Push opening brackets onto a stack; when you see a closing bracket, it must match the top of the stack.
3. At the end, the stack must be empty for the string to be valid.
