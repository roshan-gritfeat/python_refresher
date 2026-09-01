# Word Ladder

**Difficulty:** Hard
**Tags:** graph, bfs, string

## Problem

Given two words `begin_word` and `end_word`, and a dictionary `word_list`, return the length of the shortest transformation sequence from `begin_word` to `end_word`, such that only one letter can be changed at a time, and each transformed word must exist in `word_list`. Return `0` if no such sequence exists.

## Example

```
Input: begin_word = "hit", end_word = "cog",
       word_list = ["hot","dot","dog","lot","log","cog"]
Output: 5     # "hit" -> "hot" -> "dot" -> "dog" -> "cog"
```

```
Input: begin_word = "hit", end_word = "cog",
       word_list = ["hot","dot","dog","lot","log"]   # "cog" missing
Output: 0
```

## Constraints

- `1 <= len(begin_word) <= 10`
- All words have the same length and consist of lowercase English letters.
- `1 <= len(word_list) <= 5000`

## Hints

1. "Shortest transformation sequence" over a graph where edges connect words differing by one letter is exactly a shortest-path problem on an unweighted graph — that means BFS, not DFS.
2. The naive way to find neighbors of a word (compare it against every other word in the list) is O(len(word_list)) per word, which is expensive. Instead, generate all possible one-letter-changed variants of the current word (e.g. `"hit"` -> `"ait", "bit", ..., "hia", "hib", ...`) and check membership against a *set* of the word list in O(1).
3. Run standard BFS from `begin_word`, tracking distance; each time you pop a word, generate its variants, and for any variant present in the (mutable, so you can remove-as-visited) word set, enqueue it and remove it from the set to avoid revisiting.
