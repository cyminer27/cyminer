# CYMiner technical notes

For v1.0.4 setup and coin availability, see [README.md](README.md). The historical source and test examples below use the earlier Arcminer executable and build option names.

Native Windows-focused C++20 / CUDA GPU cryptocurrency miner project.

Foreground mining reads the NVIDIA GPU core temperature before each batch.
The dashboard marks 80–85 °C red and Arcminer inserts progressively longer
pauses between GPU batches (200 ms at 80 °C, up to 1,000 ms at 84–85 °C).
Above 85 °C, Arcminer stops hashing,
checkpoints measured active GPU time, and closes its pool sessions. Mining
requires an available temperature sensor; an unreadable sensor stops the run.
This is a monitoring and stop limit, not automatic overclocking or a guarantee
that temperature cannot briefly exceed 85 °C during a GPU batch.
The pauses reduce Arcminer's GPU duty cycle; they do not change clocks, power
limits, or other programs' GPU use. Paused time is not counted as active GPU
hashing time for the developer fee.

Optional NVIDIA clock offsets can be requested without MSI Afterburner:
`--core-offset-mhz N` and `--memory-offset-mhz N` on an explicit `--mine`
command. They default to unchanged. Arcminer checks the driver's reported
range, applies the requested offsets through NVML, and attempts to restore the
previous values on normal exit. An unsupported setting or missing privilege
stops mining before connecting to pools. A process crash or power loss may
leave offsets in place until they are reset manually or by a driver restart.
These options do not select a Kryptex profile or automatically tune clocks.
An optional `--power-limit-w W` sets an explicit NVIDIA board power cap after
checking the card's reported range. It requires driver support and administrator
rights. The original cap is restored on normal exit when the driver permits;
a crash can leave the requested cap active. Arcminer changes the power limit
only when `--power-limit-w` or `--auto-power-limit` is supplied. The tested
RTX 3060 Ti reports a 200 W current cap and a 100–220 W allowed range. An
unelevated diagnostic write was rejected with insufficient permissions; an
elevated power change and restoration remain untested.
`--auto-power-limit` is a separate opt-in control. It verifies write permission
before pool connection, starts from the current cap (or the explicit
`--power-limit-w` cap), lowers it by up to 10 W at 80–85 °C, and raises it by
up to 5 W after cooling to 75 °C. Changes are at least five seconds apart and
stay within the driver's range and the initial cap. Batch pacing and the
above-85 °C stop still apply. This feature requires administrator rights and
has not been tested with a hardware power-limit write on the RTX 3060 Ti.
For this card, use `--auto-power-limit` without `--power-limit-w` to keep the
existing 200 W ceiling. A `--power-limit-w 150` option instead caps it at
150 W. The power cap is an upper bound; actual draw depends on the GPU workload.
On the tested RTX 3060 Ti and installed Windows driver, both NVML offset
queries returned `NVML_ERROR_NOT_SUPPORTED`; Arcminer therefore refuses an
offset request on this card before mining starts. The thermal stop still works
with the card's existing clock settings.
Run `arcminer --gpu-tuning-status [--devices INDEX]` for a read-only report of
core temperature, clock offset support, and current and allowed power limits. It does
not start mining or change GPU settings.
To check write permission and restoration without a pool, run
`arcminer --power-limit-check [--devices INDEX]` in an Administrator PowerShell.
The command briefly lowers the current power cap by up to 10 W within the
driver's allowed range, reads it back, restores the original cap immediately,
and verifies the final readback. It does not hash or contact a pool. If it
reports a failure, run `arcminer --gpu-tuning-status` to inspect the current
cap; a driver or process failure can prevent automatic restoration.

Arcminer is licensed under [GNU GPL version 3](LICENSE). Windows releases
include the complete project source, build scripts, tests, and third-party
license notices. Source copied from kawpowminer is documented in
[docs/THIRD_PARTY.md](docs/THIRD_PARTY.md). Its fused CUDA search kernel is
used by default on the tested RTX 3060 Ti (SM 8.6) and falls back to the
original Arcminer path if unavailable. Set `ARCMINER_FUSED=0` in the process
environment to disable it, or `ARCMINER_FUSED=1` to test it on another GPU.
Candidate hashes are independently checked on the CPU before submission.

## Algorithms

`arcminer --help` now presents the coin and algorithm choices in an option menu.
It separates currently accepted CLI mining routes from planned algorithms and
labels XNA as unverified on a live pool. The menu documents only options that
the CLI parses; Rigel's dual/triple, TLS, and failover syntax is not available.
Follow [the CLI help layout guide](docs/CLI_HELP_LAYOUT.md) when adding a route
or option so the menu, parser, status table, and tests stay aligned.

**Available in Arcminer today: KawPoW for Ravencoin (RVN), experimental
MeowPow for Meowcoin (MEWC), experimental Etchash for Ethereum Classic
(ETC) mainnet, and Autolykos v2 for Ergo (ERG).**
Measured GPU tuning results for these four routes are in
[docs/GPU_TUNING.md](docs/GPU_TUNING.md). Arcminer does not require MSI
Afterburner to build or mine.
MeowPow CUDA mining for Meowcoin (MEWC) is experimental. Its CPU verifier
matches three Meowcoin Core hashes, the CUDA worker matches those vectors,
and mock pool tests cover share submission and the pool's target updates.
An independently CPU-verified MEWC user share was accepted by Cedric's live
pool at height 2128544 on 2026-10-27. The 2Miners ETC account API reported
three valid shares for Arcminer's `arcminer-test` worker on 2026-10-28, with
zero stale or invalid shares. The remaining algorithms are planned and their
names are not accepted by the current `--algo` option.
Coin tickers in the table are examples from
[Rigel's supported-algorithms list](https://github.com/rigelminer/rigel#supported-algorithms),
not claims that Arcminer supports those coins or their pool protocols. Each
coin/network needs its own implementation and verification before release.

| Algorithm | Coin examples in Rigel | Arcminer status |
| --- | --- | --- |
| `abelian` | ABEL | Pinned CPU and experimental CUDA reference vectors pass; no pool protocol or mining route |
| `alephium` | ALPH | Experimental Windows CUDA mining; CPU-verified mainnet share accepted |
| `autolykos2` | ERG | CPU/GPU reference hits and a live FenixPool SOLO share accepted; Windows CLI mining enabled |
| `etchash` | ETC | Experimental CUDA mining; CPU-verified mainnet shares accepted |
| `ethash` | ETHW, XPB, OCTA | Planned |
| `fishhash` | IRON | Planned |
| `karlsenhashv2` | KLS | Planned |
| `kawpow` | RVN, XNA, QUAI, NEOX | **RVN mainnet available**; other coins planned |
| `meowpow` | MEWC | Experimental CUDA mining; CPU-verified mainnet share accepted |
| `nexapow` | NEXA | Planned |
| `octopus` | CFX | Planned |
| `progpowz` | ZANO | Planned |
| `sha256ton` | GRAM | Planned |
| `sha3x` | XTM | Planned |
| `sha512256d` | RXD | CUDA batch runner routes CPU-verified shares through separate user/developer sessions; dual mock-pool test passes; real-pool acceptance is pending and CLI mining remains disabled |
| `xelishashv3` | XEL | Planned |
| `zil` | ZIL | Legacy Rigel entry; Zilliqa 2.0 mainnet is proof of stake |

ZelHash was evaluated and excluded: Flux mainnet activated Proof of Nodes at
block 2,020,000, and the current official Flux daemon says traditional PoW
mining is no longer used. Alephium mining is experimental. Its Windows CUDA
path CPU-verifies candidates and uses separate user/developer sessions with
persisted 1% active-time fee routing. The first bounded live run at Cédric's
GPU endpoint found a CPU-verified user share that the pool accepted; no
developer share has yet been observed. No release dates are set for planned
algorithms.

The Autolykos v2 work validates Ergo's height-dependent table size and a
published CPU PoW hit, checks the supplied user and developer payout addresses,
parses live 2Miners jobs and subscription nonce spaces, and CPU-gates a
mock-tested share request through separate account-bound sessions. A bounded
CUDA diagnostic matches the published Ergo hit, searches short nonce ranges,
and CPU-verifies returned candidates. It also builds bounded GPU table chunks
that match the CPU reference at tested height and index boundaries. A bounded
GPU table window can be grown between batches in interruptible chunks and
used for diagnostic hashes. Its capacity follows an explicit byte budget and
leaves a 256 MiB GPU memory reserve. It has no
full-height dataset or measured mining throughput. Each cache entry now stores
the 31 bytes used by Autolykos v2; a full cache at
height 1,905,531 would require 7,054.37 MiB before the memory reserve.
A bounded GPU-filtered search CPU-verifies candidates, and a job-bound nonce cursor reserves distinct
suffixes across consecutive batches. The batch runner measures CUDA kernel
time and routes diagnostic work through the persisted 1% active-time fee
ledger. Two local mock pool sessions accepted an independently CPU-verified
ERG GPU candidate on only the selected account route. The bounded diagnostic
loop also submits shares through each route to local mock pools, polls for
new jobs between GPU batches, reconnects a dropped pool session with new work
and nonce space, and persists measured active time. A foreground diagnostic
API can run until its caller requests stop and checks that request between
GPU batches and cache chunks. It prebuilds its memory-budgeted cache window
before hashing by default; a local benchmark can measure bounded build chunks.
An easy-target overflow is retried in smaller GPU chunks; local mock pools
accepted all 65 CPU-verified shares on either fee route.
The `arc_ergo_live_work_check WALLET WORKER [NONCES<=32768]` diagnostic connects to
`erg.2miners.com:8888`, compares one GPU hit with the independent CPU result
on a current job, and searches a bounded range without submitting shares.
The default is 64 nonces. Both supplied payout accounts passed a 1,024-nonce
read-only check on 2026-10-29. The GPU search, nonce cursor, and mock-tested
fee route now accept up to 32,768 nonces per batch. A no-match maximum-size
batch is covered by CUDA tests.
GPU candidates carry the exact nonce bytes used by the kernel; the host checks
those bytes against the reserved pool nonce before CPU-verifying a share.
The ERG search now uses three bounded CUDA stages: derive indices, calculate
the 32 table elements per nonce in parallel, then test the final hit. The
serial kernel remains available for diagnostic comparison. Returned
candidates still require independent CPU verification.
`arc_ergo_search_benchmark [HEIGHT [CACHE_MIB<=8192 [RESERVE_MIB=256..4096 [NONCES<=32768]]]]` compares bounded cache
sizes using synthetic work and reports GPU kernel and full call times without submitting
shares. A custom cache run measures zero-cache searches before and after it.
A requested large cache keeps a 1 GiB free-memory reserve by default; an
explicit reserve can be set down to 256 MiB. It can
take minutes to build. A
replacement cache now accounts for the old window's reclaimable VRAM when
sizing its new allocation.
`arc_ergo_full_cache_check HEIGHT [RESERVE_MIB] [--preflight-only]` is a separate read-only
full-table check. It defaults to a 256 MiB reserve; an explicit reserve may be
lowered to 64 MiB for a controlled diagnostic on an otherwise idle GPU. It
reserves an additional 5 MiB for its temporary search buffers and refuses to
allocate unless the complete 31-byte table fits with both reserves accounted
for. It stops between build chunks on Ctrl+C and compares a GPU hit with the
independent CPU result before seven 4,096-nonce synthetic ranges. Use
`--preflight-only` to check available VRAM without allocating the table. The
diagnostic reports the median search time and index, element, and finalize
stage times. These measurements do not submit shares and are not pool
hashrate measurements. No ERG speed fix has been established.
It is not an
efficient production search. See
[protocol notes](docs/AUTOLYKOS2_PROTOCOL.md).
The main CLI accepts `--check-config --coin erg`, a read-only
`--probe-pool --coin erg`, and explicit Windows CUDA mining with
`--mine --coin erg --pool HOST:PORT --wallet ADDRESS`. The mining path uses
the same CPU-verified candidate submission and persisted fee ledger as the
accepted FenixPool diagnostic. The dashboard clears before its first frame
and redraws its solid border when the visible console size changes. On Windows,
it writes Unicode directly to the console so border glyphs do not depend on
the active code page.
A short foreground main-CLI smoke run against FenixPool SOLO accepted two
additional CPU-verified user shares with zero rejected shares; Ctrl+C
stopped after the current batch. A SOLO share is not a block payout.
An explicit, time-limited `arc_ergo_live_share_test USER_WALLET WORKER SECONDS
[DEVICE] [--full-cache] [--fenix-solo]` executable is available for mainnet share tests. It uses
separate user and developer sessions, submits only independently
CPU-verified candidates, and stores its 1% active-GPU-time fee state in
`LocalAppData\ArcMiner\autolykos2-ergo-mainnet.fee-state`.
The default user and developer endpoints are `erg.2miners.com:8888`. The
optional `--fenix-solo` routes only the user session to `fenixpool.com:9052`;
the developer session stays on 2Miners. FenixPool SOLO payouts require a block.
A two-minute
live run on 2026-10-30 completed 7,606 batches at the former 1,024-nonce cap.
A second two-minute run with the 8,192-nonce cap completed 5,105 batches and
approximately 41.8 million nonces, over five times the searched work. Both
runs found no shares. A later FenixPool run accepted a real share.
A third two-minute run at the 32,768-nonce cap searched exactly 59,210,926
nonces in 1,815 batches with no share found. Its final batch took about 65 ms
wall time, and measured GPU kernel time was 93.167 seconds across the run.
It starts at 64 nonces per batch and adjusts later batches toward 250 ms of
observed batch time, capped at 32,768. Fee accounting still uses measured GPU
time. A GPU launch already in progress cannot
be interrupted by the tuner.
The default cache is 8 MiB. `--full-cache` explicitly requests the complete
height-dependent table and refuses before allocation unless it fits with
256 MiB left free. The time limit includes cache preparation.
The diagnostic prints cache progress, searched nonces, accepted/rejected share
counts, and the next batch size during a run. Its final summary separates GPU
kernel, search-call, and durable fee-state write time. On 2026-10-30,
a ten-minute FenixPool SOLO run searched 307,625,920 nonces and the pool
accepted one CPU-verified user share with zero rejected shares. The developer
session had zero accepted or rejected shares during that interval.

The Etchash implementation uses a separate ETC mainnet CPU verifier. It
checks the ECIP-1099 transition at block 11,700,000: dataset sizing switches
to 60,000-block epochs, and each epoch uses a seed reached by two steps on
the original 30,000-block seed chain.
The verifier checks complete mix and target values against a pinned Apache-2.0
reference, a published Ethash vector, and four ETC mainnet block
vectors at heights 11,699,999, 11,700,000, 11,730,000, and 11,760,000. The ETC
reference's exported symbols are isolated from KawPoW and both CPU verifiers
pass together in one process. A standalone ETC address checker validates
`0x` plus 40 hexadecimal digits, ERC-55 mixed-case checksums, and rejects the
zero payout address. Hex addresses do not identify a chain or prove wallet
ownership. A bounded parser validates three- and four-field ETC eth-proxy
work (including a live 2Miners block height) and its post-fork seed. The
ETC developer account is disclosed in [protocol notes](docs/ETCHASH_PROTOCOL.md).
Use `--check-config --coin etc --pool HOST:PORT --wallet YOUR_ETC_ADDRESS`
for offline validation, or replace `--check-config` with `--probe-pool` for a
read-only eth-proxy login and work check. A mock pool covers login success,
rejection, and invalid seed. `--mine --coin etc` starts explicit foreground
Etchash hashing with a separate user wallet and the disclosed developer session.
A persistent per-account Etchash session now handles job replacement,
reconnection, and CPU-verified share submission in mock pool tests. The
read-only probe uses this session. A CUDA worker matches
the CPU mix and final hashes at ETC heights 11,700,000, 11,760,000, and
25,622,670 on the tested GPU. It generates DAG items from the light cache on
demand for correctness tests. The mining path builds a full GPU DAG and searches
up to 1,048,576 consecutive nonces per batch, filtering the complete target on
the GPU. Tests compare full-DAG candidates, target filtering, and overflow
handling with the independent CPU reference. The pool session verifies each
candidate again on the CPU before submission. A two-minute RTX 3060 Ti run
measured about 13.3 MH/s wall-clock and 15.4 MH/s of GPU kernel time; these are
observations from the original kernel, not promises for other GPUs. The tested
SM 8.6 card now uses a three-stage full-DAG search by default; set
`ARCMINER_ETCHASH_SPLIT=0` to use the original path or `=1` to test the new
path on another GPU. A bounded synthetic run on the RTX 3060 Ti at ETC height
25,622,670 measured 41.8 MH/s in GPU kernels and 40.9 MH/s per search call
over ten 1,048,576-nonce batches. The split path matched the independent CPU
reference at the fork and a recent height, including target and overflow
checks. In a subsequent foreground run, 2Miners accepted a CPU-verified user
share from the split path. Separate user and
developer mock sessions pass fee-routing tests, and the live 2Miners account
reported three valid user shares. See [the ETC pool test](docs/REAL_ETC_POOL_TEST.md).
Checkpointing the durable fee ledger after roughly one measured GPU second raised a
subsequent 120-second live run to about 40.8–41.0 MH/s overall on this card.
`--algorithm` is now an alias for `--algo`.

### Meowcoin preparation

Use `--check-config --coin mewc --pool HOST:PORT --wallet YOUR_MEWC_ADDRESS`
to validate a Meowcoin mainnet legacy Base58Check address and show a redacted
configuration summary. `--coin mewc` selects `meowpow` unless `--algo` is
explicitly set. JSON uses `"algorithm": "meowpow"` and
`"network": "meowcoin-mainnet"`. Arcminer's disclosed MEWC developer payout
address is `MGPRFHxNNpy3gXJ1rzrKZKZdggAa6rzisS`; its configured plain-TCP
pool is `meowcoin.cedric-crispin.com:4284`. The user chooses their own pool and
wallet. Meowcoin's published mainnet legacy address versions are 50 (pubkey)
and 122 (script). The address checker does not yet accept Bech32 addresses.
`--probe-pool --coin mewc --pool HOST:PORT --wallet YOUR_MEWC_ADDRESS` validates
one pool job without hashing or submitting a share.

### Alephium read-only probe

ALPH address formats are validated for the read-only proposal-Stratum probe:

```powershell
.\build\windows-vs\Release\arcminer.exe --probe-pool --coin alph --pool HOST:PORT --wallet YOUR_ALPH_ADDRESS
```

The probe authorizes the user account, reads one target-bearing job, and sends
no shares. The supplied user and developer accounts both passed this probe on
Cédric's GPU TCP endpoint `alephium.cedric-crispin.com:4084`. ALPH `--mine`
remains disabled until a CPU-verified live share is accepted.

The bounded CUDA share diagnostic is available separately:

```powershell
.\build\windows-vs\Release\arc_alephium_live_share_test.exe YOUR_ALPH_ADDRESS 120
```

It uses separate user/developer sessions, requests the pool's published
minimum share difficulty, verifies each candidate on CPU, and stops after one
accepted user share, timeout, or Ctrl+C. A 120-second run found no share, so
the diagnostic does not enable `--mine`.

`arc_meowpow_live_work_check HOST PORT WALLET [PASSWORD]` is a separate
read-only CUDA diagnostic. It validates a current pool job and compares one
GPU mix and final hash with the independent CPU reference; it submits no share.
See [the MEWC pool test notes](docs/REAL_POOL_TEST.md).

To run experimental mining:

```powershell
.\build\windows-vs\Release\arcminer.exe --mine --coin mewc --pool meowcoin.cedric-crispin.com:4284 --wallet YOUR_MEWC_ADDRESS --worker ArcWorker1 --password x,d=0.0128
```

The user wallet must differ from the fixed developer wallet. The password
requests the pool's published minimum difficulty for testing; omit it to use
the default `x`. The password is also accepted as `user.password` in JSON
configuration and is omitted from the redacted configuration summary.

WoolyPooly is also supported for the user session. Its live endpoint was
probed, its MeowPow job validated, and a short foreground run authorized and
hashed on the RTX 3060 Ti. Use
`--pool pool.woolypooly.com:3116 --password x` with the same wallet and worker.
Its observed initial share target was harder than Cedric's minimum; no
WoolyPooly share was found during the short check. The fixed developer session
still uses the disclosed Cedric endpoint.

The [Meowcoin mining page](https://www.mewccrypto.com/mining) identifies
MeowPow as its GPU algorithm. The [Meowpowminer source](https://github.com/Meowcoin-Foundation/meowpowminer)
uses a 7,500-block epoch, six-block program period, 16 registers, six cache
accesses, and nine math operations per round. Its README's 12,000-block epoch
statement conflicts with its source and current Meowcoin Core. Arcminer uses
the [pinned MeowPow CPU reference](docs/THIRD_PARTY.md), corrected for Core's
epoch-110 DAG size transition. The CPU and CUDA outputs match Core at heights 0,
170915, and 825000. The KawPoW GPU path is not used for MEWC. A
read-only probe of the Cedric pool authorized and validated a live job at
height 2128259 on 2026-10-27. Its standard Stratum V1 subscription shape is
covered by a mock test. Mock pool share submission and acceptance pass with
independent CPU verification. A live CPU-verified user share was accepted by
Cedric's pool at height 2128544. WoolyPooly's alternate subscription form is
also mock tested and was validated by a live read only probe.

## Dual mining roadmap

**Arcminer does not support dual or triple mining yet.** The table records the
combinations listed in [Rigel's dual-mining table](https://github.com/rigelminer/rigel#dual-mining)
as future Arcminer targets. A check means *planned for evaluation*, not a
working Arcminer command or a promise that a particular coin pair will remain
mineable. Each combination needs GPU scheduling, independent share verification,
separate pool/account routing, and real-pool testing.

The first implementation target is **RVN KawPoW + ETC Etchash**. Both user
addresses supplied for the local trial passed Arcminer's offline address checks,
and read-only probes authorized on RavenMiner and 2Miners ETC on 2026-10-30.
A portable pair validator requires separate, valid RVN and ETC account
configurations on the same GPU. A measured-active-time coordinator alternates
bounded RVN and ETC batches in a 50/50 diagnostic split. A CUDA test runs both
workers on one GPU with an ETC light cache and a bounded RVN partial DAG, then
independently CPU-verifies each GPU result. On 2026-10-30,
`arc_dual_live_work_check RVN_WALLET ETC_WALLET` authorized distinct user and
developer sessions for both coins and CPU-verified one GPU hash from each
current user job on the same RTX 3060 Ti. It submitted no shares. The diagnostic
reserves 256 MiB of free VRAM and caps the RVN cache at 16 MiB; ETC uses its
on-demand light-cache path. A separate durable batch wrapper now records one
authoritative 1% developer fee cycle across both algorithms and restores the
diagnostic 50/50 lane balance from per-coin active-time counters. Portable tests
cover fee transitions, failure recovery, lock exclusion, and restart; the CUDA
test checkpoints measured GPU time from both workers and reopens all states.
The live read-only diagnostic does not use this fee wrapper, and no dual-mining
CLI or dual share-submission loop is enabled yet.

| Primary algorithm | `+alephium` | `+sha256ton` | `+sha3x` | `+sha512256d` |
| --- | :---: | :---: | :---: | :---: |
| `abelian` | ✓ | ✓ | ✓ | ✓ |
| `autolykos2` | ✓ | ✓ | ✓ | ✓ |
| `ethash` | ✓ | ✓ | — | ✓ |
| `etchash` | ✓ | ✓ | — | ✓ |
| `fishhash` | ✓ | ✓ | ✓ | ✓ |
| `karlsenhashv2` | — | ✓ | ✓ | — |
| `octopus` | ✓ | ✓ | ✓ | ✓ |

Rigel's README also lists adding `+zil` to any single or dual algorithm
combination. That describes legacy Zilliqa PoW mining:
[Zilliqa 2.0's developer documentation](https://dev.zilliqa.com/zilliqa2/changes/)
says it uses proof of stake and mining is no longer necessary. Arcminer will
not target ZIL mainnet PoW without evidence of a
currently mineable network and independently accepted shares. There is no
current `+` algorithm syntax.
The **1% Arcminer developer share is measured in active hashing time**, not
user rewards; planned algorithms and dual mining will need to retain that
disclosed accounting policy.

## Current state

Arcminer is an **experimental KawPoW miner for one Windows CUDA GPU**, not a
production-ready multi-algorithm miner. RavenMiner accepted a CPU-verified GPU
share on 2026-10-25. In a short RTX 3060 Ti run on 2026-10-26, the fused search
path received 24 accepted user shares and zero rejects; its completed rate was
about 19.5–20.6 MH/s. These are observed short-run results, not a hashrate
promise. The GPU results match three published Ravencoin vectors, and a mock
pool has accepted verified shares through both user and developer sessions.
See [real-pool evidence](docs/REAL_POOL_TEST.md) and the
[performance history](docs/KAWPOW_PERFORMANCE.md) for details.

On the tested SM 8.6 GPU, the default fused kernel comes from pinned GPLv3
kawpowminer source and is compiled by NVRTC at runtime. Arcminer retains its
own period-specialized and generic fallbacks. The Windows ZIP includes matched
NVRTC DLLs. Set `ARCMINER_FUSED=0` to use an original Arcminer path, or
`ARCMINER_FUSED=1` to test the fused path on another GPU architecture.

## Build on Windows (Visual Studio)

Install Visual Studio with **Desktop development with C++** and a compatible
NVIDIA CUDA Toolkit. Then in a Developer PowerShell:

```powershell
cmake --preset windows-vs
cmake --build --preset windows-release
ctest --preset windows-release
.\build\windows-vs\Release\arcminer.exe --list-devices
.\build\windows-vs\Release\arcminer.exe --fee-demo
```

The Windows preset currently selects Visual Studio 2026. If you use Visual
Studio 2022, select its installed generator and an x64 build directory.

### Running on another Windows PC

The CUDA Toolkit and Visual Studio are needed only on the **build PC**.
Arcminer links the CUDA runtime statically, so a target PC does not need the
Toolkit or a separate `cudart` DLL. The miner still requires
a supported NVIDIA GPU and an NVIDIA driver compatible with the CUDA version
used for its release. The regular Windows build also uses the Microsoft Visual
C++ runtime DLLs; a target PC needs the matching Visual C++ Redistributable.
The Windows release script below links that runtime statically and packages
the two NVRTC DLLs used by the fast KawPoW path.
Other libraries introduced in later milestones must be packaged or linked
appropriately. A ZIP package can be produced after the Release build:

```powershell
cpack --config build\windows-vs\CPackConfig.cmake -C Release -G ZIP -B build\package
```

The ZIP contains `bin/arcminer.exe`, matching NVRTC and builtins DLLs for the
experimental kernel, complete buildable Arcminer source and tests, the pinned
Apache-2.0 CPU reference source, this README, dependency notes, GNU GPLv3,
the Apache-2.0 license, and the CUDA Toolkit EULA. Extract it and run
`bin\arcminer.exe --help`; start with the command line or a private config
file as described below. No config, fee state, wallet, or test binary is included. This package
has been checked on the build PC after extraction, but not on a clean target
PC with only the driver and Visual C++ Redistributable.

The default Windows ZIP embeds native KawPoW kernels for SM 7.5, 8.6, 8.9,
and 12.0. The build PC's CUDA compiler must support all four targets; set
`ARCMINER_CUDA_ARCHITECTURES` at configure time to select a different set.
Only SM 8.6 has been run and verified on local hardware. The other kernels
compile into the ZIP but still need GPU and driver tests on target machines.
Use `bin\arcminer.exe --list-devices` to check a target GPU's SM version.

## Portable CPU-only scaffold check

```sh
cmake --preset portable
cmake --build --preset portable
ctest --preset portable
```

The portable Ninja preset requires a C++ compiler on PATH. On Windows, run it
from a Developer PowerShell or use the Visual Studio preset above.
The KawPoW CPU reference is fetched from a pinned Git commit during CMake
configuration, so the first configure requires network access. Its license and
provenance are listed in [docs/THIRD_PARTY.md](docs/THIRD_PARTY.md).

## CI packages

Gitea uses [.gitea/workflows/packages.yml](.gitea/workflows/packages.yml) to
build and test Windows and experimental Linux CUDA miners. It uploads a
Windows artifact with `arcminer.exe`, `nvrtc64_130_0.dll`,
`nvrtc-builtins64_134.dll`, `LICENSE.txt`, `THIRD_PARTY.txt`, and the CUDA
Toolkit EULA, plus five coin launchers,
and an experimental Linux artifact with `arcminer` and the same notices. The
Windows host runner needs the `windows-cuda:host` label, Visual Studio 2026,
and a compatible CUDA Toolkit. The `dkg4` runner advertises `linux:host`, so
the Linux job requests `linux`. Its CUDA Toolkit is reached through
`/usr/local/cuda`, while CMake and Ninja must be available on `PATH`. These
host runners accept pushes and manual runs; pull requests do not run arbitrary
code on either host. Gitea selects the `.gitea` workflow directory before
`.github`. The jobs use the installed compatible toolchains rather than
installing a fixed CUDA version. The Windows executable links the MSVC and
CUDA runtimes statically. Keep both NVRTC DLLs beside `arcminer.exe` for the
fused and period JIT KawPoW paths.

## GitHub Actions packages

The [Build standalone CUDA miners workflow](.github/workflows/packages.yml) runs on pushes to
`main` or `master`, pull requests targeting those branches, `v*` tags, and
manual dispatch. It builds and tests both platforms, then uploads separate
Windows and experimental Linux artifacts. The Windows artifact contains
`arcminer.exe`, `nvrtc64_130_0.dll`, `nvrtc-builtins64_134.dll`,
`LICENSE.txt`, `THIRD_PARTY.txt`, `CUDA-Toolkit-EULA.txt`, and five `.bat`
launchers. The Linux artifact contains `arcminer`, `LICENSE.txt`,
`THIRD_PARTY.txt`, and five `.sh` launchers.
Pull requests build and test without uploading a release artifact.
The GitHub Windows job installs NVIDIA CUDA Toolkit 13.4.1 so its NVRTC pair
has the names shown above.
The standalone executable contains the CUDA miner and
requires a compatible NVIDIA GPU and driver to mine. GitHub-hosted runners have
no GPU, so CUDA tests that require one report as skipped there; the GPU path is
tested on local hardware separately. The Linux job builds an experimental CUDA
miner for RVN, XNA, MEWC, and ETC. The Linux
GPU paths have not yet been tested on a GPU or against a live pool. The `dkg4`
runner has no GPU, so GPU tests skip there. Mock pool session tests run on the
runner; the mining loop mock tests require a GPU. Linux uses generic CUDA
kernels if the compatible NVRTC runtime is unavailable.
The Alephium pool probe is Windows-only; Linux reports it as unavailable.
The Linux workflow artifact is not a published release or a fully static Linux
executable; a compatible NVIDIA driver and host runtime are still required.
The matching project source and third-party licenses are available at the workflow run's exact repository
commit; distribute corresponding source alongside the executable as required
by GPLv3. The workflow artifact is not automatically published as a GitHub
Release.

To build the Linux CUDA package on a Linux x64 machine with a compatible CUDA
Toolkit and C++ compiler:

```sh
cmake -S . -B build/linux-cuda -G Ninja -DCMAKE_BUILD_TYPE=Release -DARCMINER_ENABLE_CUDA=ON -DARCMINER_REQUIRE_RELEASE_MINER=ON
cmake --build build/linux-cuda --parallel
ctest --test-dir build/linux-cuda --output-on-failure
cpack --config build/linux-cuda/CPackConfig.cmake -G TGZ -B dist
```

## Windows release

From Windows PowerShell, run the release script without staging arguments:

```powershell
./scripts/package_release.ps1
```

It shows the last successful release from `docs/releases/LAST_VERSION`, suggests
the next 0.x.x or 1.x.x patch version, and prompts for any change. `-Version
0.1.9` can supply the version without a prompt. The script updates `VERSION`,
configures, builds, and tests the Windows CUDA miner, then checks the executable
with its matched NVRTC DLL pair. The resulting `dist/arcminer-VERSION-win.zip`
contains `arcminer.exe`, the two NVRTC DLLs, and five `.bat` launchers at its root. No release notes are
requested. The tested SM 8.6 GPU uses the fused KawPoW search path by default.
Keep the executable and DLLs together. A compatible NVIDIA GPU and driver and the
Windows OS are also required.

The script also creates a separate `dist/arcminer-VERSION-source.zip` with the
corresponding source and required license notices. Provide that archive
alongside the binary ZIP when distributing this GPLv3 miner. A successful run
records the version; a failed run restores the previous project `VERSION`.
Linux GPU and real-pool validation remain roadmap items for all algorithms.

## Coin launchers

Release artifacts and CPack archives include `ergo`, `etc`, `mew`, `rvn`, and
`xna` launchers. The Windows files end in `.bat`; Linux files end in `.sh`.
The Windows files are exact copies of the supplied examples, including their
pool, worker, wallet placeholder, and `pause` lines. The Linux files translate
those commands and quote the same `[WALLET ADDRESS]` placeholder for the shell.
Edit that placeholder in the launcher you want to use, then run the file.
Mining is explicit and opt-in; the disclosed developer fee is 1% of active
hashing time. The experimental Linux CUDA miner does not yet support ERG mining,
so `ergo.sh` will report that limitation when run. After downloading a Linux
workflow artifact, run `chmod +x *.sh arcminer` if the artifact service removed
executable permissions.

## Start from one command

From the extracted ZIP's `bin` directory, replace the wallet placeholder with
your own Ravencoin mainnet address:

```powershell
.\arcminer.exe --mine --algo kawpow --pool stratum+tcp://stratum.ravenminer.com:3800 --wallet YOUR_RVN_ADDRESS --worker W1
```

`--mine` is the explicit opt-in to foreground GPU hashing. The user pool and
wallet come from the command line. The separate developer session always uses
`stratum.ravenminer.com:3800`, payout address
`RREmTz9PHsKAKS7QbGh3aU7cZHUeBavMcC`, and worker `arcdev`. Arcminer validates both addresses,
prints the developer session and the 1% active GPU time policy, and stores
the fee ledger in `%LOCALAPPDATA%\ArcMiner\kawpow-ravencoin-mainnet.fee-state`.
The existing `ArcMiner` directory name is retained so earlier fee ledgers
remain available after the display-name change.
Preserve that file across runs. Add `--devices 0` to select a GPU; only one
GPU per process is supported. Press Ctrl+C to request a graceful stop: Arcminer
finishes the in-flight GPU batch, submits any verified shares, checkpoints its
active GPU time, closes both pool sessions, and exits. Closing the terminal
window or force-killing the process does not allow that drain. A command with pool and
wallet but without `--mine` will not start hashing.

The fused kernel runs by default on SM 8.6 for both one-line and config mining.
It compiles each new block period before its first fee-accounted batch. The
original period kernel remains a fallback; the generic kernel remains available
if specialization fails. To use the original generic kernel in PowerShell:

```powershell
$env:ARCMINER_PERIOD_JIT='0'
$env:ARCMINER_FUSED='0'
.\arcminer.exe --mine --algo kawpow --pool stratum+tcp://stratum.ravenminer.com:3800 --wallet YOUR_RVN_ADDRESS --worker W1
```

Remove both variables to restore the SM 8.6 default.

KawPoW, MeowPoW, Etchash, Autolykos2, and Alephium use the same live console dashboard.
ETC sessions request fresh work every ten minutes while connected. KawPoW and MeowPoW
sessions with no pool messages for twenty minutes reconnect for a fresh job
before common pool idle timeouts; a transient reconnect may briefly pause
hashing. A zero developer share count is possible even while the 1% active-time
allocation is working because shares depend on pool difficulty. The interactive
dashboard retains its recent events panel and uses compact Rigel-style GPU
hardware and hashrate tables, each with a total row. Narrower consoles use a
single GPU summary table. GPU name columns use available console width so card
names remain readable when space permits. The layout fits one GPU per row and
reduces the event area as GPU rows are added. Recent Events also receives a
compact GPU and total report every 3 minutes. Redirected output prints
timestamped events and a periodic summary without clearing the log.
Alephium samples its displayed hashrate about every 3 seconds and redraws
immediately when a share count changes.
The display format accommodates additional GPU rows, but mining still supports
only one GPU per process. NVIDIA telemetry is optional and appears as `n/a`
when unavailable. The local
rate measures completed hashes per wall
second; the GPU-kernel rate excludes pool waits and fee-state checkpoints.
Neither is a pool-reported rate, which depends on accepted shares.
Set `ARCMINER_TIMING=1` before mining to show per-sample fee checkpoint,
submission, and GPU initial/mix/final/filter timing. The extra GPU events are
recorded only in this diagnostic mode.

## JSON configuration alternative

Create a JSON file with your own user pool and valid Ravencoin mainnet payout
address. The developer pool, payout, and worker are built into Arcminer:

```json
{
  "algorithm": "kawpow",
  "network": "ravencoin-mainnet",
  "user": {"host": "your-user-pool.example.com", "port": 3333,
           "wallet": "YOUR_VALID_RVN_ADDRESS", "worker": "rig1"},
  "devices": [0]
}
```

The address string above is a placeholder and will be rejected. A `developer`
field in a JSON config is rejected, even if it repeats the built-in values. Run
`arcminer --check-config --config config.json`. Optional `--algo`, `--pool
HOST:PORT` (or `--pool stratum+tcp://HOST:PORT`), `--wallet`, `--worker` and `--devices 0,1` override user settings
from the file. The checker validates the schema, ports, device indices, and
Ravencoin mainnet Base58Check payout addresses. It prints a redacted summary;
it does not contact pools or start mining. Hostnames must currently be DNS
names or IPv4 addresses. TLS and actual device availability checks belong to
later milestones. Keep config files
private because they contain payout addresses.

To check a live user-pool handshake and one KawPoW job without mining:

```powershell
.\build\windows-vs\Release\arcminer.exe --probe-pool --config config.json
```

This validates the user account and the built-in developer account, connects only the user pool,
checks the job's epoch seed and target syntax, prints the public share target,
then disconnects. It does not
hash, submit shares, or start the developer session.

To start explicit foreground hashing on one CUDA GPU with the JSON user settings,
run:

```powershell
.\build\windows-vs\Release\arcminer.exe --mine --config build\arcminer.local.json
```

For ETC mainnet, use your own ETC payout address and the regular pool:

```powershell
.\build\windows-vs\Release\arcminer.exe --mine --coin etc --pool etc.2miners.com:1010 --wallet YOUR_ETC_ADDRESS
```

The ETC miner builds a full GPU DAG and pauses hashing unless both the user
and developer pool sessions have valid work. Its fee ledger checkpoints every
measured GPU batch. A GPU needs enough free VRAM for the current DAG plus a
256 MiB reserve; the observed DAG was 4,439 MiB on 2026-10-28. ETC uses
plain TCP and one GPU per process.

`--algo`, `--pool`, `--wallet`, and `--worker` may override the user settings.
The developer account cannot be set or overridden from a config file or CLI.
Press Ctrl+C to stop. Mining
creates `FILE.fee-state` beside the config; preserve that file across runs.
The CLI prints the absolute state path and a 1% active GPU time disclosure.
For KawPoW and MeowPow, the fee state records a pending checkpoint before
hashing and settles measured active GPU time in groups about once per second.
It also settles before a pool wait and on clean exit.
The same account mines throughout each checkpoint group; a fee-account switch
may therefore lag by up to a group plus one GPU batch. If the process is
interrupted before a group is settled, restart charges a one-second estimate
and prints the cumulative estimated time. Keep the state file across runs.
An interrupted group may have used less or more than one second of actual GPU
time; exact crash-time GPU duration cannot be recovered from a state file.
The current run loop supports one GPU, plain TCP, and one pool endpoint per
account. A disconnected account reconnects with bounded exponential backoff
(500 ms to 15 seconds) and must receive a fresh job before hashing resumes.
Hashing pauses while either account is unavailable, so pool downtime does not
count toward active GPU hashing time. Ctrl+C is checked between GPU batches;
initial DAG cache preparation checks for Ctrl+C between 16 MiB chunks.

## Networking development state

The Windows network module supports a bounded TCP handshake for one documented
KawPoW Stratum dialect: subscribe, authorize, receive `mining.notify`, and
validate the job fields. It handles `mining.set_difficulty` and validates
`mining.set_target` notifications while using the target carried by each job.
It rejects malformed
or oversized frames. Local mock tests cover success, failed authorization, and
an invalid job. The executable's read-only `--probe-pool` command connected to
HeroMiners on 2026-10-25, authorized the user account and validated a live
KawPoW job at height 4596313, without submitting shares.
Another probe on 2026-10-25 observed a share target implying about 1.07 billion
hashes per accepted share on average. At the later measured ~24,000 H/s, that
would be about 12 hours of active hashing; pool targets and actual outcomes vary.
The same day, a separate RavenMiner static-difficulty 0.001 endpoint supplied a
valid KawPoW job with a target implying about 43 million hashes per share and
accepted one CPU-verified GPU share.
The module can submit a candidate from a job issued by that same session only
after independent CPU verification. A local mock pool test checks its wire
encoding and accepted/rejected responses. A second mock pool test exercises
the full GPU mining loop and both fee routes. It has no TLS or endpoint
failover. The mining-loop mock also checks
reconnects after either account drops before a job and after a share response
is lost.

## KawPoW CPU verification development state

The CPU module validates a job's header, epoch seed, height, and 256-bit target,
then independently checks a candidate nonce and mix digest. Tests compare
three published Ravencoin KawPoW vectors and reject altered mixes, invalid
seeds, and hashes above target. The live GPU worker hands each candidate to
this verifier before submission. RavenMiner's accepted share confirms target
and nonce encoding for that pool's current Stratum dialect.

The CUDA library computes all 64 KawPoW mix rounds and final hashes. Its tests
compare Keccak stages, 512 initial registers, dataset items at two epochs, and
full hashes at three published heights with the independent CPU reference.
The persistent 64 and 256 nonce batch tests check every lane against the CPU reference
and the mock pool checks the exact nonce at an inclusive target boundary, plus
no submission when the target is one step lower.
It keeps the light cache and bounded batch buffers on the GPU across jobs and
generates uncached DAG items on demand. The search kernel distributes each hash
over 16 CUDA threads in 256-thread blocks, stores the indexed mix in shared
memory, and prepares its block-period operation schedule once. Initial and
final Keccak stages run in separate kernels. When the full DAG is cached, a
specialized search kernel omits the on-demand dataset-generation path.
Production mining transfers only compact target matches; an overflow falls
back to a complete batch scan. The diagnostic path still returns every hash.
Production mining gives the GPU a consecutive nonce range, avoiding a large
host nonce buffer and transfer for each batch; diagnostic tests retain the
arbitrary-nonce path.
Production nonce batches adapt toward 250 ms of measured kernel time, up to
2,097,152 nonces, to reduce per-batch fee checkpoint overhead while leaving
more VRAM available alongside the full DAG on 8 GB GPUs. Published vector,
full-DAG CPU/GPU equality, target-boundary and overflow, and mock-pool tests
pass. A short uncontended full-DAG live run measured about 3.87–3.90 MH/s
completed work on an RTX 3060 Ti.
It attempts to cache the full DAG when
VRAM permits and reduces the allocation in 256 MiB steps if needed. On the
RTX 3060 Ti, an epoch 612 run cached all 5,919 MiB. The earlier serial kernel
reported 23,700–25,400 H/s with 256 nonce batches. Cache preparation takes time, uses
substantial VRAM, and is not
charged as active hashing time. A more efficient DAG strategy and parallel mix
layout remain needed for practical speed.

## Roadmap

1. Configuration parser and CLI validation (implemented as an offline checker).
2. Stratum transport and per-algorithm protocol adapters, TLS, reconnect.
3. Working CUDA KawPoW hashing and independently verified share submission.
4. Active hashing-time accounting, persistent fee state, transition-safe jobs.
5. Continue Autolykos v2 performance work; add verified additional algorithms
   (Alephium is next; Flux ZelHash was excluded after its PoW-to-PoN transition).
6. Multi-GPU scheduling, performance benchmarking, and broader hardware checks.
7. Clean-PC release validation and signed Windows binaries.

## Fee policy

Official product target: **1% developer mining time**, explicitly shown.
The scheduler allocates 60 seconds per 6000 active seconds. The miner records
a pending batch before each GPU launch, then replaces it with measured GPU
kernel time in a checksummed, atomically replaced state file. A restart with
an unfinished batch charges a one-second estimate exactly once and reports
the cumulative estimate. The Windows checkpoint retries transient
sharing failures before stopping mining. It opens separate user/developer sessions and
routes each candidate only to the session that issued its job. A mock test
covers restart persistence, the fee transition, and both share routes.
The one-second recovery value is an estimate; exact time for an interrupted
kernel remains unknown, so crash accounting needs further validation before a
production release. Pool downtime pauses hashing rather than skipping a developer interval.
One percent of time does not guarantee one percent of revenue.

## Safety and transparency

No background startup, stealth mode, remote deployment, or hidden mining.
Only run on devices where the owner explicitly authorizes mining.
The project name may conflict with existing Arcminer projects; check before release.
