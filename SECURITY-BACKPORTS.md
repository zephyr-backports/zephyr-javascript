# Security backports

Base: `ZephyrProtocol/zephyr-javascript` commit `9d022ded` ("Updates for version 2.3.0", 2025-06-16;
the state the Zephyr v2.3.0 wallets were built from), itself on `woodser/monero-javascript` `1dd86702`
(2023-06-09). Policy: https://github.com/zephyr-backports/zephyr-backports (docs/POLICY.md). Each entry
is one cherry-picked monero-ts commit into the C++ bridge, one dependency pin (trailer `Advisory:`) or one
artefact rebuild (trailers `Built-From:`); the Type column is `pick`, `pin` or `artefact`.

**v2.3.0-sec.1** (draft, not tagged)

| # | Tag | Type | Advisory / source | Component | Summary |
|---|---|---|---|---|---|
| SB-035 | v2.3.0-sec.1 | artefact | zephyr `v2.3.0-sec.2`, zephyr-cpp `v2.3.0-sec.1` | `dist/` (WASM, glue, worker bundle) | the wallet WASM rebuilt from the security forks, so their backports reach the library |

## Advisories

<!-- One section per change; template: docs/ADVISORY-TEMPLATE.md in zephyr-backports/zephyr-backports -->

### SB-035 — The wallet WASM rebuilt from zephyr v2.3.0-sec.2 and zephyr-cpp v2.3.0-sec.1

- **Tag:** v2.3.0-sec.1
- **Type:** artefact
- **Origin:** built from the release set of 2026-11-07: zephyr-backports/zephyr `v2.3.0-sec.2`
  (`8c6a120256b4bd887d6e7167d929d76e3507e0e5`), zephyr-backports/zephyr-cpp `v2.3.0-sec.1`
  (`3a2ee602ee28e6b7941b617ad4706349ad992641`) and ZephyrProtocol/zephyr-javascript `9d022ded2a8d188127173aa7cf79fdd5ee006353`
  (this fork's base), with the webwallet project's reproducible WASM build (`tools/wasm/build.sh`, sha256
  `05ff98b3835515d82cac03b8f86ae740586c8f60a36badd15deda6041c8eefc5`: emsdk 3.1.10, boost 1.84.0, OpenSSL and
  unbound headers only) at the canonical build path `/tmp/zeph-wasm/build`. The commit carries one
  `Built-From:` trailer per source; `dist/build-manifest.json` records pins, patches, toolchain, flags and output hashes,
  `dist/SHA256SUMS` the hashes of the seven `dist/` files.
- **Class:** the classes of the advisories it carries: memory, dos, funds, privacy
- **Component:** `dist/monero_wallet_full.{wasm,js}`, `dist/monero_wallet_keys.{wasm,js}` (the wallet core compiled
  to WebAssembly, with its emscripten glue) and `dist/monero_web_worker.js` with its source map (the web worker bundle,
  which includes both glue files)
- **Affects:** every user of this library's prebuilt `dist/`, which is what the Zephyr web and desktop wallets
  ship: the wallet core in upstream's `dist/` is built from zephyr v2.3.0 and zephyr-cpp `4a99272`, without the
  backports. The WASM scans on-chain data and talks to a remote node, so the untrusted input the backports guard
  against reaches it.
- **Impact:** the rebuilt WASM contains the backports whose changed files are compiled into it, 19 of the 34
  advisories of the release set: zephyr SB-001 to SB-009 and SB-015 to SB-019 (epee, crypto, serialization,
  wallet2, logging) and zephyr-cpp SB-011 to SB-014 (monero-cpp wallet and model). SB-010 and SB-020 to SB-034
  (DNSSEC anchor, depends, RandomX, Windows build, daemon RPC and ZMQ, levin) do not reach the WASM (file-level check
  against the build's compiler dependency files). Without the rebuild, those fixes exist in the forks but not in the
  library wallets run.
- **Compatibility statement:** POLICY §4 checked: the WASM is built from the release set, whose advisories each carry
  their own compatibility statement (rules 1 to 5); the rebuild adds no source change of its own beyond the webwallet
  project's build patches, which the manifest lists (guard 4: the modified upstream files are exactly those the
  patches touch). The JavaScript API is unchanged: zephyr-javascript's source is untouched, the C++ bridge is built
  from the base commit, and the unresolved imports match the pinned sets (guard 2: full 10, keys 3 symbols). Wallet files: the harness's cross-open
  passes both ways with upstream's prebuilt `dist/` (data saved by either opens in the other). The version string is
  the upstream one.
- **Conflict resolution:** none (no cherry-pick)
- **Test evidence:** done 2026-11-07 (local run; files in evidence/wasm/2026-11-07/): WASM build of the release set at
  `/tmp/zeph-wasm/build`, guards 1–4 OK, no host path in any output (`set-2026-11-07-build-manifest.json`,
  `harness-results.md`, section "Release set 2026-11-07"); `keys.wasm`, both `.js` and the symbol map byte-identical
  to the 2026-11-07 build of the published sec.2 commits, `full.wasm` different only by the build path in two strings;
  harness smoke test (keys from seed, 40-block sync to tip, reserve info, serialisation, reopen) OK and cross-open both
  ways with upstream's prebuilt dist (`9d022ded`, keys `a541c83c`) OK; worker bundle built with zephyr-javascript's own
  `npm run build_web_worker` (webpack 5.77.0), which on upstream's own `dist/` reproduces upstream's bundle
  byte-for-byte (`set-2026-11-07-dist.md`); which advisories are compiled in: `set-2026-11-07-wasm-inputs.md`. Pending:
  fork CI provenance run on the published branch; the signature of the build manifest with the release key; the
  wallet flows the harness does not reach (sending, which needs funds or a mocked node).
- **Notes:** the worker bundle also contains third-party JavaScript (`crypto-js`, `ajv`, `lodash`, `qs`) at the
  versions of the base lockfile (`crypto-js` 4.1.1, `ajv` 6.12.6, `lodash` 4.17.21, `qs` 6.5.3). Byte identity with
  upstream's `dist/` is not a goal (upstream's build embeds paths of its build machine); byte identity between two
  builds at the same path is (2026-11-05, same inputs, same path: identical bytes).
