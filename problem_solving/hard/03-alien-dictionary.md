# Alien Dictionary

**Difficulty:** Hard
**Tags:** graph, topological-sort

## Problem

There is a new alien language that uses the English alphabet, but the order among letters is unknown. You are given a list of words from this language's dictionary, where the words are sorted lexicographically according to the alien language's (unknown) rules. Derive one valid ordering of the letters. If no valid ordering exists (contradictory or invalid input), return an empty string/`None`.

## Example

```
Input: words = ["wrt","wrf","er","ett","rftt"]
Output: "wertf"    # one valid ordering (others may also be valid depending on ties)
```

```
Input: words = ["z","x","z"]
Output: ""   # contradiction: z < x AND x < z implied
```

## Constraints

- `1 <= len(words) <= 100`
- `1 <= len(words[i]) <= 100`
- All characters are lowercase English letters.

## Hints

1. Compare each pair of adjacent words to find the *first* differing character — that gives you one ordering constraint (e.g. `'w' comes before 'e'`). Careful: if one word is a prefix of the previous one but comes after it, that's an immediate invalid case.
2. Model letters as nodes in a directed graph with edges representing "comes before" relationships, then this becomes a topological sort problem, just like course scheduling.
3. Build in-degrees for each letter that actually appears in the input, run Kahn's algorithm (repeatedly remove in-degree-0 nodes), and if you can't order all letters that appear (a cycle exists), there's no valid ordering.
