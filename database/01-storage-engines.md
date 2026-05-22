# Storage engines: heap, LSM, columnar

**Summary** — three families of how a database lays bytes on disk. Each
makes a different trade between read latency, write amplification, and
scan throughput.
**Why I'm reading this** — to read a "we picked X engine" choice and
know what they gave up to get what they gained.
**Status** — draft.

## The three families

### Heap (PostgreSQL, MySQL InnoDB)

Rows live in pages. New rows append to a page with free space; updates
write a new row version in the same page (or a different page if it
doesn't fit). Indexes are separate structures pointing at row locations.

- **Reads** — random IO on the heap, fast through indexes.
- **Writes** — small for inserts; updates can be expensive if they push a
  row to a new page (HOT updates avoid this when no indexed column moves).
- **Scans** — sequential when pages are co-located, but row versioning
  (MVCC) means dead rows take up scan budget until VACUUM runs.

### LSM (RocksDB, LevelDB, Cassandra, ClickHouse merge tree)

Writes go to an in-memory **memtable** + a write-ahead log. When the
memtable fills, flush it to an immutable sorted file on disk
("SSTable"). Background **compaction** merges SSTables across levels.

- **Reads** — may have to check the memtable + every level until the key
  is found (Bloom filters help skip levels).
- **Writes** — append-only, fast and predictable.
- **Scans** — fast for sorted ranges; pays read amplification across
  levels.
- **The catch** — *write amplification* during compaction. Tuning the
  compaction strategy (size-tiered vs leveled) is most of the operator
  work.

### Columnar (ClickHouse columns, Parquet on disk, Hydra)

Each column stored separately. A row "exists" as a tuple of offsets across
column files.

- **Reads** — narrow projections (`SELECT a, b FROM big`) only read those
  columns. Compression ratios are dramatically better because column data
  is homogeneous.
- **Writes** — slow per-row (have to update many files); batch-friendly.
- **Scans** — dominant strength. Predicate pushdown + vectorized
  evaluation. This is why analytics workloads picked columnar.

## A useful axis: write amplification

```
Heap     ~1x         (one row write = one row written, maybe to WAL too)
LSM      3-10x       (memtable flush + compaction across levels)
Columnar high        (per-column update across many files)
```

Write amplification is what determines how much SSD endurance you burn
through per row of data ingested.

## When to pick what

| Workload                                 | Engine        |
|------------------------------------------|---------------|
| OLTP: small reads + small writes         | Heap          |
| Logging, time-series, K/V at scale       | LSM           |
| Analytics: scan-heavy, narrow projection | Columnar      |
| Hybrid: lookup + occasional analytics    | Heap + replica into columnar |

## What I got wrong the first time

Thinking LSM was "for write-heavy workloads." It's for **append-heavy**
workloads. An LSM that gets a lot of small updates (rewrites of existing
keys) suffers badly — the writes are fast, but compaction has to merge
those overlapping versions, which is exactly the work you avoid in a heap.

## References

- *Designing Data-Intensive Applications*, ch. 3
- *Architecture of a Database System*, Hellerstein et al.
- RocksDB wiki — compaction tuning is a small book in itself
