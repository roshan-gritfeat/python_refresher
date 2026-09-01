# LRU Cache Design

**Difficulty:** Hard
**Tags:** design, hash-map, linked-list

## Problem

Design a data structure for a Least Recently Used (LRU) cache with a fixed capacity. It should support:

- `get(key)` — return the value if the key exists, else `-1`. This access counts as "recently used."
- `put(key, value)` — insert or update the value. If inserting causes the cache to exceed capacity, evict the least recently used entry first.

Both operations must run in **O(1) average time** (using Python's built-in `dict` alone won't give you O(1) eviction-by-recency without more structure — that's the crux of the problem).

## Example

```
cache = LRUCache(capacity=2)
cache.put(1, 1)
cache.put(2, 2)
cache.get(1)       # returns 1, and 1 is now "most recently used"
cache.put(3, 3)    # evicts key 2 (least recently used)
cache.get(2)       # returns -1 (not found)
cache.put(4, 4)    # evicts key 1
cache.get(1)       # returns -1
cache.get(3)       # returns 3
cache.get(4)       # returns 4
```

## Constraints

- `1 <= capacity <= 3000`
- Up to `2 * 10^5` calls to `get`/`put`.
- Key/value are integers.

## Hints

1. A dict alone gives O(1) lookup but has no notion of "order of use" that you can cheaply reorder and cheaply evict from either end.
2. Combine a hash map (key -> node) with a **doubly linked list** that maintains usage order (most-recently-used at one end, least at the other). Every access moves a node to the "recently used" end in O(1) via pointer surgery.
3. In Python, `collections.OrderedDict` actually gives you this for free (`move_to_end` and `popitem(last=False)`), but implement it once yourself with an explicit doubly linked list + dict to understand the mechanics before reaching for the shortcut.
