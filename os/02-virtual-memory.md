# Virtual memory & page tables

**Summary** — every process sees its own contiguous address space; the kernel
+ MMU translate per-page on the fly via a multi-level page table.
**Why I'm reading this** — to understand TLB misses, huge pages, and why
`mmap` is cheap until you touch the bytes.
**Status** — draft.

## The address-translation pipeline

```
virtual addr (48 bits, x86-64)
  ┌──── PML4 idx ─── 9 bits ────┐
  │                              ▼
  │      PML4 table  ──► PDPT  ──► PD  ──► PT  ──► physical frame (52 bits) + 12-bit offset
  │
  ▲ TLB lookup happens here; on miss the MMU walks the tables above.
```

x86-64 currently uses 4-level (48-bit) or 5-level (57-bit) page tables. Each
level is indexed by a 9-bit slice of the virtual address. The final 12 bits
index inside the 4 KiB page.

| Page size  | How it's reached                                   | TLB entry coverage |
|------------|----------------------------------------------------|--------------------|
| 4 KiB      | 4 levels of indirection                            | 4 KiB              |
| 2 MiB      | 3 levels (last level is a huge-page leaf)          | 2 MiB              |
| 1 GiB      | 2 levels (PDPT entry is a huge-page leaf)          | 1 GiB              |

Huge pages **reduce TLB pressure** — one entry covers 2 MiB / 1 GiB
instead of 4 KiB — at the cost of internal fragmentation and weaker
copy-on-write granularity.

## Why mmap is "cheap"

`mmap()` only sets up the VMA (virtual memory area) record in the kernel.
No page table entries are populated until the first access, which **page
faults** and lazily allocates a physical frame. This is why `mmap`-ing a
100 GB file takes microseconds — but reading through it materializes
pages as you go.

## What I got wrong the first time

Thinking the TLB was a per-process structure. It's per-CPU. Process switches
invalidate it (unless tagged with PCID/ASIDs, which modern x86 supports —
look up `CR4.PCIDE`).

## References

- *Operating Systems: Three Easy Pieces*, ch. 18–20
- Intel SDM Vol. 3A, ch. 4
- LWN: "Five-level page tables", https://lwn.net/Articles/717293/
