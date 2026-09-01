# Coin Change

**Difficulty:** Medium
**Tags:** dynamic-programming

## Problem

You are given an integer array `coins` representing coin denominations and an integer `amount`. Return the fewest number of coins needed to make up that amount. If it's impossible, return `-1`. You have an unlimited supply of each coin.

## Example

```
Input: coins = [1,2,5], amount = 11
Output: 3        # 5 + 5 + 1

Input: coins = [2], amount = 3
Output: -1
```

## Constraints

- `1 <= len(coins) <= 12`
- `1 <= coins[i] <= 2^31 - 1`
- `0 <= amount <= 10^4`

## Hints

1. Greedy (always take the largest coin that fits) fails for some denominations — this needs to consider all combinations, which suggests dynamic programming.
2. Define `dp[x]` = minimum coins needed to make amount `x`. What's the recurrence relating `dp[x]` to `dp[x - coin]` for each coin?
3. `dp[x] = 1 + min(dp[x - coin] for coin in coins if coin <= x)`, with `dp[0] = 0` and unreachable amounts marked as infinity. Build the table bottom-up from `0` to `amount`.
