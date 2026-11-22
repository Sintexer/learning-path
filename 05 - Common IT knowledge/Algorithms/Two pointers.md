**Two Pointers** is an algorithmic pattern that maintains two indices moving through an array, string, or linked list to eliminate redundant work—turning naive \(O(n^2)\) nested loops into \(O(n)\) time and \(O(1)\) extra space.

Two pointers reduces brute-force ==O(n²)== pair or subarray scans to ==O(n) time and O(1) space== when a monotonic invariant lets each step discard candidates safely.

## Three Canonical Variants

1. **Converging (Opposite Ends: `l = 0, r = n - 1`)**
   - Used on sorted arrays or symmetric checks: Two Sum II (`l167`), 3Sum (`l15`), Container With Most Water (`l11`), Trapping Rain Water (`l42`), palindrome checks.
   - Each step moves whichever pointer cannot participate in a better solution, eliminating an entire row/column of the pair matrix.
2. **Same-Direction / Sliding Window (`l`, `r`)**
   - Used for contiguous subarrays/substrings: Longest Substring Without Repeating Characters, Minimum Window Substring.
   - Expand `r` to include elements; shrink `l` whenever the window constraint is violated or can be tightened.
3. **Fast & Slow (Floyd's Tortoise & Hare / Read-Write)**
   - **Cycle detection & middle node**: `slow` moves 1 step, `fast` moves 2 steps.
   - **In-place array compaction**: `read` scans every element while `write` tracks the next output position (e.g., Remove Duplicates from Sorted Array).

In Floyd's cycle detection (fast/slow pointers), after they meet inside a cycle, resetting one pointer to head and advancing both 1 step at a time finds ==the exact start node of the cycle==.

> [!warning] Failure modes & trade-offs
> - **Negative numbers break sliding-window monotonicity**: Variable-size sliding windows for subarray sums rely on expanding `r` monotonically increasing the sum (and shrinking `l` decreasing it). If negative numbers are present, this invariant breaks—use **prefix sums + a hash map** instead.
> - **Duplicate handling in k-Sum**: In 3Sum/4Sum, forgetting to skip duplicate values after recording a match produces duplicate results or out-of-bounds reads.
> - **`l < r` vs `l <= r`**: Pair problems requiring two distinct elements use `while (l < r)`, whereas single-element boundary or in-place partition checks may need `l <= r`.

When does a variable-size sliding window fail for subarray-sum problems, and what replaces it?::When the array contains negative numbers (breaking monotonicity as the window expands/shrinks); use prefix sums + a hash map instead.

## What are the three main variants of the Two Pointers pattern? #card

1. **Converging (opposite ends)**: Start at `0` and `n - 1` and move inward (sorted pair sums, 3Sum, container with most water).
2. **Same-direction / Sliding Window**: Expand `right` and shrink `left` to maintain a contiguous subarray/substring invariant.
3. **Fast & Slow / Read-Write**: Different speeds for linked-list cycle detection/midpoint, or `read`/`write` indices for in-place array deduplication and partitioning.

## Related

- [[Binary search]]
- [[Algorithms TODO]]
- [[Data Structures]]
