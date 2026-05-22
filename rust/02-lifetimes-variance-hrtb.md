# Lifetimes, variance, HRTB

**Summary** — lifetimes are *labels* the borrow checker uses to track how
long a reference is valid; variance describes how those labels survive
substitution; HRTBs let a function accept "any lifetime".
**Why I'm reading this** — to stop getting tripped by `for<'a>` and to read
trait bounds like `Fn(&'a T)` correctly.
**Status** — draft.

## Lifetimes: just labels

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str { ... }
```

`'a` is **a label**. The compiler unifies the lifetimes of `x` and `y` into a
single region that must outlive the call. `'a` is not "a duration" you pick —
the compiler chooses the smallest region that satisfies the constraints.

## Variance: how lifetimes substitute

For a type `T<'a>`, the relationship between `T<'long>` and `T<'short>`
when `'long: 'short`:

| Variance       | Means                                  | Examples                  |
|----------------|----------------------------------------|---------------------------|
| **Covariant**  | `T<'long>` is-a `T<'short>`            | `&'a T`, `Box<T>`, `Vec<T>` |
| **Contravariant** | `T<'short>` is-a `T<'long>`        | `fn(&'a T)` in arg position |
| **Invariant**  | Neither direction                      | `&'a mut T`, `Cell<T>`     |

`&mut T` is invariant because you can both read and write through it;
allowing covariance would let you stash a longer reference into a slot
expecting a shorter one. That's UB.

## HRTB (higher-ranked trait bound): `for<'a>`

A normal bound:

```rust
fn run<F: Fn(&str)>(f: F) { ... }   // sugar — what does &str's lifetime mean?
```

The desugared form makes it explicit:

```rust
fn run<F: for<'a> Fn(&'a str)>(f: F) { ... }
```

This says: `F` must implement `Fn(&str)` *for any choice of `'a`*. It's
"for all `'a`", quantified inside the bound. Without HRTB, the lifetime
would be tied to the call site, which is too strict for functions stored
in containers.

## What I got wrong the first time

Believing `'static` means "lives forever". It actually means "may live
forever if needed" — i.e. there is no shorter region the value is forced
into. `String::leak()` returns `&'static str` because the heap allocation
is intentionally never freed.

## References

- *Rustonomicon* — Subtyping & Variance
- RFC 0387 — Higher-ranked trait bounds
- *Rust for Rustaceans*, ch. 1–2
