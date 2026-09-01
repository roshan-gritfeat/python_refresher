# Rate Limiter Design (Token Bucket)

**Difficulty:** Hard
**Tags:** design, system-design

## Problem

Design a per-client API rate limiter using the **token bucket** algorithm: each client has a bucket with a maximum capacity of `N` tokens that refills at a rate of `R` tokens per second (never exceeding capacity). Each request costs 1 token. Implement:

- `allow_request(client_id)` — returns `True` (and consumes a token) if a token is available for that client right now, `False` otherwise (request should be rejected/throttled).

Design this to be reasonably memory-efficient for many distinct clients (i.e. don't literally schedule a timer to "add a token" every tick for every client).

## Example

```
limiter = TokenBucketLimiter(capacity=5, refill_rate=1)  # 5 tokens, refills 1/sec
limiter.allow_request("clientA")   # True (5 -> 4 tokens)
# ... 4 more quick calls exhaust the bucket ...
limiter.allow_request("clientA")   # False (0 tokens left)
# after waiting ~1 second
limiter.allow_request("clientA")   # True (1 token refilled)
```

## Constraints

- Must support many independent clients without pre-allocating state for clients that never make a request.
- Should be correct even if `allow_request` calls for the same client arrive from multiple threads concurrently.

## Hints

1. You don't need a background thread ticking constantly — instead, compute how many tokens *should* have refilled lazily, at the moment a request comes in, based on elapsed time since the last check.
2. Per client, store `(tokens, last_refill_timestamp)`. On each request: `tokens = min(capacity, tokens + elapsed_seconds * refill_rate)`, update the timestamp, then check if `tokens >= 1`.
3. For thread safety, guard the read-modify-write of a client's bucket state with a lock (e.g. one lock per client, or a striped lock scheme, to avoid a single global lock serializing all clients). Think about what happens if two threads for the same client race on the same bucket — that's exactly the bug a naive implementation would have.
