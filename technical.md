# cypherpunk4096 — technical specification

The normative companion to [`STANDARD.md`](STANDARD.md). Where the standard states the doctrine, this
document states the **checkable requirements**: what an artifact must do to carry the mark, and how any
party — human or machine — verifies each claim without trusting the author.

Key words **MUST**, **MUST NOT**, **SHOULD**, **MAY** are used per RFC 2119. An artifact is
**cypherpunk4096-compliant** only if it satisfies every MUST below. Compliance is all-or-nothing
(see [`STANDARD.md` § Non-compatibility](STANDARD.md#non-compatibility-statement)).

---

## C1 · Determinism as identity

**Requirements**
- An on-chain artifact **MUST** deploy via a content-addressed factory — CREATE2 (Nick's factory
  `0x4e59b44847b379578588920cA78FbF26c0B4956C`) or a CREATE3 rail — with a **fixed salt** and a
  **fixed constructor argument set**.
- The deployed address **MUST** be computable off-chain from `(factory, salt, initcodeHash)` (CREATE2)
  or `(factory, deployer, salt)` (CREATE3) **before** deployment.
- The same artifact **MUST** resolve to the **same address on every chain** it is deployed to. A
  deployment whose address depends on a deployer nonce **MUST NOT** claim the mark.

**Verification**
```
predicted = CREATE2(factory, salt, keccak256(initcode))     # or CREATE3(factory, deployer, salt)
assert predicted == deployed_address on each chain
```
CREATE3 is preferred where the constructor carries values that would otherwise perturb `initcodeHash`
across chains: it makes the address independent of constructor arguments.

## C2 · Zero dependencies

**Requirements**
- A **permanent** unit (immutable, non-upgradeable) **MUST NOT** import code it cannot vendor and
  verify. No package-manager edge (npm, OZ-as-dependency) may sit between source and deployed bytecode.
- Source **MUST** compile offline / airgapped. The dependency graph of a permanent unit **MUST** be
  self-contained or fully vendored into the repository.
- Tooling and front-ends **MUST NOT** load code from a CDN or fetch from any external host at runtime;
  assets are inlined or served same-origin.

**Verification**: cut the network, compile, and diff the artifact. A compliant unit builds byte-identical
with no network access.

## C3 · Verification over trust — the green checkmark

**Requirements**
- Deployed bytecode **MUST** be verified on a public explorer: published source compiled and matched
  **byte-for-byte** against the on-chain runtime bytecode.
- The repository **MUST** publish the exact build inputs — a standard-JSON input (or equivalent) plus
  compiler version, optimizer settings, and EVM version — so anyone can reproduce the match.
- "Verified" is a **reserved word**. It refers **only** to this mechanical, reproducible check. An
  audit, a self-attestation, or a screenshot **MUST NOT** be called *verified*.

**Verification**
```
recompiled = solc(standard_input)               # pinned version + settings from the repo
assert recompiled.runtime == eth_getCode(address)   # byte-for-byte
```

## C4 · Precision without approximation

**Requirements**
- Values that can be exact **MUST** be stored exact: integers or fixed-point at full declared width
  (e.g. 18 decimals / wei parity). Floating-point or lossy storage **MUST NOT** be used where an exact
  integer representation exists.
- Rounding, where unavoidable, **MUST** be a **display** operation, never a storage one, and its rule
  **MUST** be documented.
- Discrete scales **SHOULD** use power-of-two ladders (the 4096 discipline).

**Verification**: conservation and round-trip invariants — e.g. `Σ balances == totalSupply` exactly,
across arbitrary operation sequences (fuzz/invariant tested).

## C5 · Quantum compliance

**Requirements**
- Every signature-verifying surface **MUST** accept the signature as `bytes`, **MUST NOT** hardcode a
  fixed `(v, r, s)` / 65-byte layout. Rationale: post-quantum signatures do not fit — ML-DSA-44 is
  2,420 bytes; SLH-DSA-128s is 7,856.
- Contract signers **MUST** be admissible: ERC-1271 (`isValidSignature`) **MUST** be honored, so
  multisigs, smart accounts, and delegated EOAs can sign.
- The verification scheme **MUST** be migratable **without moving assets** — e.g. the verifier is an
  address the contract reads, replaceable behind a **published timelock** (a `VerifierRegistry`
  pattern). Hybrid classical+PQC verification **MAY** be used during transition.
- An auth path that is **only** raw secp256k1 `ecrecover((v,r,s))` with no migration route **MUST NOT**
  claim the mark, regardless of present correctness.

**Verification**: the ABI takes `bytes`; an ERC-1271 signer is accepted in tests; a scheme-migration
path exists and is timelocked.

---

## Non-compatibility & not-accepted (normative)

An artifact that ships any pattern in the [`STANDARD.md` not-accepted table](STANDARD.md#software-and-patterns-not-accepted)
**MUST NOT** claim the mark. Summarized: upgradeable proxies / `delegatecall` upgrades; admin keys,
owner backdoors, pausable exits on custody; CDN / remote fetch in shipped surfaces; package-manager
supply chains in permanent units; closed-source or unverified bytecode; third-party
analytics/telemetry; `tx.origin` auth, hardcoded gas, `SELFDESTRUCT`; `(v,r,s)`-only signatures;
non-deterministic / per-chain-divergent deploys; lossy storage where exact is possible.

## Time & measurement

Measurement is first-class. A cypherpunk4096 system that denominates value **SHOULD** source:
- **kairos.oracle** — price from AMM pair **reserves read directly on-chain** (the pool *is* the
  price); aggregators are enrichment, never the source.
- **chronos.oracle** — time as a service: blocktime, normalized by measured average blocktime, attested
  on-chain (never the wall clock). chronos.oracle carries a **Moore's-law measure** supplied by
  **kronos.agent**, indexing historical time in the doubling of storage.

**Historical timestamp.** This specification is dated to the *age of the terabyte*: as of 2026 the
Bitcoin blockchain has not yet crossed its first terabyte and is climbing toward it per Moore's law.
Progress is legible against the doubling, not only the wall clock.

## Conformance & claiming the mark

An artifact claims cypherpunk4096 by satisfying C1–C5 and shipping none of the not-accepted patterns.
To make the claim checkable it **MUST** publish, in its repository:
1. the deterministic address and the salt/factory used (C1);
2. a self-contained, offline-buildable source tree (C2);
3. the standard-JSON build input + explorer verification links (C3);
4. the precision invariants and their tests (C4);
5. the signature interface and scheme-migration route (C5).

State the claim plainly, link this document, and let anyone verify. The mark is earned by
reproducibility, not granted by assertion.

## Reference implementation

**[SCIEN·TIFIC](https://github.com/cypherpunk4096/scientific)** — deterministic at
`0x99999923fAb5D50Df0F3b2F89a49d18EC82Bea79` on every chain, verified on Ethereum, Optimism, Base, and
Arbitrum; zero-import; `2²⁵⁶−1` fixed supply at 18-decimal precision. It satisfies C1–C4 by
construction and C5 vacuously (no signature surface — the `overlord` gate is a plain `msg.sender`
check). A signature-bearing reference (a `bytes`/ERC-1271 door with a timelocked verifier) is the
next work.

---

*Stands on [cypherpunk2048](https://github.com/cypherpunk2048). CC0 — adopt it, extend it, hold the
line. Companion: [`STANDARD.md`](STANDARD.md) · [`llms.txt`](llms.txt).*
