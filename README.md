# safex-nodejs-pool

Mining pool software for **SafexCash (SFX)** written in Node.js: a stratum server for SafexCash's proof of
work, RandomSFX (`rx/sfx`, a RandomX variant), with share checking, block accounting through the pool wallet,
and batch payouts. It is a fork of [cryptonote-nodejs-pool](https://github.com/dvandal/cryptonote-nodejs-pool)
by Dvandal (GNU GPL v2), via its [arqma-nodejs-pool](https://github.com/newsmoneymaker/arqma-nodejs-pool)
adaptation, for SafexCash's own daemon and wallet-rpc. Live example: <https://safex.pool-pay.com>.

## What it does

* **Stratum server** (plain TCP and TLS ports, the plain cryptonote-nodejs-pool JSON-RPC shape -- `login`/`job`/
  `submit`): the pool asks the node for a block template (`get_block_template`), gives every miner the real
  hashing blob and a share target that follows the miner's hashrate (vardiff), and checks every share itself
  with a small C++ helper (`hasher/rxhash`) linked against SafexCash's own compiled RandomX library. A share
  that meets the network difficulty is submitted to the node with `submit_block`.
  **Use poolpayminer with `-a rx/sfx`** -- this algorithm has been part of the RandomX family since this
  project's original 2019 base fork, so no miner update is needed specifically for this coin.
* RandomSFX is stock [RandomX](https://github.com/tevador/RandomX) with only the Argon2 salt changed
  (`"RandomSFX\x01"` instead of upstream's `"RandomX\x03"`) -- every other parameter (scratchpad sizes,
  instruction frequencies, dataset size) is identical to the reference configuration.
* **Accounts** are SafexCash addresses (`Safex...`, base58, 101 characters), optionally `+worker`, with a
  reward mode prefix `prop:` (shared, default) or `solo:`.
* **Rewards:** PROP with time weighting (slush) or SOLO.
* **Block unlocker:** a block is settled after `depth` blocks. Duplicate block candidates (which would
  otherwise pay a miner twice for the same block) are detected and skipped.
* **Payment processor:** everyone who is due is paid in **one wallet `transfer` per round** (the network fee
  is shared by the paid miners or paid by the pool). The balance is debited before sending; a batch whose
  outcome is unknown (crash, timeout) is found again among the wallet's recent transactions on its own address
  and never sent twice; refused or failed batches go back to the balances. Dry-run mode, a whitelist for
  rehearsals and an emergency brake (`deployment/pause-payments.sh`).
* **Website and API:** a ready website (`website_example/`) with the dashboard and its graphs, blocks,
  payments, top miners, worker statistics, a "Getting started" page with a config generator, and the public
  read-only JSON API.
* **Protection against connection floods:** limits per IP, a login deadline, an optional IP allow list, banning
  of miners with many invalid shares.

## Developer donation (please read)

The pool takes a **developer donation** from the reward of every block it finds, before the miners' shares are
computed. It is configured in `config.json` (see `config_examples/sfx.json`):

```json
"blockUnlocker": {
  "poolFee": 2,
  "donations": { "Safex5yy5ox1pNxAH8RsbQbCgoZgPmk695Pedf9USsFj3PF7BbyTtVR8NwQ19UuZTYa25MBgwVq98Jrm8ZiUM7iPGGdwVPh1Viy4d": 1 }
}
```

* `poolFee` is the fee of the pool operator (percent). `donations` is a table `SafexCash address -> percent`
  for the developers of this software; **the default is 1% to the address of the project's own pool**. The
  donation is paid like any other balance, and is additive with `poolFee` (total real cut from miners here is
  `poolFee + donations`, e.g. 2% + 1% = 3%).
* It is included in the "Pool Fee" figure that the pool's website and API show to the miners (as in the
  original software).
* You are free to change the percentage, the address or to empty the table (`"donations": {}`): it is your
  pool and the license is the GPL. Please tell your miners the truth about the fees of your pool.
* This has nothing to do with the miner poolpayminer (a separate project with its own fee).

## Installation

Requirements: Linux, Node.js 18 or newer, Redis, a SafexCash node (`safexd`, built from
[github.com/safex/safexcore](https://github.com/safex/safexcore)) synchronised with the network, and its own
`safex-wallet-rpc` with the pool wallet, a C++11 compiler and SafexCash's own compiled `librandomx` (for the
`rxhash` helper), a web server for the website and a TLS certificate for the TLS stratum ports.

**Build `rxhash` against SafexCash's OWN RandomX, never another coin's prebuilt library** -- the only
difference from stock RandomX is the Argon2 salt, baked into SafexCash's own `external/randomx/src/
configuration.h`. See `hasher/Makefile` for the exact build command.

If your node and pool run on separate hosts, point `config.json`'s `node`/`wallet` hosts/ports at an SSH
tunnel to the node host, same as this pool's own live deployment.

## Miners

This algorithm (`rx/sfx`) has been supported since poolpayminer's original 2019 xmrig base fork -- any version
works. Use [poolpayminer](https://github.com/newsmoneymaker/poolpayminer) (Windows and Linux, free, has its own
fee). Example: `poolpayminer -o safex.pool-pay.com:5102 --tls -u YOUR_SFX_ADDRESS+rigname -p x -a rx/sfx`.

## Money warning

The payment processor moves real coins. Rehearse first: `"dryRun": true`, then a whitelist (`onlyAccounts`)
with a few small payouts of your own, then enable it. A transaction that has been sent to the network can not
be cancelled. Keep the wallet's keys/seed backed up separately and privately.

## License and credits

GNU GPL v2 (see [LICENSE](LICENSE)). Based on cryptonote-nodejs-pool, Copyright (c) Dvandal and contributors.
SafexCash itself is a separate program and is not included.
