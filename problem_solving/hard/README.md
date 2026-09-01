# Hard Problems

This tier is for deliberate, focused practice. A **hard** problem here means:

- Requires a **nontrivial algorithmic insight** — the efficient approach isn't a small tweak on brute force, it's a different idea entirely (e.g. a specific algorithm like Dijkstra, KMP, or Union-Find).
- Often needs careful **optimization** to meet time/space constraints, or careful **design** thinking (state, interfaces, edge cases) for system-y problems.
- May involve concurrency, or model a real backend-engineering concern (rate limiting, caching, thread safety).
- Should take **45+ minutes**, and it's normal to need to look up an algorithm's shape (not its solved-code) before implementing it.

These are meant to stretch you. If you get stuck, that's expected — use the hints, and don't hesitate to research the underlying algorithm (e.g. "how does the union-find data structure work") without looking up a full solution to the specific problem.

## Index

| # | File | Description | Tags |
|---|------|-------------|------|
| 1 | [01-dijkstra-shortest-path.md](01-dijkstra-shortest-path.md) | Find shortest paths from a source in a weighted graph | graph, dijkstra, heap |
| 2 | [02-number-of-connected-components-union-find.md](02-number-of-connected-components-union-find.md) | Count connected components in an undirected graph | graph, union-find |
| 3 | [03-alien-dictionary.md](03-alien-dictionary.md) | Derive character ordering of an alien language from sorted words | graph, topological-sort |
| 4 | [04-implement-trie.md](04-implement-trie.md) | Implement a prefix tree supporting insert/search/startsWith | trie, design, string |
| 5 | [05-kmp-substring-search.md](05-kmp-substring-search.md) | Find all occurrences of a pattern in a text in linear time | string-matching, kmp |
| 6 | [06-lru-cache-design.md](06-lru-cache-design.md) | Design an O(1) get/put least-recently-used cache | design, hash-map, linked-list |
| 7 | [07-rate-limiter-design.md](07-rate-limiter-design.md) | Design a token-bucket API rate limiter | design, system-design |
| 8 | [08-median-of-two-sorted-arrays.md](08-median-of-two-sorted-arrays.md) | Find the median of two sorted arrays in logarithmic time | binary-search, arrays |
| 9 | [09-word-ladder.md](09-word-ladder.md) | Find the shortest transformation sequence length between two words | graph, bfs, string |
| 10 | [10-n-queens.md](10-n-queens.md) | Place n queens on an n x n board so none attack each other | backtracking |
| 11 | [11-edit-distance.md](11-edit-distance.md) | Find the minimum number of edits to transform one string into another | dynamic-programming, string |
| 12 | [12-thread-safe-bounded-blocking-queue.md](12-thread-safe-bounded-blocking-queue.md) | Implement a thread-safe bounded blocking queue | concurrency, design |
