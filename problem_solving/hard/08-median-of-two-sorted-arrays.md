# Median of Two Sorted Arrays

**Difficulty:** Hard
**Tags:** binary-search, arrays

## Problem

Given two sorted arrays `nums1` and `nums2` of sizes `m` and `n` respectively, return the median of the two sorted arrays combined. The overall run time complexity should be **O(log(min(m, n)))**.

## Example

```
Input: nums1 = [1,3], nums2 = [2]
Output: 2.0

Input: nums1 = [1,2], nums2 = [3,4]
Output: 2.5
```

## Constraints

- `0 <= m, n`, and not both empty.
- `-10^6 <= nums1[i], nums2[i] <= 10^6`

## Hints

1. Merging the two arrays and taking the middle is O(m+n) — correct but not fast enough for the intended constraint. The log-time solution uses binary search, not merging.
2. Binary search on the *partition point* of the smaller array. You want to split both arrays into a "left half" and "right half" (combined) such that every element in the left half is `<=` every element in the right half, and the halves are (near-)equal size.
3. For a candidate partition of the smaller array at index `i`, the corresponding partition of the other array is determined by the total left-half size needed (`j = (m+n+1)//2 - i`). Check the boundary values (`nums1[i-1] <= nums2[j]` and `nums2[j-1] <= nums1[i]`); adjust `i` up or down via binary search until this holds, then compute the median from the boundary elements.
