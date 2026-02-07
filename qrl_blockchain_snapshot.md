# QRL Blockchain Snapshot

Syncing your QRL Node from scratch is a slow process that could take about 2 weeks.

If you don't want to wait that long, you can download a recent snapshot of the blockchain data and continue syncing from that. This will get you to a fully synced state in less than an hour.

## Download Latest Snapshot

**📅 Date:** 2025-11-23

**💻 Size:** 15.80 GB

**💾 Torrent File:** [qrl-snapshot-2025-11-23.torrent](/files/qrl-snapshot-2025-11-23.torrent)

**🧲 Magnet Link:** 
```
magnet:?xt=urn:btih:9db9c705d5f80cfe7e68713fe7e0b818e1b0f1e5&dn=qrl-snapshot-2025-11-23.tar.gz&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce
```

---

The blockchain state is in the form of `*.tar.gz` archive available for download as a torrent. You can download it using any standard app, like [qBittorrent](https://www.qbittorrent.org) or [Transmission](https://transmissionbt.com).

*Note the current snapshot contains over 7 years of QRL history. It likely won't be updated again until the QRL 2.0 is released (which is expected to be in 2026). The final snapshot of QRL 1.0 blockchain will then be available here.*

## How to Use

To extract the archive on Windows you might need app like [NanaZip](https://github.com/M2Team/NanaZip) or [7-Zip](https://www.7-zip.org). On Linux you can extract it using the native `tar` command.

The extracted archive contains a single folder `state` which needs to be put into a correct place where your QRL node expects it. That varies based on the operating system and your configuration.

If you used my [QRL Node - Docker Setup Guide](qrl_node_docker.md), read the relevant [section](qrl_node_docker.md#optional-fast-sync-with-blockchain-snapshot) there, because the files are in a different path and require special ownership.

Otherwise the data are by default stored in `~/.qrl/data/` (on Linux), or `C:\Users\<username>\.qrl\data\` (on Windows).

### The general approach

1) Stop the QRL node if it is already running.
2) Identify where your QRL data are stored (here we assume it's `~/.qrl/data/`).
3) If there is already `state` folder inside the data folder, delete it.
    ```bash
    rm -rf ~/.qrl/data/state
    ```
4) Extract the `.tar.gz` archive into the data folder.
    ```bash
    tar -xzf qrl-snapshot-2025-11-23.tar.gz -C ~/.qrl/data/
    ```
    This should result in this path having many files inside `~/.qrl/data/state/...`.
5) Start the QRL node again.

The node should now sync the remaining blocks, starting from the snapshot height.
