# Index

Rolling table of contents. Items prefixed `[ ]` are planned, `[x]` are
written, `[~]` are drafts I haven't reviewed yet.

## os/

- [ ] 01 — Processes vs threads vs coroutines
- [ ] 02 — Virtual memory and page tables
- [ ] 03 — Linux scheduler: CFS to EEVDF
- [ ] 04 — Syscalls, vDSO, io_uring
- [ ] 05 — File descriptors, epoll, kqueue
- [ ] 06 — cgroups v2 and namespaces

## network/

- [ ] 01 — TCP handshake, congestion windows
- [ ] 02 — Congestion control: Cubic, BBR, BBRv3
- [ ] 03 — TLS 1.3 handshake walkthrough
- [ ] 04 — HTTP/2 multiplexing, HTTP/3 over QUIC
- [ ] 05 — DNS, ECH, encrypted DNS
- [ ] 06 — Linux networking stack: from NIC to socket

## algorithms/

- [ ] 01 — Sorting trade-offs (cache, stability, parallelism)
- [ ] 02 — Hash tables: open addressing vs chaining, robin hood
- [ ] 03 — Trees: BST, B-tree, LSM
- [ ] 04 — Graph: BFS/DFS, Dijkstra, A*
- [ ] 05 — String matching: KMP, Aho–Corasick, suffix automaton
- [ ] 06 — Probabilistic structures: Bloom, Count-Min, HLL

## rust/

- [ ] 01 — Ownership & borrowing model
- [ ] 02 — Lifetimes, variance, HRTB
- [ ] 03 — Trait objects and dynamic dispatch
- [ ] 04 — Async runtime internals (Tokio)
- [ ] 05 — Pinning and self-referential structs
- [ ] 06 — Unsafe Rust and the aliasing model

## harmonyos/

- [ ] 01 — Stage model overview
- [ ] 02 — ArkTS vs TypeScript: what changes
- [ ] 03 — ArkUI declarative model
- [ ] 04 — LazyForEach and @Reusable
- [ ] 05 — Distributed soft-bus
- [ ] 06 — Performance: cold start, frame drops

## distributed/

- [ ] 01 — CAP, PACELC
- [ ] 02 — Raft walk-through
- [ ] 03 — Paxos family
- [ ] 04 — Vector clocks vs hybrid logical clocks
- [ ] 05 — CRDTs
- [ ] 06 — Distributed transactions: 2PC, Saga, Percolator

## database/

- [ ] 01 — Storage: heap vs LSM vs columnar
- [ ] 02 — PostgreSQL MVCC
- [ ] 03 — Query planner basics
- [ ] 04 — Indexes: B-tree, GIN, GiST, BRIN
- [ ] 05 — Vector indexes: IVF, HNSW, DiskANN
- [ ] 06 — WAL and crash recovery

## security/

- [ ] 01 — Symmetric vs asymmetric, when to use what
- [ ] 02 — TLS / mTLS
- [ ] 03 — OAuth2 / OIDC flows
- [ ] 04 — Password hashing: bcrypt → argon2
- [ ] 05 — Common web attacks (XSS / CSRF / SSRF)
- [ ] 06 — Threat modeling: STRIDE in practice
