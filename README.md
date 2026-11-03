# Arcminer

> **Picture goes here:** Add an Arcminer screenshot or banner above this guide.
>
> Suggested Markdown: `![Arcminer dashboard](path/to/your-picture.png)`

Arcminer is an experimental cryptocurrency miner for Windows PCs with an NVIDIA CUDA GPU. It runs in the foreground, so you choose when to start and stop it. It currently mines on **one GPU per process**.

**Developer fee:** Arcminer uses **1% of active GPU hashing time** for a separate developer pool account. This is time spent hashing, not 1% of your rewards. The dashboard shows the active account and fee time.

## Get mining

1. Extract the Windows release ZIP. Keep `arcminer.exe` and the included DLLs together.
2. Pick a coin launcher: `rvn.bat`, `ergo.bat`, `etc.bat`, or `mew.bat`.
3. Open the launcher in a text editor and replace `[WALLET ADDRESS]` with **your own address for that coin**. Save the file.
4. Run the launcher. Press **Ctrl+C** in the miner window to stop cleanly.

You need a compatible NVIDIA GPU and driver. A CUDA Toolkit and Visual Studio are only needed if you build Arcminer yourself. If your wallet address is invalid, Arcminer will refuse to start.

### Run from PowerShell instead

From the folder containing `arcminer.exe`, this example starts Ravencoin mining. Replace the wallet address before running it:

```powershell
.\arcminer.exe --mine --coin rvn --pool stratum+tcp://stratum.ravenminer.com:3800 --wallet YOUR_RVN_ADDRESS --worker ArcWorker1
```

`--mine` starts foreground hashing. Run `.\arcminer.exe --list-devices` to see your GPUs, then add `--devices 0` if you need to choose one. Run `.\arcminer.exe --help` for all options.

## Coins you can choose

| Coin | Algorithm | Status |
| --- | --- | --- |
| Ravencoin (RVN) | KawPoW | Mainnet shares accepted |
| Ergo (ERG) | Autolykos v2 | Mainnet shares accepted; experimental |
| Ethereum Classic (ETC) | Etchash | Mainnet shares accepted; experimental |
| Meowcoin (MEWC) | MeowPoW | Mainnet share accepted; experimental |
| Alephium (ALPH) | Alephium | Mainnet share accepted; experimental |

Use the launcher or `--coin` value for the coin you intend to mine. Other algorithms mentioned in the project notes are planned or diagnostic work; they are not miner choices. XNA has a launcher, but live pool share acceptance is still unverified.

## What to expect

The dashboard shows your pool connection, accepted and rejected shares, GPU activity, and the developer account. A share may take time to appear. Arcminer checks candidate hashes on the CPU before submitting them.

Arcminer pauses between batches when the GPU reaches 80 °C and stops hashing above 85 °C. It requires a working temperature reading. Optional clock and power controls are described in [GPU tuning notes](docs/GPU_TUNING.md); leave them off unless you intend to change your GPU settings.

Keep Arcminer's `.fee-state` file between runs so active hashing time carries over. Arcminer prints its location when mining starts. Use **Ctrl+C** to let it finish its current batch and save the state.

Arcminer is still experimental. Live shares have been accepted for the coins listed above, but the miner has not been validated on every GPU, driver, or pool. Use it only on a device you are authorized to mine with.

## More information

- [Detailed project and development notes](README-TECHNICAL.md)
 
 
 
