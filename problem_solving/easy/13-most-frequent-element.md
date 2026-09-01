# Most Frequent Element

**Difficulty:** Easy
**Tags:** hash-map, counting

## Problem

Given a list of elements, return the element that occurs most frequently. If there is a tie, return any one of the tied elements.

## Example

```
Input: [1, 3, 2, 1, 4, 1, 3]
Output: 1        # appears 3 times
```

```
Input: ["a", "b", "a", "b", "c"]
Output: "a" or "b"   # tied at 2
```

## Constraints

- `1 <= len(items) <= 10^5`
- Elements are hashable.

## Hints

1. You need a count of every distinct element — a dictionary is the natural fit.
2. `collections.Counter(items)` builds the frequency map for you in one call.
3. `Counter.most_common(1)` gives you the top element and its count directly.
