# CAP and PACELC

**Summary** — CAP says you can't have all three of consistency, availability
and partition tolerance during a partition. PACELC adds the otherwise-
ignored axis: *what happens during normal operation (no partition)*?
**Why I'm reading this** — to read distributed-DB marketing material
without taking it at face value.
**Status** — draft.

## CAP — what it actually says

Three properties:

- **C**onsistency — every read returns the latest write or an error.
- **A**vailability — every request gets a non-error response (no timeouts).
- **P**artition tolerance — the system keeps working through network
  partitions.

The theorem: during a partition, you can have at most two. Since real
networks partition (NICs flap, switches die, cables get tripped over), **P
is not optional in practice** — what you're really picking is C-vs-A
**during a partition**:

- **CP** systems return errors while partitioned (some replicas refuse
  writes to preserve consistency). Examples: HBase, ZooKeeper, etcd.
- **AP** systems keep accepting writes on each side and reconcile later
  (last-write-wins or CRDT). Examples: Cassandra, DynamoDB (most modes),
  Riak.

## What CAP leaves out

CAP only says anything during a partition. But systems spend almost
**all** their time *not* partitioned, and there's still a trade-off:

> Even when there's no partition, do you optimize for **latency** or
> **consistency**?

A strongly consistent system has to coordinate (Paxos round, quorum read)
on every operation, adding latency. A latency-optimized system can serve
from a local replica without that round-trip.

PACELC names this:

```
PA / EL    AP during partition, latency during normal      (Cassandra default)
PA / EC    AP during partition, consistent during normal   (rare)
PC / EL    CP during partition, latency during normal      (rare)
PC / EC    CP during partition, consistent during normal   (Spanner, BigTable)
```

So Cassandra and Spanner are not just "different points on CAP" — they're
**different points on PACELC**, and the EL/EC choice is what bites you in
daily life.

## A concrete framing

For your design discussion, replace "is this AP or CP?" with two questions:

1. During a partition, do you fail the write or accept it?
2. With no partition, do you read from any replica (faster, possibly stale)
   or from the leader / quorum (slower, current)?

These are the actually-actionable knobs.

## What I got wrong the first time

Believing "eventual consistency" was a clean named property. It's a family:
read-your-writes, monotonic reads, causal, session — each with different
guarantees. "Eventually consistent" alone doesn't tell you whether reading
your own write after writing it returns the new value.

## References

- Brewer, *Towards Robust Distributed Systems*, PODC 2000 (original CAP)
- Abadi, *Consistency Tradeoffs in Modern Distributed Database System Design*
- Kleppmann, *DDIA*, ch. 9
