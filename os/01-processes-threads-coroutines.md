# Processes, threads, coroutines

**Summary** — three units of scheduling that differ in *who creates them, who
schedules them, and what they cost to switch*.
**Why I'm reading this** — to stop confusing M:N runtimes (Go, Tokio) with
true threading and to be able to estimate when each model wins.
**Status** — draft.

## Layered picture

```
+---------------------------------------------------+
| user code: tasks / coroutines                     |   userspace, M:N runtime
+---------------------------------------------------+
| OS threads (1:1 to kernel threads)                |   created via clone(2)
+---------------------------------------------------+
| processes (separate address spaces)               |   created via fork(2) / clone(2) with CLONE_NEWNS etc.
+---------------------------------------------------+
| kernel scheduler (CFS / EEVDF)                    |
+---------------------------------------------------+
```

| Unit       | Created by        | Scheduled by       | Address space    | Context switch cost (approx, x86-64) |
|------------|-------------------|--------------------|------------------|--------------------------------------|
| Process    | `fork`/`clone`    | Kernel             | Separate         | ~2–10 µs (TLB flush)                 |
| Thread     | `clone(CLONE_VM)` | Kernel             | Shared           | ~1–3 µs                              |
| Coroutine  | User runtime      | User runtime       | Shared           | ~50–200 ns                           |

Numbers are order-of-magnitude — actual cost depends on cache state.

## Where the wins come from

- **Process** isolation buys safety (one crash doesn't take everyone) at the
  cost of expensive IPC.
- **Threads** share memory cheaply but force you to think about locks, the
  memory model, and lifetime of shared state.
- **Coroutines** sidestep the kernel for the switch itself, which makes
  millions of concurrent "tasks" feasible, *as long as none of them
  block the kernel thread underneath*.

## What I got wrong the first time

Confusing "Go has goroutines so it doesn't use threads" — Go's runtime
actually uses an M:N model with N OS threads under the hood, and a blocking
syscall in any goroutine will park the underlying thread (the runtime then
spawns another). Same idea applies to Tokio's blocking task pool.

## References

- *Operating Systems: Three Easy Pieces*, ch. 4 (processes), ch. 26 (threads)
- Linux `clone(2)` man page
- Go runtime: https://go.dev/src/runtime/proc.go (`schedinit`, `newm`)
