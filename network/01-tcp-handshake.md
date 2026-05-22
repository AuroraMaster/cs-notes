# TCP handshake & congestion windows

**Summary** — three-way handshake establishes sequence numbers and per-side
state; the congestion window controls how aggressive each side is allowed
to be while ramping up.
**Why I'm reading this** — to understand why slow-start hurts on short
connections and what BBR is actually fixing.
**Status** — draft.

## The handshake

```
   client                 server
     │ ─── SYN, seq=X ─────► │
     │ ◄── SYN-ACK,         ─│
     │     seq=Y, ack=X+1   │
     │ ─── ACK, ack=Y+1 ───► │
     │  data ─────────────► │
```

1. Each side picks a random initial sequence number (ISN). Modern stacks
   pick this carefully — predictable ISNs were a real attack vector in the
   1990s (Mitnick's spoof).
2. After the third packet (the final ACK), both sides have agreed on a
   starting offset and per-side window. No data has actually moved yet on
   *this* exchange.
3. **TCP Fast Open** allows data piggybacked on the SYN if the client has a
   cookie from a prior session. Wide adoption is slow but Linux + Chrome
   support it.

## Slow start

Each side starts with a small congestion window (cwnd), classically 10
segments. cwnd grows **exponentially** during slow start — one extra MSS per
ACK received — until it hits `ssthresh`, then transitions to additive
increase.

The cost: on a short connection (e.g. one HTTPS request), most of the
transfer happens *during* slow start. You never fully use the pipe.

Mitigations:

- **HTTP/2 multiplexing** so one connection handles many requests, keeping
  cwnd warm.
- **HTTP/3 / QUIC** which keeps state across network changes (NAT rebinds,
  mobile → wifi) so cwnd survives.
- **Initial window tuning** (`tcp_init_cwnd` sysctl on Linux) raised from 4
  to 10 segments in RFC 6928 (2013), more in some CDNs.

## What I got wrong the first time

Thinking the 3-way handshake "wastes" 1 RTT. It's actually setting up state
both sides need — sequence numbers, MSS, SACK options, timestamps — that
can't easily be batched into a single packet without TCP Fast Open.

## References

- RFC 9293 — TCP (consolidating spec, 2022)
- RFC 5681 — TCP congestion control
- RFC 6928 — Initial Window of 10 MSS
- RFC 7413 — TCP Fast Open
