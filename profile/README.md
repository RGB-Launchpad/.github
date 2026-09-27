<p align="center">
  <img src="https://raw.githubusercontent.com/RGB-Launchpad/.github/main/profile/media/banner.svg" alt="Darkhorse — launch and trade RGB assets on Bitcoin, verify every step" width="900">
</p>

<p align="center">
  <a href="https://x.com/RGBHorse">@RGBHorse</a>
</p>

## What it is

An RGB asset is not a row in a contract on some chain. Its supply is sealed to a Bitcoin UTXO,
its history travels with whoever holds it, and the block carries a 32-byte commitment and
nothing else. Bitcoin's guarantee that a UTXO can be spent once is what protects ownership of
the asset committed to it.

Darkhorse issues those contracts and runs the market for them. A token's pool exists from the
moment the token is created: there is no listing step to wait for and no allocation held back.
The whole supply goes into the pool, and a creator who wants a position buys it on the same
curve as everyone else.

Matching happens off the chain; the asset stays on it. A trade settles in the platform ledger,
so it waits for no confirmation and pays no miner fee. The token itself is a real RGB contract
and can be withdrawn to a wallet whose keys you hold, at any time.

While a balance sits on the platform, the platform holds it. That is custody, and the two
sections below are what it can be checked against.

## The life of a token

| Stage | What happens |
|---|---|
| **Launch** | The RGB contract is issued and the pool is created. The creator may buy in the same step |
| **On the curve** | Price follows the bonding curve; the real reserve accumulates |
| **Graduation** | The reserve reaches the threshold: outside liquidity opens and the token enters the main listings |
| **After** | Trading continues in the same pool, and anyone may provide liquidity |

**The pool never migrates.** Graduation is a threshold being met, not a move: no new contract,
no new pool, no migration window, and holdings are untouched. Each pool's parameters are fixed
when it is created; a later change to the platform defaults applies only to pools created after
it.

| Default | Value |
|---|---|
| Total supply | 1,000,000,000, precision 8 |
| Free allocation to the creator | 0 |
| Starting virtual reserve | 5,000,000 sats |
| Graduation threshold | 12,500,000 sats |
| Open to graduation | 12.25× |
| Swap fee | 1% on the curve, 0.5% after graduation |
| Launch fee | 5,000 sats |

The swap fee on the curve goes 20% to the creator and 80% to the platform. After graduation it
is 35% platform, 55% by liquidity share, 10% creator — the share of the locked liquidity has no
owner and goes to the platform, while real providers keep their part in full. Splits accrue as
claimable rather than landing in a balance trade by trade. A referral rebate is carved out of the
platform's share, so the creator and the liquidity providers receive the same with a referrer as
without one.

## What can be checked

Every accounting action writes one ledger record. The records form a hash chain, and both
halves of a record — the public fields and the private ones — enter their record's hash as
commitments, never as text. So the chain can be published without the contents being public,
and altering any record in history makes every later hash disagree.

| Published | Not published |
|---|---|
| Sequence, timestamp, previous hash, this hash | Account identity, withdrawal addresses |
| Both commitments, and the salt of the public half | The private payload |
| The public payload of a trade and of a platform event | Bitcoin transaction ids |

Three things carry that further:

- **The chain head goes into a Bitcoin transaction once a day.** After that, the age of that
  stretch of history is fixed by Bitcoin's block time rather than by our word.
- **Trade payloads are written to Arweave**, one batch per time window, each batch naming the
  transaction before it, with a daily manifest carrying each batch's `sha256`. The figures on
  the verification page are read back from Arweave, not from the platform.
- **Reserves are published daily against liabilities**, as a merkle sum tree per asset: every
  node carries a hash and the liability beneath it, and folding your own leaf up its path has
  to arrive at the published root *and* the published total.

Where a recomputation and the platform disagree, the recomputation is the answer. The hash
formulas, the endpoints and the procedure are published, and so is what each check cannot settle
— a sum tree proves the published total equals the sum of the leaves in it, but not that no
account was left out. An omitted account's own proof fails, which is how that gap closes.

<p align="center">
  <img src="https://raw.githubusercontent.com/RGB-Launchpad/.github/main/profile/media/verify.webp" alt="The verification page: batches on Arweave, what a published trade payload contains, and what it never contains" width="900">
</p>

## What it does not claim

| | |
|---|---|
| **It is not non-custodial** | A balance held on the platform is held by the platform. Withdrawing to your own wallet is what makes it yours to control |
| **It does not vouch for a token** | Anyone can launch. A listing mark records that an application was checked against published criteria; it says nothing about whether an asset is safe or worth anything |
| **It does not decide graduation** | The threshold is public data compared with a public number. Nobody can bring it forward or hold it back |
| **The engine is beta software** | Upstream describes rgb-lib as beta and unaudited. That applies here; keep mainnet amounts small |
| **Test networks carry no value** | Signet and regtest exist for trying things out. Bitcoin and tokens on them are worth nothing |

## The wallet

**[Darkhorse Wallet](https://github.com/RGB-Launchpad/wallet-ext)** is a browser extension for
RGB assets on Bitcoin. Keys and consignments stay on the device; the wallet validates a
consignment itself, in the tab. Source, release notes and `SHA256SUMS` for every build are in
the repository, and a site can ask it to sign in or to produce an invoice without ever seeing a
key.

**[⬇ Download the latest release](https://github.com/RGB-Launchpad/wallet-ext/releases/latest/download/rgb-wallet.zip)**
· [Installation guide](https://github.com/RGB-Launchpad/wallet-ext/blob/main/INSTALL.md)

## Built on other people's work

RGB comes from the [RGB Working Group](https://github.com/RGB-WG) and the
[LNP/BP Standards Association](https://www.lnp-bp.org/). The engine behind every contract,
consignment and transfer here is [rgb-lib](https://github.com/RGB-Tools/rgb-lib) by
[RGB-Tools](https://github.com/RGB-Tools), MIT. Running that engine inside a browser tab rests
on [rgb-lib-wasm](https://github.com/UTEXO-Protocol/rgb-lib-wasm), the WebAssembly bindings
maintained by [UTEXO](https://github.com/UTEXO-Protocol) under the same licence. Neither
project is ours, and neither is a small part of this.

## Repositories

| | |
|---|---|
| [**wallet-ext**](https://github.com/RGB-Launchpad/wallet-ext) | Darkhorse Wallet: the browser extension, its release zips and their checksums. No npm dependencies and no bundler. Apache-2.0 |
| [**rgb-lib-wasm**](https://github.com/RGB-Launchpad/rgb-lib-wasm) | A fork of UTEXO's WebAssembly bindings of rgb-lib. Two branches hold fixes for things the wallet hit in the browser. MIT |

The platform's own services are not published. What it exposes instead is a read-only API and
the ledger behind it, both documented, and both usable without an account.
