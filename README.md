# Monero360 - XmrRig Monerod & P2Pool Docker Stack
Help the community to secure the blockchain by running this simple, all-in-one `docker-compose` setup to run a full Monero P2Pool mining config. It includes:
1.  **Monero Node (`monerod`):** A pruned node to sync and validate the blockchain.
2.  **P2Pool:** The decentralized pool server that connects to your node.
3.  **XMRig:** The CPU miner that connects to your local P2Pool.

It's designed to be "set it and forget it." All services will automatically restart on machine reboot.

## Table of Contents
* [Quick Start](#-quick-start)
* [Checking the Status](#-checking-the-status)
* [Configuration](#-configuration)
* [Services used](#-services-used)


## Intro : 

If you want to support my future XMR related-work you can send a tip here 

`8573DHrvz9i9HgKZgJWBUxTzQTi3yDFStLuecoPqf79pNWNzG8SFH88adQcQEVy48QD3osyqySBiW3PWVHMWqv5yEkn1LML`

Otherwise you can just star this repo, run your own node and help the community secure the blockchain, it will make me very happy

You can also follow me on [X/Twitter @MelkoXMR](https://x.com/MelkoXMR)

## Quick Start

1.  **Install Docker & Docker Compose.**
    * [Get Docker](https://docs.docker.com/engine/install/)
    * [Get Docker Compose](https://docs.docker.com/compose/install/)

    If it's too complicated just ask your favorite LLM how to install docker & docker compose on your OS and i'm sure it will be happy to help you

Optional : **If you want to allocate huge pages memory access for better perfs on your mining**

```bash sudo sysctl -w vm.nr_hugepages=1280```

2.  **Clone this Project (or Download Files).**
    If you have Git:
    ```bash
    git clone https://github.com/Buggs777/Monero360.git
    cd Monero360
    ```
    Otherwise, just create a new folder and add the files from this repository.

3.  **Edit the `.env` file.**
    This is the **only file you need to edit**.
    ```bash
    nano .env
    ```
    Fill in your details:
    * `WALLET`: Set this to your Monero wallet address.
    * `CPU_PERCENTAGE`: Set the max CPU usage you want to allow to the miner (e.g., `75`).
    * `P2POOL_FLAG`: Leave as `--mini` (recommended for CPUs) or set to empty for the main pool.

4.  **Start everything.**
    Run this command in your terminal:
    ```bash
    docker-compose up -d
    ```

### First Launch : 
On your first launch, every container is gonna return "conection refused", it's absolutely normal because Monerod has to first sync the blockchain, check the next part to see how to follow the sync (it might take some a few hours or days for the first launch)

No worries you can turn off your device at any time, it will save the syncying and start again automatically at  reboot

## Checking the Status

You can check the logs for each service at any time.

### 1. Check `monerod` (The Node)

This is the most important first step.
```bash
docker-compose logs -f monerod
```

Wait until you see `You are now synchronized with the network.` **Nothing else will work until this is complete.**

### 2. Check `p2pool` (The Pool)

`p2pool` will wait patiently for `monerod` to sync.
```bash
docker-compose logs -f p2pool
```

### 2 .3. Check xmrig (The Miner)
xmrig will repeatedly try to connect to p2pool.
```bash
docker-compose logs -f xmrig
```
While waiting: You will see connection refused or DNS error (this is normal, p2pool isn't ready yet).

Success: Once p2pool is ready, xmrig will connect. The magic word you are looking for is ACCEPTED (in green text). This confirms you have successfully submitted a share to the pool.

## - Configuration

All user configuration is done in the `.env` file.

| Variable | Description | Example |
| :--- | :--- | :--- |
| `WALLET` | Your Monero wallet address (for payouts). | `48...` |
| `CPU_PERCENTAGE` | Max CPU load (1-100). `xmrig` will auto-tune. | `75` |
| `P2POOL_FLAG` | Set to `--mini` for the low-hashrate P2Pool "mini" sidechain. Highly recommended for CPUs. Set to empty (`P2POOL_FLAG=`) for the main pool. | `--mini` |

## Services used

The three services work together in a chain:

**`monerod`** ➔ **`p2pool`** ➔ **`xmrig`**

1.  **`monerod`** syncs the blockchain and provides block templates.
2.  **`p2pool`** gets these templates from `monerod` and runs the decentralized pool, offering a stratum connection on port `3333`.
3.  **`xmrig`** connects to `p2pool`'s stratum, gets mining jobs, and starts hashing.