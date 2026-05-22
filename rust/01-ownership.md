# Ownership & borrowing — the smallest possible model

**Summary** — every value has exactly one owner; references are *temporary
loans* that must not outlive the owner and must respect aliasing rules.
**Why I'm reading this** — to stop pattern-matching `&` vs `&mut` and
actually reason about what the borrow checker is enforcing.
**Status** — draft.

## The three rules

1. **Each value has one owner.** When the owner goes out of scope the value
   is dropped.
2. **Many readers OR one writer, never both at the same time.** Either
   any number of `&T` or exactly one `&mut T`, never a mix, never
   overlapping.
3. **A reference may not outlive the value it points to.** Lifetimes are
   the compiler's bookkeeping for this.

That's the entire model. Everything else (`Box`, `Rc`, `RefCell`, `Cell`,
`Arc`, `Mutex`, ...) is a tool for *bending* one of these rules in a
specific, audited way.

## Mental tool: "who has the only mutable handle right now?"

For any piece of state, walk the call path and ask:

- Is anyone *reading* this right now? (immutable borrows alive)
- Does *exactly one* function hold the mutable handle? (a `&mut` or owned
  value)

If both can answer "yes" simultaneously, the borrow checker will refuse the
program. The fix is one of:

| Symptom                                | Likely fix                            |
|----------------------------------------|---------------------------------------|
| Want shared *read* across owners       | `Rc<T>` (single-threaded) / `Arc<T>`  |
| Want interior mutability               | `Cell<T>` / `RefCell<T>` / `Mutex<T>` |
| Want to split a struct into pieces     | Destructure, then borrow each field   |
| Want to mutate while iterating         | Collect indices first, mutate after   |

## What I got wrong the first time

Reaching for `Arc<Mutex<T>>` whenever I saw a borrow error. Most of the time
the real fix is a smaller scope or splitting the struct — `Arc<Mutex>` is
the answer to a *concurrency* question, not a borrow question.

## References

- *The Rust Programming Language*, ch. 4
- *Rust for Rustaceans*, ch. 1
- RFC 2025 (Stacked Borrows) for the formal aliasing model
