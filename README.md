<p align="center">
<img src="images/dlsu_logo.png" alt="De La Salle University Logo" width="150"/>
</p>

# Case Study Project #1 - README

### Cache Memory Machine: 8-Way BSA + LRU vs. 8-Way BSA + MRU

**Submitted by Group 7 [S01]:**

- GALICIA, Lance Krystofer A.
- KE, Xan Luo C.
- MOJICA, Maurienne Marie M.
- PARADO, Sky Hannah G.
- YAMSUAN, Rhian Claire V.

*August 4, 2026*

---

## About

This repository contains the Machine 9 submission for the CSARCH2 Simulation Project 
(3rd Term, AY 2025-2026): an 8-way block-set-associative (BSA) cache simulator that 
compares LRU and MRU replacement policies on the same access sequence. It is a 
web-based GUI application built with vanilla HTML, CSS, and JavaScript, requiring no 
build step or server.

**Deployment Link:** https://claireyams.github.io/CSARCH2-Case-Study-1/

**Video Walkthrough:** https://youtu.be/La1JZhkZk0o?si=0WEaUhh5fg33Ij3T

**Screenshots:** [View the full screenshot compilation](./CSARCH2_S01_Group7_Machine9_Screenshots.pdf) — or view individual screenshot images at `images/screenshots`

---

## Project files

```
CSArch_CaseStudy1/
|-- index.html            | landing page / usage guide
|-- simulator.html        | cache simulator interface
|-- about.html            | project credits and submission info
|-- style.css             | shared site styling
|-- simulator.js          | cache simulation logic and UI behavior
|-- README.md             | project documentation
|-- assets/
|-- images/
    |--screenshots/
```

No build step or server is needed. Clone the repo and open `index.html`
(or `simulator.html` directly) in a browser.

## Specifications and parameters

| Parameter | Value |
|---|---|
| Cache mapping | 8-way block-set-associative, sets = cache blocks / 8 |
| Set mapping | set = block address mod number of sets |
| Replacement policy | parameter: LRU or MRU, applied per set |
| Block size | parameter: power of 2, minimum 2 words (default 16) |
| Number of cache blocks | parameter: power of 2, minimum 8* (default 16, i.e. 2 sets x 8 ways) |
| Main memory size | fixed, 1024 blocks |
| Read policy | parameter: non-load-through or load-through |
| Cache access time | 1 ns |
| Main memory access time | 10 ns per word |

\* The general spec allows a minimum of 4 blocks, but an 8-way set needs 8 blocks,
so this machine enforces a minimum of 8.

Timing per access:

- Hit: `1 ns` (one cache access)
- Miss, non-load-through: `1 + 10*B + 1 ns` — check the cache, load the block
  (B words) from memory into the cache, then read the word from the cache
- Miss, load-through: `1 + (10 + 10*B) / 2 ns` — the requested word could be the first
  or the last word to arrive from memory depending on its position in the block, so
  the CPU's expected wait is the average of the best case (10 ns, word arrives first)
  and worst case (10*B ns, word arrives last), on top of the cache check


AMAT = total access time / number of accesses.

## Outputs produced by the simulator

- Visual snapshot of the cache (every set and way, with recency bars and MRU/LRU markers)
- Toggle between a step-by-step animated trace and the final snapshot only
- Full text trace log (always generated, downloadable as `cache-trace-log.txt`)
- Statistics: access count, hit count, miss count, hit rate, miss rate, AMAT, total access time
- A comparison table that re-runs the same sequence under both LRU and MRU

---

## Sample test cases and expected output

All samples below use the default configuration:
**n = 16 cache blocks (2 sets x 8 ways), block size = 16 words.**
With 2 sets, even blocks map to set 0 and odd blocks map to set 1, so each set acts
like an 8-entry buffer for its half of the stream.

Miss penalties at this block size: non-load-through = 162 ns, load-through = 86 ns.
Hits cost 1 ns.

### Test case a — Sequential

What to do: select test case `a`, run once with LRU, once with MRU.

Generated sequence (2n blocks, repeated twice, 64 accesses):

```
0 1 2 ... 31   0 1 2 ... 31
```

Expected output:

| Policy | Read policy | Accesses | Hits | Misses | Hit rate | Miss rate | AMAT | Total time |
|---|---|---|---|---|---|---|---|---|
| LRU | non-load-through | 64 | 0 | 64 | 0.00% | 100.00% | 162.00 ns | 10,368 ns |
| LRU | load-through | 64 | 0 | 64 | 0.00% | 100.00% | 86.00 ns | 5,504 ns |
| MRU | non-load-through | 64 | 16 | 48 | 25.00% | 75.00% | 121.75 ns | 7,792 ns |
| MRU | load-through | 64 | 16 | 48 | 25.00% | 75.00% | 64.75 ns | 4,144 ns |

What you should see in the trace: under LRU, every access is a miss. Under MRU,
the first pass is all misses, but on the second pass the early blocks of each set
(0–13) hit, because MRU kept them in place.

### Test case b — Mid-repeat

What to do: select test case `b`, run once with LRU, once with MRU.

Generated sequence (160 accesses): `0..15`, then `0..31` twice, then the reverse:
`15..0`, then `31..0` twice.

```
0..15   0..31   0..31   15..0   31..0   31..0
```

Expected output:

| Policy | Read policy | Accesses | Hits | Misses | Hit rate | Miss rate | AMAT | Total time |
|---|---|---|---|---|---|---|---|---|
| LRU | non-load-through | 160 | 16 | 144 | 10.00% | 90.00% | 145.90 ns | 23,344 ns |
| LRU | load-through | 160 | 16 | 144 | 10.00% | 90.00% | 10.90 ns | 12,400 ns |
| MRU | non-load-through | 160 | 74 | 86 | 46.25% | 53.75% | 87.54 ns | 14,006 ns |
| MRU | load-through | 160 | 74 | 86 | 46.25% | 53.75% | 6.91 ns | 7,470 ns |

What you should see in the trace: LRU's 16 hits all happen right after the direction
reverses (the most recently used blocks are touched again immediately). MRU hits
repeatedly on the low-numbered blocks it kept resident through the whole run.

### Test case c — Random

What to do: select test case `c`, set the seed to `42` (or any value — the seed makes
a run reproducible), run once with LRU, once with MRU.

Generated sequence: 64 random block addresses in the range 0–1023.

Expected output with seed 42:

| Policy | Read policy | Accesses | Hits | Misses | Hit rate | Miss rate | AMAT | Total time |
|---|---|---|---|---|---|---|---|---|
| LRU | non-load-through | 64 | 0 | 64 | 0.00% | 100.00% | 162.00 ns | 10,368 ns |
| LRU | load-through | 64 | 0 | 64 | 0.00% | 100.00% | 86.00 ns | 5,504 ns |
| MRU | non-load-through | 64 | 1 | 63 | 1.56% | 98.44% | 159.48 ns | 10,207 ns |
| MRU | load-through | 64 | 1 | 63 | 1.56% | 98.44% | 84.67 ns | 5,419 ns |

Exact numbers depend on the seed, but both policies should land near 0% hits.
The cache only covers 16 of the 1024 memory blocks (about 1.6%), so with 64
uniformly random accesses a repeat within the cache's lifetime is unlikely.

---

## Analysis

### Test case a — Sequential

This trace is the worst case for LRU. Each set receives 16 distinct blocks per pass
but can only hold 8. By the time the sequence wraps around, LRU has always just
evicted the block that is needed next, so every single access misses and the cache
contributes nothing. MRU does the opposite: on a conflict it throws away the block
that was just used, which in a long loop is the one that won't be needed for the
longest time. This effectively pins the first blocks of each set, and they all hit
on the second pass. MRU ends with a 25% hit rate and roughly 25% less total access
time than LRU.

### Test case b — Mid-repeat

This trace mixes a small inner loop (0..n-1) with a wider loop (0..2n-1), plus a
reversed copy of both. For LRU the wide loop behaves the same as test case a —
constant eviction of the next needed block — so its only hits (16) come from the
turn-around points, where recently used blocks are re-touched immediately. That is
the one kind of locality LRU is built for. MRU again keeps a stable subset of the
loop resident, so those blocks hit in the forward passes and again in the reverse
passes. Its hit rate (46.25%) is more than four times LRU's, and total access time
drops by about 40%.

### Test case c — Random

With no locality in the address stream, the replacement policy stops mattering.
Almost every access is a compulsory miss for both policies, and any small
difference between LRU and MRU changes with the seed. Performance here is decided
entirely by the miss penalty, not by replacement.

### Non-load-through vs load-through

The read policy never changes which accesses hit or miss; it only changes what a
miss costs. Under non-load-through the CPU waits for the entire block and then reads
the word back out of the cache (162 ns at B = 16), while under load-through the CPU
only waits for its one requested word — but since that word could be anywhere in the
block, the expected wait is the average of the best case (10 ns, word arrives first)
and worst case (10 x B ns, word arrives last): 1 + (10 + 10 x 16)/2 = 86 ns at B = 16,
so each miss is 76 ns cheaper. The total effect is proportional to the miss count: in
test case b it saves 144 x 76 = 10,944 ns under LRU (23,344 -> 12,400 ns) but
86 x 76 = 6,536 ns under MRU (14,006 -> 7,470 ns), so it helps most exactly where the
replacement policy is doing worst. Note that unlike non-load-through, the
load-through penalty still scales with block size — just more gently, since it grows
with the average position of the word in the block instead of the full transfer.

### Conclusion for Machine 9

On these three required traces, MRU comes out ahead overall, because two of the
three are looping patterns whose working set is larger than one set's capacity —
the specific pattern where LRU degrades to (near) 0% hits while MRU retains part of
the loop. This is a property of the chosen test traces rather than a general rule:
most real programs reuse recently touched data, which is exactly what LRU protects
and MRU destroys, so LRU is the safer default in practice. MRU is mainly useful for
known large sequential scans (a common example is database scan workloads). The
random case shows that when there is no locality at all, neither policy helps, and
the only lever left is the miss penalty — which is where load-through matters most,
cutting the cost of every miss from 162 ns to 12 ns.

---

## Declaration of AI Usage

> In the development of this project, the group used AI as a supporting tool throughout the design and development process. AI assistance was primarily used for debugging layout, rendering, and state-management issues; drafting and refining the simulator and animation logic; writing explanatory inline code comments; improving UI spacing, contrast, and responsive behavior; and reviewing the clarity, grammar, and terminology of the written analysis.

>All AI-assisted code and content were manually reviewed, tested locally, and verified against the machine specifications before being committed to the project. AI tools were treated as a guide for troubleshooting and refinement, not as a substitute for the group's own understanding, and final implementation decisions, simulator behavior, and written analysis were confirmed and adjusted by each member individually.
