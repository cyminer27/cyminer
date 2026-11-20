# Arcminer release notes

## v1.0.3 notes

- Added experimental single-threaded Zephyr Protocol (ZEPH) RandomX CPU mining on Windows. Enable it explicitly with `--mine --experimental-randomx`. Read-only pool probing is available. Live share acceptance is still unverified.
- Added development groundwork for Kaspa (KAS), Conflux (CFX), Zano (ZANO), Gram (GRAM), XTM, XELA, and Zilliqa (ZIL). Mining for these coins remains disabled. Karlsen (KLS) also remains at the scaffolding stage.
- Added Radiant (RXD) CUDA search and pool-session tests against mock pools. Radiant remains in development.
- Updated Windows and experimental Linux release builds to reorder selected host functions and restored the GPLv3 license required by packaging.

Arcminer remains opt-in. The developer fee is 1% of active hashing time.

**Download status.** The latest ZIPs in this repository are v1.0.2. These are draft notes for v1.0.3.

## v1.0.2

- Added experimental Zent Cash (ZTC) mining on Windows. A mainnet user share has been accepted.
- Aurora Borealis Coin (ABRS) now has accepted mainnet user shares.
- Added experimental Ravencoin CPU mining, including a CPU and GPU mode. A live pool has not yet accepted a CPU-found share.
- Added secure pool connections for Windows KawPoW and MeowPoW mining.
- Added Neoxa (NEOX) and Karlsen (KLS) mining choices. Live share acceptance is still unverified for these and Neurai (XNA).
- KpowBIT (KPOW) can be checked in the configuration, but mining is not yet available.

Arcminer remains opt-in and uses 1% of active hashing time for its developer fee, not 1% of rewards. See [README.md](README.md) for the full coin list and setup steps.

**Download status:** The ZIPs currently in `release/latest` are still v1.0.1. The v1.0.2 packages have not been uploaded yet.

## v1.0.1

### Release overview

- Windows x64 standalone package for NVIDIA CUDA GPUs. An experimental Linux x64 CUDA package is also available.
- Foreground, opt-in mining on one GPU per process.
- Ravencoin (KawPoW) mining has accepted mainnet shares. Ergo (Autolykos v2), Ethereum Classic (Etchash), Meowcoin (MeowPoW), and Alephium mining have also accepted mainnet shares and remain experimental.
- Added XNA as a coin choice with Windows and Linux launchers. Its live pool share acceptance remains unverified.
- Added Aurora Borealis Coin (ABRS), a KawPoW coin, with `abrs.bat` and `abrs.sh` launchers in the v1.0.1 packages. Live pool share acceptance by Arcminer has not yet been verified.
- Candidate shares are checked on the CPU before submission.
- The disclosed developer fee uses 1% of active GPU hashing time through a separate developer pool account; it is not a percentage of rewards.

### Before running

- Set your own coin-specific wallet address in a launcher or on the command line. Arcminer refuses an invalid wallet address.
- Use a compatible NVIDIA GPU and driver. Keep the Windows executable and included DLLs together.
- Preserve the `.fee-state` file between runs so active hashing time carries over. Stop with Ctrl+C for a clean shutdown.

### Known limitations

- Mining has not been validated on every GPU, driver, or pool.
- XNA mining has not yet been verified by an accepted live pool share.
- ABRS mining has not yet been verified by an accepted live pool share.

See [README.md](README.md) for setup and [README-TECHNICAL.md](README-TECHNICAL.md) for detailed status.
