# cs-notes

Personal study notes, mostly written while reading textbooks, papers and
source code. Each section is a folder of short markdown files — one topic per
file, kept small enough to re-read in a sitting.

The point is to **understand**, not to look exhaustive. Notes that go stale
get rewritten rather than padded.

## Topics

| Section | What's in it |
|---|---|
| [`os/`](./os) | Process model, memory, scheduling, syscalls, Linux internals |
| [`network/`](./network) | TCP/IP, congestion control, HTTP, TLS, modern QUIC/BBR |
| [`algorithms/`](./algorithms) | Classic algorithms, data structures, complexity |
| [`rust/`](./rust) | Ownership, lifetimes, async, unsafe, ecosystem deep-dives |
| [`harmonyos/`](./harmonyos) | ArkTS, ArkUI, Stage model, performance notes |
| [`distributed/`](./distributed) | Consensus, replication, CAP, time |
| [`database/`](./database) | Storage engines, query planning, PostgreSQL internals |
| [`security/`](./security) | Auth, crypto, common pitfalls, threat modeling |

## Conventions

- One file per topic. Filename = `nn-kebab-case.md` so they sort.
- Each file opens with a one-line summary and a "why I'm reading this" line.
- Code snippets are runnable when reasonable; otherwise marked `pseudo`.
- Sources are cited at the bottom under `## References`.

## Status

Seeded. See [`INDEX.md`](./INDEX.md) for the live table of contents as files
land.

## License

CC-BY-4.0 for the prose; code snippets are MIT unless otherwise noted.
