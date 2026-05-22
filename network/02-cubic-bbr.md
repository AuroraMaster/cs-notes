# Congestion control: Cubic, BBR, BBRv3

**Summary** — Cubic uses **loss** as the congestion signal; BBR uses
**bandwidth × RTT estimates**. The shift moves TCP off the assumption
"loss == congestion" — an assumption that hurts on wireless and on shallow
buffers.
**Why I'm reading this** — to know which `tcp_congestion_control` to pick
for what workload and what BBRv3 actually changed.
**Status** — draft.

## Cubic in one paragraph

Default on most Linux for a decade. Loss-based: cwnd grows aggressively
(cubic function of time since last loss), then halves on loss event. Works
well on wired networks with deep buffers. **Pathological on wireless**: a
single random loss halves the window even though the path is fine. Also
suffers on shallow-buffer links: it fills the buffer to find capacity
("buffer bloat") then crashes.

## BBR's model

Maintain two estimates over a sliding window:

- `BtlBw` — bottleneck bandwidth (max observed delivery rate)
- `RTprop` — minimum RTT observed

Send at exactly `BtlBw` with `cwnd = BtlBw * RTprop`. Probe occasionally:

```
PROBE_BW     8 cycles, each = 1 RTprop:
  cycle 0    pacing_gain = 1.25  → speculate more bandwidth exists
  cycle 1    pacing_gain = 0.75  → drain the queue you just made
  cycles 2-7 pacing_gain = 1.0   → steady state at BtlBw
```

In **PROBE_RTT** (every 10 s), cwnd drops to 4 packets for 200 ms so you
can re-measure RTprop without queueing.

The key property: BBR **doesn't fill the buffer to find capacity**. It
estimates capacity from the *delivery rate*, so it converges near the
bottleneck without inducing the bufferbloat that Cubic causes.

## What BBRv3 fixed (vs v1/v2)

- **Convergence with Cubic** on shared links. BBRv1 was unfair to Cubic
  (BBR's cwnd ignored loss; Cubic's halved on loss → Cubic loses every time).
- **Adaptive PROBE_BW gain**, less queue buildup in PROBE_BW.
- **Loss handling** — react somewhat to high loss rates instead of
  ignoring loss completely. This was the biggest pain point in v1/v2.

## When to pick what

| Workload                              | Best CC               |
|---------------------------------------|-----------------------|
| Wired, RTT < 50 ms, deep buffers       | Cubic                 |
| Wireless / lossy / mobile             | BBR (v2 or v3)        |
| CDN edge over high-bandwidth WAN      | BBR — measurably wins |
| Datacenter, low RTT, no loss          | DCTCP or Cubic        |

`sysctl net.ipv4.tcp_congestion_control=bbr` — but you also need `fq` or
`fq_codel` qdisc; BBR's pacing requires it.

## What I got wrong the first time

Believing BBR was "always better." On a path where the bottleneck is *very*
deep buffered (some satellite links, some old enterprise switches) BBR
actually underperforms Cubic, because Cubic happily floods the queue and
finds capacity that way. BBR is the right default but not universal.

## References

- Cardwell et al., *BBR: Congestion-Based Congestion Control*, CACM 2017
- Cardwell et al., *BBR v2: Better Loss Handling and Coexistence*, IETF 2019
- IETF draft-cardwell-iccrg-bbr-congestion-control-02 (BBRv3 sketch)
- `kernel.org/doc/Documentation/networking/tcp.rst`
