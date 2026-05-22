# Sorting trade-offs (cache, stability, parallelism)

**Summary** — there is no single "best" sort. The relevant axes are
*comparison vs distribution*, *cache behavior*, *stability*, and *whether
the work parallelizes*. Real-world libraries pick a hybrid based on
which axis matters most.
**Why I'm reading this** — to stop reflexively reaching for `Vec::sort`
when I should be reaching for `sort_unstable` or a radix sort.
**Status** — draft.

## Algorithm cheat sheet

| Algorithm     | Time          | Stable | In-place | Best at                                  |
|---------------|---------------|--------|----------|-------------------------------------------|
| Quicksort     | O(n log n) avg | no    | yes      | general comparison, cache-friendly        |
| Mergesort     | O(n log n)    | yes    | no       | stable, external sort, linked lists       |
| Heapsort      | O(n log n)    | no     | yes      | worst-case guarantees                     |
| Insertion     | O(n²) / O(n)  | yes    | yes      | small n, almost-sorted input              |
| Radix (LSD)   | O(n·k)        | yes    | sometimes| integer keys, large n                     |
| Radix (MSD)   | O(n·k)        | yes    | sometimes| strings, when sorting in-place matters    |
| Timsort       | O(n log n)    | yes    | no       | real-world arrays (mix of sorted runs)    |
| Pdqsort       | O(n log n)    | no     | yes      | pattern-defeating, default in Rust std    |

## What library defaults actually use

- **Python** `list.sort` → Timsort. Stable, exploits "natural runs."
- **Rust** `slice::sort` → Timsort (stable). `slice::sort_unstable` → Pdqsort.
- **C++** `std::sort` → introsort (quicksort + heapsort fallback). Not stable.
  `std::stable_sort` → mergesort-flavored.
- **Java** `Arrays.sort` → Timsort for objects, Dual-Pivot Quicksort for primitives.

The pattern: **library default = "I don't know what the user has, be safe and
adaptive"**. That's why Timsort wins as a default for generic data.

## Why cache behavior dominates wall-clock time

Big-O hides constants. On modern hardware, an L1 hit is ~4 cycles, L3 is
~40, main memory is ~200+. A sort with O(n log n) comparisons but bad
cache behavior loses to one with the same big-O and better locality.

This is why **quicksort beats mergesort in practice** despite having a
worse worst case: partition step is sequential and cache-friendly, while
mergesort allocates a scratch buffer and writes to it.

Pdqsort goes further: it uses block-style partitioning that avoids
branch mispredicts on the pivot comparison. Branchless partitioning was
the win in Rust's 1.x → 1.78 change to pdqsort.

## When to abandon comparison sorts

For 64-bit integers with n in the millions: a **radix sort** beats
comparison-based sorts by 2–5×. Same story for strings up to ~16 chars.
The reason is comparison sorts are stuck at `Ω(n log n)` — radix isn't.

But: radix sort needs to know the bit width and a working buffer. Library
APIs don't expose enough information to pick it automatically, so you do
it yourself.

## What I got wrong the first time

Believing "stable" meant "deterministic order." It doesn't. Stable means
**equal-key elements keep their relative input order**. A stable sort is
deterministic if you also tie-break, but un-stability isn't randomness —
it's "we promise nothing about equal keys".

## References

- *Engineering a Sort Function*, Bentley & McIlroy (still essential)
- Pdqsort paper: https://arxiv.org/abs/2106.05123
- Tim Peters' original `listsort.txt` (Timsort)
