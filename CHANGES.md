# Changes

## 1.0.0

* First release: SafexCash (SFX) pool derived from arqma-nodejs-pool. Stratum for `rx/sfx` (RandomSFX, a
  RandomX variant with only the Argon2 salt changed), share checking with the RandomX helper `hasher/rxhash`
  built against SafexCash's own compiled RandomX library, `get_block_template` / `submit_block`, SafexCash
  style addresses (prefix `0x10003798`, 101 characters), wallet payouts with `transfer`.
* Fixed address validation (`lib/utils.js`'s `ADDRESS_PREFIXES`, `lib/pool.js`'s error message) left over from
  the ArQmA template -- was rejecting every real SafexCash address as "Invalid ArQmA address" before this fix.
* Duplicate block candidate protection in `blockUnlocker.js` (the same fix applied across every pool in this
  project -- see critical-double-credit-bug memory) applied from the start, not retrofitted later.
* Real end-to-end tested 2026-11-01: real shares accepted and credited against the real, fully-synced mainnet
  node and this pool's own real wallet.
