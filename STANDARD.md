<div align="center">

# cypherpunk4096 — the standard

**The next power of two.** `2¹²`.

[![tier: 2^12](https://img.shields.io/badge/tier-2%C2%B9%C2%B2-000000?style=for-the-badge)](#)
[![supersedes: cypherpunk2048](https://img.shields.io/badge/supersedes-cypherpunk2048-8338ec?style=for-the-badge)](https://github.com/cypherpunk2048)
[![verification](https://img.shields.io/badge/verification-green%20checkmark-2ea44f?style=for-the-badge)](#iii--verification-over-trust--the-green-checkmark)
[![quantum](https://img.shields.io/badge/signatures-quantum--ready-627EEA?style=for-the-badge)](#v--quantum-compliance)

</div>

---

## Lineage — it stands on cypherpunk2048

cypherpunk4096 does not replace **[cypherpunk2048](https://github.com/cypherpunk2048)** (`2¹¹`) — it
inherits its four commitments (write code · sovereignty over custody · power-of-two discipline ·
verification over trust) and **doubles the bar**. Everything that carried the 2048 mark is the floor
here, not the ceiling. Read the predecessor first: <https://github.com/cypherpunk2048>.

> **The rule of the doubling.** Each tier is a strict superset of the last. Meeting 4096 means meeting
> 2048 *and* the additions below — never fewer.

**A standard for the age of the terabyte.** `2⁴⁰` bytes now fits in a pocket; storage is no longer
scarce. So the old excuses expire with it: there is no reason to approximate a value that could be
exact, to prune an audit trail that could be kept whole, or to reach for a remote dependency instead
of vendoring the tree. When the terabyte is trivial, keep the whole record, carry the full precision,
and verify everything — abundance of storage is answered with abundance of proof.

**Where we stand — a timestamp in plain language.** We write this at the threshold. As of 2026 the
Bitcoin blockchain — the longest-running public ledger there is — has **not yet crossed its first
terabyte**; it is climbing toward it block by block, its growth tracking Moore's law. The age of the
terabyte is not history yet; it is dawning, and this standard is dated to that dawn. When the chain
that started it all passes 1 TB, cypherpunk4096 will already have been built for the era on the other
side of that line.

**Moore's-law measure in chronos.oracle, from kronos.agent.** chronos.oracle — the DeltaVerse
time-as-a-service — carries a **Moore's-law measure** supplied by **kronos.agent**: it indexes
historical time not only in blocks and seconds but in the doubling of storage, so "the age of the
terabyte" is a timestamp the protocol itself can read. kronos.agent computes the doubling; chronos.oracle
serves it. Progress is clocked against the doubling, not only against the wall.

## The commitments

### I · Determinism as identity
One deterministic address on every chain. The initcode **is** the name — CREATE2/CREATE3 with a fixed
salt and fixed constructor, so the same artifact resolves to the same address everywhere, verifiable
before deployment. An address that differs per chain, or depends on a deployer's nonce, does not carry
the mark.

### II · Zero dependencies
A unit meant to outlast frameworks inherits no supply chain. The whole surface is self-contained,
hand-rolled where necessary, auditable in one screen, offline-compileable. No package manager stands
between the source and the bytecode for anything meant to be permanent.

### III · Verification over trust — the green checkmark
**"Verified" means exactly one thing: the green checkmark of source-code verification on a public
explorer**, where the published source is compiled and matched **byte-for-byte** against the deployed
bytecode, reproducible by anyone with the standard-input JSON. An audit you paid for, a claim you
typed, a screenshot — none of these are *verified*. They may be called what they are. Don't trust —
verify, and let the verification be public, mechanical, and repeatable.

### IV · Precision without approximation
Quantities that can be exact **are** exact. Full-width decimals carried end to end; power-of-two
ladders where scale is discrete. Rounding is a display decision, never a storage decision.

### V · Quantum compliance
A unit built to hold value for years must survive the cryptography of those years. cypherpunk4096
requires that every signature surface be **scheme-agnostic and migratable**:

- **Signatures are `bytes`, never `(v, r, s)`.** A fixed 65-byte parameter list cannot express a
  post-quantum signature — ML-DSA-44 is 2,420 bytes; SLH-DSA-128s is 7,856 — so a `(v, r, s)`
  interface can *never* migrate schemes and becomes a tomb the day secp256k1 falls. `bytes` can.
- **ERC-1271 over EOA-only.** Accepting `bytes` also admits smart accounts, multisigs, and delegated
  EOAs as first-class signers — the treasuries that hold the most important positions.
- **Migration behind a timelock, not a redeploy.** A verifier a contract points at must be
  replaceable (e.g. a `VerifierRegistry` behind a published timelock) so the scheme can advance
  without moving a single locked asset.

A contract whose only auth path is a raw secp256k1 `ecrecover((v,r,s))` with no migration route is
**not** cypherpunk4096-compliant, however correct it is today.

## Non-compatibility statement

cypherpunk4096 is **not backward-permissive**. Compliance is all-or-nothing: an artifact carries the
mark only if it satisfies **every** commitment above — not most of them, not the ones that were
convenient. Meeting a lower standard, or no standard, does not confer 4096 by default.

It is also **not compatible with the "ship it and patch later" model.** Because compliant units are
immutable (no proxy, no upgrade, no admin over state), correctness is a pre-condition of deployment,
not a follow-up. The price of removing the key is that a bug found after deployment is permanent — the
standard accepts that price deliberately and expects the rigor that price demands.

## Software and patterns NOT accepted

An artifact that ships any of the following does **not** carry the cypherpunk4096 mark:

| Not accepted | Why |
|---|---|
| **Upgradeable proxies / `delegatecall` upgrade patterns** | Mutable code is not identity (§I); an upgrade key is a backdoor. |
| **Admin keys, owner backdoors, pausable exits on custody/locks** | Sovereignty over custody; an operator who can freeze exits does not hold a lock. |
| **CDN / external scripts / remote fetch in a shipped surface** | Zero dependencies (§II); no external host may sit in the trust path. |
| **Package-manager supply chains in permanent units** (npm/OZ imports baked into an immutable unit) | A permanent unit must not inherit a mutable dependency tree. |
| **Closed-source or unverified bytecode** | Fails the green checkmark (§III). |
| **Third-party analytics, telemetry, trackers, beacons** | Attention belongs to the one who gives it. |
| **`tx.origin` authentication, hardcoded gas, `SELFDESTRUCT`** | Fragile, upgrade-hostile, coercion-prone. |
| **Raw `(v, r, s)`-only signatures with no migration route** | Fails quantum compliance (§V). |
| **Non-deterministic / per-chain-divergent deploys** | Fails determinism as identity (§I). |
| **Floating-point or lossy storage where exact integers are possible** | Fails precision without approximation (§IV). |

## Carrying the mark

An artifact is cypherpunk4096 when: it deploys deterministically to one address across chains; it
imports nothing it cannot vendor and verify; its deployed bytecode carries the green checkmark on a
public explorer; it stores value at full precision; and every signature surface takes `bytes` and can
migrate its scheme. State it plainly, link this document, and let anyone check.

## Reference deployment

**[SCIEN·TIFIC](https://github.com/cypherpunk4096/scientific)** — the scientific-maximum ERC-20,
deterministic at `0x99999923fAb5D50Df0F3b2F89a49d18EC82Bea79` on every chain, verified on each
explorer. The first production deployment published to this commons.

---

*Stands on [cypherpunk2048](https://github.com/cypherpunk2048). CC0 — this standard belongs to no one;
adopt it, extend it, hold the line.*
