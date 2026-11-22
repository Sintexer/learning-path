---
flashcards:
  q-x2d7: { nid: 1795356730618, hash: pk6giq4m, sync: b36sst8b }
  q-zexp: { nid: 1795356730642, hash: v6qs4fyd, sync: s6kb5ign }
  q-8j38: { nid: 1795356730669, hash: 6ixearja, sync: 6sga9tiy }
  q-akfd: { nid: 1795356730701, hash: 4xwpj5ga, sync: ab4gx8xw }
---

**Binary search** finds a target or boundary in \(O(\log n)\) time by halving the search space on each step. It appears not only in sorted arrays, but also in trees ([[B-Tree Index]]), disk indexes ([[SSTable]]), debugging (`git bisect`), and math/optimization problems like finding the root of a monotonic function or "binary search on the answer."

Binary search requires only a ==monotonic predicate== (e.g., `False, False, ..., False, True, True, ...`), not necessarily a sorted array.
^q-x2d7

## Array Binary Search Implementation

- Use [[Two pointers|two pointers]] for `l` and `r`.
- **Exact match (closed interval `[l, r]`)**: Use `while (l <= r)`—`<=` ensures you check the last remaining element when `l` equals `r`. Update `l = mid + 1` or `r = mid - 1` based on comparing `arr[mid]` to the target.
- **Boundary / insertion point (first `true`, half-open or closed `[l, r]`)**: Use `while (l < r)` with `mid = l + (r - l) / 2`. If `pred(mid)` is true, keep `mid` (`r = mid`); otherwise discard it (`l = mid + 1`). When the loop ends, `l` equals `r` at the boundary.

Why should you compute `mid` as `l + (r - l) / 2` instead of `(l + r) / 2` in binary search?::To prevent signed integer overflow when `l + r` exceeds `Integer.MAX_VALUE` (in Java, `(l + r) >>> 1` also avoids overflow).
^q-zexp

> [!warning] Failure modes & trade-offs
> - **Integer overflow**: `(l + r) / 2` overflows 32-bit signed integers for large arrays (a famous historical bug in Java's `Arrays.binarySearch`). Always use `l + (r - l) / 2` or `(l + r) >>> 1`.
> - **Off-by-one infinite loops**: When finding the *last* `true` with `l = mid`, floor division (`l + (r - l) / 2`) stalls forever when `r` equals `l + 1` because `mid` equals `l`. Bias `mid` upward (`l + (r - l + 1) / 2`) whenever a branch sets `l = mid`.
> - **Small-N cache effects**: On tiny arrays (< ~32–64 elements), a simple linear scan often beats binary search in real wall-clock time due to CPU branch misprediction and hardware prefetching.

Fix your binary search interval invariant (`[l, r]` vs `[l, r)`) before writing the loop and prove the search space strictly shrinks on every branch. #card-reminder
^q-8j38

## Why can updating `l = mid` with floor-division `mid = l + (r - l) / 2` cause an infinite loop? #card

When the search narrows to two adjacent indices (`r` equals `l + 1`), floor division computes `mid` equal to `l`. If the branch updates `l = mid`, neither `l` nor `r` changes and `while (l < r)` loops forever.

**Fix**: Use ceiling division `mid = l + (r - l + 1) / 2` whenever a branch updates `l = mid`, or convert the problem to finding the *first* element matching the inverted predicate with `r = mid`.
^q-akfd

## Related

- [[Two pointers]]
- [[Algorithms TODO]]
- [[B-Tree Index]]
- [[SSTable]]
- [[Data Structures]]
