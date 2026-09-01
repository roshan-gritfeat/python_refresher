# Implement Trie (Prefix Tree)

**Difficulty:** Hard
**Tags:** trie, design, string

## Problem

Implement a trie (prefix tree) with the following operations:

- `insert(word)` — inserts the string `word` into the trie.
- `search(word)` — returns `True` if `word` is in the trie (as a complete, previously inserted word).
- `starts_with(prefix)` — returns `True` if any previously inserted word starts with `prefix`.

As a follow-up, extend it to support wildcard search where `.` can match any single character (a common trie extension used in word-search-style problems).

## Example

```
trie = Trie()
trie.insert("apple")
trie.search("apple")     # True
trie.search("app")       # False
trie.starts_with("app")  # True
trie.insert("app")
trie.search("app")       # True
```

## Constraints

- `1 <= len(word), len(prefix) <= 2000`
- Lowercase English letters only.
- Up to `3 * 10^4` calls total across all operations.

## Hints

1. Each trie node needs a way to reference its children (one per possible next character) and a flag marking "a word ends here."
2. A `dict` mapping character -> child node works well as the children structure (avoids allocating 26 slots per node).
3. `insert` walks/creates a chain of nodes character by character, marking `is_end_of_word=True` on the last one. `search` walks the same way but must also check that final flag; `starts_with` walks the same way but doesn't check the flag. For the wildcard follow-up, use recursive DFS that branches over all children when a `.` is encountered.
