# GPU tuning measurements

CYMiner mines at the GPU's existing settings. MSI Afterburner is a development
measurement tool on this Windows host, not a runtime or build dependency. A
clock profile must be tested separately for each algorithm: KawPoW (RVN),
MeowPow (MEWC), Etchash (ETC), and Autolykos v2 (ERG). Results below are
bounded synthetic measurements on one RTX 3060 Ti, not promised pool rates.
No benchmark submits a share. ETC and MEWC check a sample GPU result against
their independent CPU references before timing; RVN's fused test checks
candidates against its verified base path. The ERG search benchmark compares
the serial and staged GPU paths.

## Stock settings before the MEWC kernel change, 2026-10-31

The card's configured power limit was 200 W (100%). During prior ETC live
hashing, memory ran at 6801 MHz. These tests did not change clocks or power.
The measurements have different workloads and must not be compared as coin
profitability or as live-pool hashrates.

| Coin | Benchmark workload | Stock measurement |
| --- | --- | ---: |
| ETC | Height 25,641,257; 20,971,520 nonces; full 4,439 MiB DAG | 41.81 MH/s GPU; 41.30 MH/s search call |
| RVN | Block 4,598,812; epoch 613; 2,097,152 nonces; fused, 256 threads; full DAG | 22.17 MH/s GPU median |
| MEWC | Height 2,128,502; 2,621,440 nonces; 6,144 MiB cached DAG | 0.0463 MH/s GPU; 0.0463 MH/s search call |
| ERG | Height 1,905,531; 32,768 nonces; zero cache; staged path | 37.72 ms GPU; 40.76 ms search call |

MEWC's current DAG exceeds the GPU's 8 GiB VRAM. The measured 6 GiB cache
leaves missing DAG items to be generated during search; its mix stage used
56,622.7 of 56,629.5 measured GPU milliseconds. Memory clock alone may not
address that dominant recomputation cost. ERG's full table was about 7,054
MiB for this height, but the baseline above used zero cache and does not
represent a full-table run.

## MEWC cache-miss kernel change

When a MEWC item misses the partial DAG cache, four adjacent lanes need
different words of the same 64-byte fragment. The earlier kernel ran the
entire 512-parent fragment calculation separately in each lane. The updated
kernel assigns four words to each lane, uses a warp shuffle to obtain the
parent selector, and exchanges the Keccak input and output through shared
memory. The independent CPU reference matches the new GPU result at four
tested heights, including two nonces in one warp at height zero.

On the same 6,144 MiB cache and five batches of 262,144 synthetic nonces at
height 2,128,502, the pre-change kernel measured 48,916 H/s and the updated
kernel measured 71,960 H/s (47% higher). The mix stage still dominates. This
is a local GPU-kernel measurement, not a live-pool rate; desktop GPU use can
change either figure. The clock offsets were +0 for both runs.
On the longer ten-batch workload, the earlier kernel measured 40,690 H/s and
the updated kernel measured 67,472 H/s. The samples were taken at different
times under variable desktop GPU use, so the paired five-batch comparison is
the clearer local measurement.
After a read-only pool check at height 2,132,798 (epoch 284), a separate
five-batch, 6,144 MiB cached synthetic run measured 70,052 H/s in GPU time.
This run was repeated after another GPU workload ended; the interrupted run
while both programs shared the GPU was discarded.

## ERG follow-up, 2026-10-31

After removing experimental full-cache-specialized and fixed-length BLAKE2b
paths that measured slower, the existing zero-cache benchmark was repeated at
height 1,905,531 with 32,768 nonces per search. The median staged search took
37.46 ms of GPU time (about 0.85 MH/s); the serial path took 39.40 ms. The
screen was active and Windows reported 23% GPU utilization, so this is a
contended synthetic measurement rather than a clean before/after result. It
does not establish an ERG speed fix. A full-table retest was deferred because
the 7,054 MiB table plus the default 261 MiB reserve did not fit in the 7,140
MiB then free; using the minimum reserve would leave only about 17 MiB of
headroom on this active desktop.

## Development trial: +500 MHz memory offset

With the MSI Afterburner memory offset at +500 MHz, core offset +0, and power
limit 100%, `nvidia-smi` observed a 7,301 MHz memory clock during ETC work,
roughly 1,995 MHz core clock, 147 W board power, and 51 C. It reported no
power or thermal slowdown flag. These are a single-card trial, not defaults
for users. The stock measurements above were made before Afterburner was
installed, so desktop state also changed. Windows GPU Engine counters showed
desktop composition and RustDesk using the GPU between trials.

| Coin | +500 MHz measurement on the same workload |
| --- | ---: |
| ETC | 32.62 MH/s GPU; 32.04 MH/s call after minimizing Afterburner |
| RVN | 19.70 MH/s GPU at 256 threads; 19.81 MH/s at 128 threads |
| MEWC | 0.0335 MH/s GPU; 0.0335 MH/s call |
| ERG | 43.77-45.43 ms GPU; 46.80-48.38 ms call, staged zero-cache path |

Every measured +500 result was below its earlier stock measurement. After
Reset returned both offsets to +0, a second pass in the same desktop session
gave the following results. RustDesk and Windows desktop composition still
used the GPU during this pass, so these are not clean idle-card baselines.

| Coin | +0 MHz after Reset | +500 MHz trial |
| --- | ---: | ---: |
| ETC | 30.41 MH/s GPU; 29.99 MH/s call | 32.62 MH/s GPU; 32.04 MH/s call |
| RVN | 18.50 MH/s GPU at 256 threads | 19.70 MH/s GPU at 256 threads |
| MEWC | 0.0407 MH/s GPU; 0.0407 MH/s call | 0.0335 MH/s GPU; 0.0335 MH/s call |
| ERG | 45.44 ms GPU; 47.96 ms call | 43.77-45.43 ms GPU; 46.80-48.38 ms call |

The later +0 and +500 passes suggest a possible ETC/RVN gain, no clear ERG
gain, and a MEWC loss, but GPU contention changed between runs. The earlier
idle-card ETC result was 41.81 MH/s, well above both later passes. A controlled
idle-GPU A/B/A trial would be needed before recommending a per-coin clock
profile. The tested +500 profile is not an CYMiner default, and Reset has
restored +0 offsets.

## Reproduction

Run one benchmark at a time on an otherwise idle GPU. Record the applied
core offset, memory offset, power limit, observed clocks, power, temperature,
GPU time, and complete call time for each profile. Return to the original
settings between profiles. Do not enable a profile at Windows startup during
testing.

```powershell
.\build\windows-vs\Release\arc_etchash_benchmark.exe 25641257 1048576 full-dag 20
.\build\windows-vs\Release\arc_kawpow_cuda_tests.exe --benchmark-fused 4598812
.\build\windows-vs\Release\arc_meowpow_benchmark.exe 2128502 6144 262144 10
.\build\windows-vs\Release\arc_ergo_search_benchmark.exe
```

The RVN full-DAG benchmark can take several minutes to prepare and uses most
of this card's VRAM. MEWC takes about a minute with the stated cache. Other
GPU models and memory vendors require separate measurements.