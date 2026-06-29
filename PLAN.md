# PLAN — bigger-pi (distributed, verified, open computation of more digits of π)

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

## Executive summary

**bigger-pi** is an open, distributed, **verification-first** effort to compute more digits of π on
donated compute. The intrinsic value of "more digits" is modest; the public good is the
**open, reproducible, independently-verified toolchain + chunk-coordination protocol + provenance
ledger** that produces them — most record-scale π software today is closed, bespoke, or
unreproducible. bigger-pi turns a famously single-host, heroic-effort computation into a
**citizen-science map-reduce** that ordinary donated machines can meaningfully contribute to, and
proves every digit it publishes.

The work runs in **two complementary, chunkable methods**:

- **Method A — Chudnovsky + binary splitting (the full computation).** The series sum over a
  term-range `[a, b)` reduces to an exact integer triple `(P, Q, T)`. Independent workers compute
  sub-ranges in parallel; the triples merge hierarchically (a clean, associative map-reduce). Each
  leaf range is a bounded, **credit-sized** task sized to one donated session. *Known constraint:*
  the final full-precision multiply / divide / sqrt that assembles the record-scale result is
  **memory-bound and consolidates on a single host** — distributed chunking scales the series work;
  final assembly is the bottleneck at the frontier.
- **Method B — BBP / Bellard hex-digit extraction (embarrassingly parallel + cross-check).**
  Computes hex digits of π at an **arbitrary position without the preceding digits** — every chunk
  ("hex digits at offset *k*") is fully independent, ideal for donated sessions **and** as an
  independent-formula spot-check of Method A's results.

The project **alternates two task families**: **(1) algorithm/impl work** (chunking, bignum/FFT
multiplication, merge, coordination, GPU/distribution) ↔ **(2) computation runs** (executing chunks
to extend verified coverage).

**The HARD RULE — the identity of the project.** *No digit result is accepted into the ledger or
published without independent verification.* Verification means **two independent full computations
agree bit-for-bit** (different implementation and/or bignum backend, ideally different operator)
**and** an independent-code-path **BBP/Bellard hex-digit spot-check** passes, **and** every
constituent chunk was itself recompute-verified. This gate is not a feature bolted on at the end; it
is the spine, wired in at M0, and nothing ships around it.

**Risk tier: medium.** Not health/legal/safety, but medium for three real reasons: (1) publishing
**unverified or false digits** would damage trust and waste effort; (2) large computation has a real
**energy/compute cost** that must be capped and justified — no "compute for compute's sake"; and
(3) a distributed-compute coordinator is **abuse-prone** — it must never become a covert botnet or
run on non-consenting machines. The medium-risk gate is satisfied by mandatory expert review of the
**verification methodology** (guard-digit analysis, spot-check sufficiency, two-method independence)
before any record-scale run is published.

**Honesty note on the partner & need.** This is a **moonshot** with indirect beneficiaries (open-
numerical-computing, math/CS education, distributed-compute citizen science). **No partner
organization, MOU, or externally "verified need" exists yet.** Partner and verified-need status are
**TO BE SECURED**; until then `verifiedNeed=false` and the project ships as a generic open public
good measured by reuse and verification integrity, not by a beneficiary's request.

## Problem & beneficiaries

**The problem.** Computing π to extreme precision is one of the canonical benchmarks of numerical
computing, yet the state of the art has three gaps that bigger-pi exists to close:

1. **It is effectively single-machine and elite.** Record runs require one very large, very
   expensive consolidation machine and bespoke engineering; ordinary contributors are spectators.
   π is genuinely *chunkable*, so distributed donated compute *can* be a real contributor — but no
   open, well-documented coordination protocol packages the work into credit-sized units.
2. **The toolchain is largely closed or unreproducible.** Much record software is proprietary or
   one-off; results are announced but rarely **independently reproducible from a public ledger**.
3. **Verification is under-shared.** The methods that make a digit claim trustworthy (dual
   independent computation + BBP cross-check + guard-digit analysis) are folklore among a few
   experts, not an open, reusable harness others can apply to their own long computations.

**Who is helped (beneficiaries).**
- **Open numerical-computing & distributed-computing communities** — get a reusable, MIT-licensed
  bignum + binary-splitting + NTT/FFT toolchain and a documented **distributed verified map-reduce**
  pattern applicable far beyond π.
- **Math / CS education** — binary splitting, BBP/Bellard extraction, and NTT multiplication are
  superb teaching artifacts; the explainers and reference impl are classroom-ready.
- **Citizen-science compute donors** — people who want their spare CPU/GPU to do something open and
  verifiable, with a real (not token) contribution and a public ledger crediting it.
- **Researchers** — a verified, CC0 digit corpus is a clean substrate for normality/statistics
  study and a benchmark/oracle for testing other arbitrary-precision software.

**Verified need / partner org: TO BE SECURED.** No named partner (e.g., a university math/CS
department, or an established distributed-computing project such as a BOINC-style platform) has
confirmed a need. General community value is plausible and the artifacts are independently useful,
so foundation work (M0–M2) proceeds; a named partner is required before the project claims
beneficiary outcomes beyond "open public good," and `verifiedNeed` stays `false` until one confirms
in writing.

## Goals and non-goals

**Goals.**
- Ship a **correct, deterministic, open reference implementation** of Chudnovsky binary-splitting
  (Method A) and a BBP/Bellard hex extractor (Method B).
- Make donation a **real contribution**: split the series into independent, credit-sized chunks that
  merge map-reduce and reproduce the single-host result **bit-for-bit**.
- Enforce the **HARD RULE** end-to-end: every accepted/published digit is independently verified
  (dual full computation bit-match + BBP spot-check + per-chunk recompute), with an auditable,
  hash-chained ledger and a one-command reproduction.
- Keep everything **open and reproducible**: MIT code, **CC0 / public-domain** digit data (π's
  digits are facts, not copyrightable), cited published algorithms, checksummed artifacts.
- Operate within **hard compute/energy budgets** with per-chunk caps and per-run justification.
- Run distributed compute **only on consenting donors' own machines** via a signed open client —
  never covertly, never on others' resources.

**Non-goals (constraints as identity).**
- **Not a record-at-any-cost chase.** We will not publish a "world record" claim that isn't
  independently reproducible from the public ledger; reproducibility outranks the headline.
- **Never publish unverified digits.** A result that fails any limb of the HARD RULE is not
  published, full stop — even if "probably right."
- **Not a covert distributed-compute scheme / botnet / proof-of-work coin.** No execution of
  donor-supplied code on the coordinator; no running on non-consenting machines; no cryptocurrency.
- **Not a closed or bespoke toolchain.** If it can't be reproduced by an outsider, it isn't done.
- **Not dependent on exotic hardware.** GPU and large-memory paths are *optional* accelerations; the
  reference path runs on commodity machines.
- **No PII / no surveillance.** Worker attribution is pseudonymous and opt-in; we collect no
  personal data and no behavioral telemetry.
- **No medical/legal/safety claims.** Out of domain; the only "advice-like" output is the verified
  mathematical result and its reproducibility evidence.

## Success metrics (outcomes)

Outcome-centric and honest for a moonshot. Baselines are 0 unless noted.

| Outcome | Baseline | Target | How measured |
| --- | --- | --- | --- |
| Published digits that are independently verified (dual computation + BBP) | n/a | **100% (non-negotiable)** | Ledger verification records; any exception is a sev-1 incident |
| Unverified/false results published | 0 | **0 (hard)** | Incident log; release gate audit |
| Reproducibility: published results re-derivable to the same final hash by an independent party | n/a | **100%** | Independent repro from public ledger + repro command |
| Verified precision milestone reached (Method A, distributed-then-merged, bit-matching single host) | 0 digits | A verified milestone at M1 (≥ 1e8 baseline target) and a larger M4 run sized to donated budget | ReleaseManifest + two-method agreement |
| Bit-identical map-reduce: merged chunk result == single-host result | n/a | 100% on every run | Determinism test in CI; merge equality check |
| Independent compute donors contributing verified chunks | 0 | ≥ 10 distinct donors by M2; growth thereafter | Pseudonymous worker records (opt-in) |
| External reuse of toolchain / dataset / protocol | 0 | ≥ 3 documented reuses | Citations, forks-with-use, dataset downloads-with-use |
| Educational artifacts published & used | 0 | ≥ 1 explainer set; ≥ 1 course/learner use | Published docs + adoption note |
| Compute efficiency (throughput on a fixed benchmark) | M0 baseline | Documented ≥ X× improvement by M3 | Benchmark harness, fixed input |
| Energy/compute cost disclosed per verified run | none | Disclosed for every published run | ReleaseManifest energy/compute estimate |

We deliberately avoid vanity metrics (raw digit count alone, GitHub stars). The headline number is
meaningless unless it is **verified and reproducible**, so the integrity metrics outrank the scale
metric.

## Scope

**In scope.**
- Reference Chudnovsky binary-splitting impl producing exact `(P, Q, T)` triples for a term range,
  plus the single-host final assembly (full-precision multiply/divide/sqrt).
- BBP/Bellard hex-digit extractor (standalone coverage **and** independent verification path).
- A **verification harness** enforcing the HARD RULE: dual independent full computation bit-compare,
  BBP/Bellard hex spot-checks, per-chunk recompute-and-hash, known-published-reference cross-check,
  guard-digit analysis.
- **Chunking**: a work-unit manifest and credit-sized term-range chunker with per-chunk compute and
  memory caps; hierarchical `(P, Q, T)` map-reduce merge that reproduces the single-host result.
- **Coordination protocol + append-only, hash-chained digit ledger** (assignment, results, checksums,
  verification status, merge state, releases) and a **reproducibility manifest**.
- A **donated-compute worker client** (consent, resource caps, signed) and a **coordinator service**.
- Algorithm/perf work: NTT/FFT bignum multiplication (a second independent backend), merge
  optimization, Bellard speedups, optional GPU path — all **gated by reproducibility + verification**.
- Open dataset release (CC0) with provenance, repro package, and energy/compute disclosure.
- Educational explainers and a reusable "distributed verified map-reduce" pattern writeup.

**Out of scope (explicit).**
- A record-at-any-cost run that we cannot independently verify or reproduce from the public ledger.
- Running computation on machines whose owners have not explicitly consented (no botnet behavior);
  execution of arbitrary donor-supplied code on the coordinator.
- Closed, proprietary, or unreproducible components anywhere in the critical path.
- Any cryptocurrency / blockchain proof-of-work framing or monetization of donated compute.
- PII collection, behavioral telemetry, or non-pseudonymous worker tracking.
- Hard dependence on a single vendor's hardware/library (the reference path must run on commodity
  CPUs; verification independence requires ≥ 2 distinct bignum backends).
- Mathematical claims outside π (the toolchain is reusable, but in-scope deliverables are π-specific).

## Solution approach & architecture

**Overview.** A library + CLI core (the math + verification), a coordination service + ledger (the
distribution + provenance), and a worker client (donated compute). The math is **exact integer
arithmetic** until the single final assembly step, which is the only place rounding (and hence
guard-digit analysis) enters. Distribution is **map** (independent chunks) → **reduce** (hierarchical
triple merge) → **single-host final assembly** → **verify** → **publish**.

**Components.**
- **Bignum core (dual backend by design).** Arbitrary-precision integer arithmetic with sub-quadratic
  multiplication. **Verification independence is a first-class requirement**, so the project maintains
  **two independent backends** (e.g., a vetted open library such as GMP via bindings *and* a
  clean-room NTT/FFT implementation). A correlated bug in one backend cannot pass the dual-computation
  gate if the second backend is genuinely independent. Backend choice is an ADR (M0).
- **Method A — Chudnovsky binary-splitting engine.** For a term range `[a, b)` computes the exact
  triple `(P, Q, T)` with the standard associative recurrence
  `P(a,b)=P(a,m)·P(m,b)`, `Q(a,b)=Q(a,m)·Q(m,b)`, `T(a,b)=T(a,m)·Q(m,b)+P(a,m)·T(m,b)`. Leaf terms
  follow the Chudnovsky term definition. The number of terms `N ≈ digits / 14.18`.
- **Final assembly (single-host, memory-bound — the known bottleneck).** From the root `(P, Q, T)`,
  `π ≈ (426880·√10005·Q) / T` at full precision: one big division and one big sqrt. This step
  consolidates on one machine and is the scaling limit at the frontier (benchmarked + documented).
- **Method B — BBP/Bellard extractor.** Computes hex digit(s) at offset `k` via modular
  exponentiation, fully independent per chunk. Serves both as standalone coverage and — critically —
  as an **independent-formula** spot-check of Method A's assembled result.
- **Verification harness (enforces the HARD RULE).** (a) Dual full computation: a second independent
  backend/implementation re-derives the result; **bitwise compare** (excluding declared guard
  digits). (b) BBP/Bellard **hex spot-checks** at ≥ 64 random offsets plus the hex tail, all matching
  the assembled binary. (c) **Per-chunk recompute**: each chunk independently recomputed and its hash
  matched before it enters the merge. (d) **Known-reference cross-check** against published reference
  values for overlapping prefixes. (e) **Guard-digit / rounding analysis** documenting the `G`
  trailing digits excluded from publication. A result is "Verified" **only if all limbs pass.**
- **Chunker + work-unit manifest.** Splits `[0, N)` into independent, credit-sized term-range work
  units, each carrying a **per-chunk CPU-time and memory cap** so a single chunk fits one donated
  session and cannot run away.
- **Merge/reduce engine.** Hierarchical, associative combine of accepted `(P, Q, T)` triples into the
  root triple; result must be **bit-identical** to the single-host computation of the same range.
- **Coordination service + ledger.** Assigns work units, collects results + checksums, orchestrates
  verification (dispatching the 2nd-worker recompute), records merge/release state. The **ledger is
  append-only and hash-chained** (each entry references the prior entry's hash) and signed; it is the
  public provenance record from which any result is reproducible.
- **Worker client (donated compute).** A signed, open client the donor runs **on their own machine
  with explicit consent**; fetches a work unit, computes within caps, returns result + checksum +
  environment metadata. Pseudonymous, opt-in attribution; no PII.

**Tech stack.** TypeScript/ESM + pnpm for the coordinator, CLI, ledger, manifests, and
orchestration (consistent with Elyos conventions). The numeric inner loops use the chosen bignum
backends (e.g., GMP bindings) plus a clean-room NTT path; a native/WASM module is acceptable where
TS performance is insufficient, provided it is open and deterministic. Hashing: SHA-256 (primary) +
a second independent hash for artifacts. Testing: Vitest (unit), a determinism/repro test
(same input → same hash across both backends), and a verification-gate test.

**Data model (core records).**
```
WorkUnit {
  id; method: "chudnovsky-bs" | "bbp" | "bellard";
  range: { aTerm, bTerm }            // Method A: term range [a,b)
        | { hexOffset, hexCount };   // Method B: hex digits at offset
  targetPrecisionDigits; algoVersion; bignumBackend;
  computeCapCpuSeconds; memCapBytes; createdAt;          // hard per-chunk caps
}
ChunkResult {
  workUnitId; workerId(pseudonymous);
  payloadRef; payloadHashSha256; payloadHashAlt;         // (P,Q,T) artifact | hex digits
  bignumBackend; algoVersion; env { os, arch, libVersions };
  wallTimeMs; status: "submitted" | "verified" | "rejected";
}
VerificationRecord {
  workUnitId; primaryHash; secondaryHash; secondaryBackend; secondaryWorkerId;
  bitIdentical: bool;
  bbpSpotChecks: [{ hexOffset, expected, got, pass }];
  knownReferenceMatch: bool; guardDigitsDropped: int;
  verifiedAt; verifierId; result: "verified" | "failed";
}
LedgerEntry (append-only, hash-chained) {
  seq; prevEntryHash; type: "assign"|"result"|"verify"|"merge"|"release";
  refs[]; entryHash; signer; timestamp;                  // NO secrets, ever
}
ReleaseManifest {
  runId; methods[]; base; totalDigitsPublished; guardDigitsDropped;
  finalResultHash; sqrtConstHash; twoMethodAgreement: bool;
  bbpSpotCheckCount; bbpSpotCheckAllPass: bool;
  algoVersions[]; backendVersions[]; reproCommand;
  computeCpuHours; energyEstimateKwh; verifierSignatures[];
  license: "CC0-1.0";
}
```

**Key decisions (ADRs, recorded in M0).**
1. **Bignum backend strategy** — which two *independent* backends (one vetted library + one
   clean-room NTT) and the FFI/WASM boundary; verification independence is the deciding criterion.
2. **`(P, Q, T)` artifact format** — serialization + hashing of triples for transport and ledgering.
3. **Coordination protocol + ledger schema** — work-unit lifecycle, hash-chaining, signing.
4. **Guard-digit policy** — how `G` is chosen and how published precision excludes unstable digits.
5. **Determinism contract** — what "bit-identical" means across backends/platforms (e.g., exact
   integers identical always; final-assembly rounding pinned by the guard-digit policy).

**Decision ordering (important).** The **bignum-backend ADR (#1) and guard-digit policy (#4) are
decided before** the verification harness is finalized — the harness's "bit-identical" contract and
spot-check sufficiency depend on both. TASKS.md sequences this (verification harness depends on the
ADR task).

## Data, licensing & compliance

**This section is load-bearing. Be conservative.**

**Sources & inputs.** The "data" here is mathematics, not a third-party dataset:
- **The digits of π are facts** — not copyrightable and not owned by anyone. Published reference
  values used for cross-checking (e.g., widely-circulated reference digits) are facts used only to
  *verify*, and our published digits are an **original computation**, released into the public domain.
- **Algorithms are published and unencumbered.** Chudnovsky (1988), the BBP formula (Bailey–
  Borwein–Plouffe, 1995), and Bellard's formula are openly published. **We will verify no patent
  encumbers the specific methods/optimizations we use** before relying on them, and we cite the
  primary papers in the provenance record. No proprietary or NDA-bound algorithm enters the codebase.
- **Bignum libraries** must be **open-licensed and license-compatible** (e.g., GMP is LGPL — usable
  via dynamic linking/bindings; we document the linking stance). The clean-room NTT backend is our
  own MIT code, guaranteeing at least one fully permissive path.

**Licensing rigor (critical).**
- **Our code:** **MIT.**
- **Our digit data, ledgers, manifests, datasets:** **CC0-1.0 / public-domain dedication** (facts).
- **Educational content / writeups:** **CC-BY-4.0.**
- **Third-party libraries:** each dependency's license recorded and checked for compatibility; LGPL
  components used only via the boundary their license permits; **no GPL code linked into MIT outputs**
  in a way that would relicense them. A dependency-license audit runs in CI.

**Provenance model.** Every artifact is **content-addressed** (SHA-256 + a second hash). Every
result carries its `algoVersion`, `bignumBackend`/version, environment, and the **repro command** that
regenerates it. The hash-chained ledger ties assignment → result → verification → merge → release into
an auditable chain from which any published digit is reproducible. A `ReleaseManifest` is the
human-readable head of that chain.

**Privacy / PII stance.** **Zero PII.** Workers are identified only by an opt-in pseudonymous id; no
account, email, IP-logging-for-tracking, or behavioral telemetry is required to contribute. The only
metadata collected is technical (OS/arch/library versions, wall time) and is used for reproducibility
and credit, not profiling. **No secrets, tokens, or keys** are ever written to ledgers, manifests,
logs, or artifacts (per CLAUDE.md).

**Attribution.** Algorithm authors and any reference sources are cited in provenance. Contributing
donors are credited (pseudonymously, opt-in) in the ledger. We do **not** imply endorsement by any
person or project we cite.

## Quality, review & risk gates

**Risk tier: medium.** Domain-accuracy + methodology review is mandatory; an unverified or false
digit claim, or a wasteful/abusive compute run, can do real harm to trust and to donors.

**Required reviews before a deed is "done":**
- **Code tasks (math + infra):** maintainer code review + CI green (build, test, lint, **determinism/
  repro test**, **verification-gate test**, dependency-license audit, no-secrets audit). A change that
  breaks bit-reproducibility or weakens the verification gate **cannot merge**, regardless of speed
  gains.
- **Computation-run tasks:** **no result enters the ledger or is published unless the HARD RULE
  passes** — (1) two independent full computations bit-identical (different backend/impl, ideally
  different operator), (2) BBP/Bellard independent-path hex spot-checks all pass, (3) every
  constituent chunk recompute-verified and hashed, (4) known-reference prefix match, (5) guard-digit
  analysis recorded. The **verification steward** (independent of whoever ran the computation) signs
  off; an author may **not** verify their own run.
- **Expert review (medium-risk gate):** a **computational-mathematics / numerical-analysis SME**
  reviews and signs off the **verification methodology** — guard-digit derivation, BBP spot-check
  count sufficiency (the probability bound on an undetected error region), and the genuineness of
  two-method/two-backend independence — **before any record-scale run is published** (M4).
- **Determinism & reproducibility:** every release must be re-derivable to the same `finalResultHash`
  by an independent party using only the public ledger + repro command.

**Definition of Shipped (project-level).** A **verified, reproducible π computation** that:
(1) was produced by independent, credit-sized chunks merged map-reduce to a **bit-identical**
single-host result; (2) passes the full **HARD RULE** (dual computation + BBP spot-check + per-chunk
recompute + reference cross-check + guard-digit analysis); (3) is published as an **open CC0 dataset
+ ReleaseManifest + one-command reproduction**, with compute/energy cost disclosed; (4) was produced
on **consenting donated compute** within per-chunk and per-run caps; and (5) the verification
methodology carries **SME sign-off**. Until a partner is secured, criterion-set (1)–(5) defines a
recognized **"Publicly Shipped (open public good)"** state; a later partner endorsement upgrades the
status rather than gating launch.

## Roadmap & milestones

Phased; each phase has measurable exit criteria. M0 is a thin, end-to-end cold-start spine.

- **M0 — Spine: reference impl + verification harness + coordination protocol (thin slice).**
  Goal: a correct, *verified*, end-to-end vertical slice at small scale (distribution may be simulated
  locally), with the HARD RULE wired in from the first commit.
  Exit: π computed to ≥ 1e6 digits by Method A on a single host; **independently re-derived by a 2nd
  bignum backend (bit-identical)**; BBP/Bellard hex spot-checks pass; matched against a published
  reference prefix; ledger schema + repro manifest record the run with hashes; ADRs #1–#5 recorded;
  CI runs build/test/lint + determinism test + verification-gate test + dependency-license + no-secrets
  audits. Everything MIT/CC0.

- **M1 — Chunking & map-reduce merge (donation becomes a real contribution).**
  Goal: split the series into independent, credit-sized term-range chunks; compute separately; merge
  `(P, Q, T)` hierarchically to reproduce the single-host result **bit-for-bit**.
  Exit: a target precision (baseline ≥ 1e8 digits) computed via ≥ K independent chunks merged
  map-reduce; **merged == single-host (bit-identical)**; every chunk recompute-verified + hashed +
  ledgered; chunk size tuned to a stated credit-sized CPU/memory budget; the single-host final-assembly
  bottleneck benchmarked + documented.

- **M2 — Distributed coordination + donated-compute worker client.**
  Goal: a real (not simulated) coordinator + signed worker client so independent donors run chunks on
  their **own** machines with consent + caps; public hash-chained ledger; abuse guardrails.
  Exit: ≥ 10 distinct external donors complete + verify chunks through the live coordinator; every
  accepted chunk has a 2nd-worker recompute match; ledger is public, append-only, hash-chained, and
  reproducible; the anti-abuse threat model (consent, caps, signed client, never-trust-one-worker,
  no covert/non-consenting use) is implemented and documented.

- **M3 — Algorithm/perf hardening (NTT/FFT, GPU optional, Bellard speedups).**
  Goal: the algorithm-work family — faster bignum multiplication (NTT/FFT as the second independent
  backend), better merge, optional GPU path, Bellard optimization — **without ever weakening the
  verification gate**.
  Exit: documented ≥ X× throughput on a fixed benchmark; ≥ 2 genuinely independent bignum backends
  maintained; every perf change passes the determinism + verification tests (no perf change ships if
  it breaks bit-reproducibility); SME review of the verification methodology completed.

- **M4 — Verified record-scale run + open dataset release.**
  Goal: an extended-precision run beyond the M1 baseline (target sized to justified donated budget),
  fully HARD-RULE verified, published as an open dataset + reproducibility package, with the
  final-assembly bottleneck addressed or honestly scoped to the largest feasible target.
  Exit: a new **verified** digit milestone with two-method agreement + BBP spot-checks + per-chunk
  verification; **SME-signed verification methodology**; CC0 dataset + ReleaseManifest + repro package
  published; compute/energy cost disclosed; **zero unverified digits published.**

- **M5 — Research/education outputs + partner adoption (ongoing).**
  Goal: convert the verified toolchain into reusable public good — explainers (binary splitting, BBP,
  NTT), a normality/statistics-ready dataset, and the reusable distributed-verified-map-reduce pattern
  — and secure a research/education partner.
  Exit: ≥ 1 educational artifact published + used; ≥ 1 external reuse of the toolchain/dataset/protocol;
  **named research/education partner — TO BE SECURED** (on success, `verifiedNeed` flips to `true`).

Dependencies: M1 depends on M0 (verified single-host baseline + ledger). M2 depends on M1 (working
merge + per-chunk verification). M3 runs partly in parallel with M2 but its perf changes are gated by
M0/M1 reproducibility tests. M4 depends on M2 (live distribution) + M3 (throughput) + SME sign-off.
M5 depends on M4 (a verified run + dataset to teach from) and on the partner (parallel, non-blocking).

## Work breakdown

The itemized, schema-mapped backlog lives in **TASKS.md**, organized by the M0–M5 milestones above.
Each task maps to an Elyos Task JSON (see schema), is sized (small/medium/large), risk-tagged, and
names a reviewer. TASKS.md also includes acceptance criteria for the most important tasks per
milestone, milestone Definitions of Done, a backlog, and a complete, schema-valid example Task JSON.

## Governance, roles & stakeholders

- **Maintainer (Owner): TBD.** Owns the repo, roadmap, releases, ADRs, and review standards.
- **Algorithm reviewers:** contributors competent in numerical methods / arbitrary-precision
  arithmetic; review correctness of Method A/B and merge.
- **Verification steward: TBD** — owns the HARD RULE gate, **independent of whoever ran a
  computation**; an author may never verify their own run. Signs the `VerificationRecord`.
- **Expert reviewer (medium-risk sign-off): TO BE SECURED** — a computational-mathematics /
  numerical-analysis SME who signs off the **verification methodology** before any record-scale
  publication (guard-digit analysis, spot-check sufficiency, backend independence).
- **Compute-donor coordinator: TBD** — supports donors, manages the signed worker-client releases,
  and the consent/abuse posture.
- **Steward (last-mile owner): TBD** — owns dataset/release distribution and the partner relationship.
- **Partner / requestor: TO BE SECURED** — a research/education or distributed-computing org
  confirming need and ultimately adopting/reusing the toolchain or dataset.
- **Elyos governance/board:** arbitrates edge cases, risk-tier and compute-budget decisions per the
  good-deed definition.

## Dependencies & integrations

- **Algorithms (published, cited):** Chudnovsky series; BBP formula; Bellard's formula; binary
  splitting; NTT/FFT multiplication.
- **Bignum backends:** a vetted open library (e.g., GMP, LGPL — via bindings/WASM) **plus** a
  clean-room MIT NTT implementation (verification independence).
- **Reference values:** publicly available π reference digits, used **only** to cross-check prefixes.
- **Tooling/libraries:** TypeScript/ESM, pnpm, Vitest; SHA-256 + a second hash; signing for the
  ledger and worker-client releases; optional native/WASM numeric module; optional GPU runtime (M3).
- **Hosting:** a coordinator service (small footprint) + static publication of the ledger/dataset.
- **Elyos pieces:** Task schema (`packages/schema`), CLI workspace prep / PR flow (donated lane),
  good-deed definition & risk-tier governance, review/sign-off process.
- **Human/expert dependency:** algorithm reviewers, the verification steward, and a numerical-analysis
  SME — the gating non-software dependencies for medium-risk sign-off.
- **Compute dependency:** donated CPU/GPU cycles — a *distinct* resource from the Elyos agent lanes
  (see *Open questions* on the schema gap).

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
| --- | --- | --- | --- | --- |
| Publishing unverified / false digits | Low | High | HARD RULE: dual independent computation bit-match + BBP spot-check + per-chunk recompute + reference cross-check; verification steward independent of the runner; any breach is a sev-1 | Verification steward |
| Correlated bug in a single bignum backend passes verification | Medium | High | Two **genuinely independent** backends (vetted library + clean-room NTT); BBP/Bellard is an **independent formula**, not just a re-run; known-reference cross-check | Algorithm reviewer |
| Wasted compute / energy ("compute for compute's sake") | Medium | Medium | Per-chunk CPU/memory caps; per-run total budget + justification + energy disclosure; no duplicate assignment beyond the required verification copy | Maintainer |
| Coordinator misused as a botnet / runs on non-consenting machines | Low | High | Signed open worker client run by the donor on their own machine with explicit consent; no execution of donor-supplied code on the coordinator; anti-abuse threat model (M2); kill switch | Compute-donor coordinator |
| Malicious worker returns wrong/forged results | Medium | High | Never trust one worker: mandatory 2nd-worker recompute + hash match + spot-check before acceptance; pseudonymous reputation; reject on mismatch | Verification steward |
| Single-host final-assembly bottleneck blocks record scale | High | Medium | Benchmark + document early (M1); scope targets to feasible memory; treat frontier consolidation as an explicit, separate engineering problem, not a hidden assumption | Algorithm reviewer |
| Guard-digit / rounding error in final assembly | Medium | High | Documented guard-digit policy; publish precision minus `G`; SME review of the analysis | Algorithm reviewer + SME |
| Perf optimization breaks bit-reproducibility | Medium | High | Determinism + verification tests gate every perf PR; no perf change merges if reproducibility breaks | Maintainer |
| Algorithm/library patent or license incompatibility | Low | High | Verify methods are unencumbered + cite primary papers; dependency-license audit in CI; clean-room MIT NTT guarantees a permissive path | Maintainer |
| No partner / verified need secured | Medium | Medium | Pursue research/education partner in parallel; "Publicly Shipped (open public good)" success state so a finished toolchain isn't stranded; later endorsement upgrades status | Steward |
| Over-claiming a "world record" reputationally | Medium | Medium | Reproducibility outranks headline; publish ledger + repro command; no claim that an outsider can't re-derive | Maintainer |
| Maintainer bandwidth / bus factor | Medium | Medium | Reviewer rotation, documented ADRs/processes, MIT/CC0 lowers lock-in | Maintainer |

## Security & privacy

**Threat surface.** (1) **Untrusted workers** submitting wrong/forged results; (2) the **coordinator
as an abuse vector** (covert use, running on non-consenting machines, DoS, poisoned work units);
(3) **supply-chain** risk in numeric/native dependencies; (4) **ledger tampering**; (5) **secret
leakage** into artifacts/logs.

**Controls.**
- **Never trust a single worker.** No result is accepted without an **independent 2nd-worker
  recompute + hash match + BBP spot-check**. The math itself is the integrity check.
- **Consent-first, no covert compute.** Donors run a **signed, open worker client on their own
  machine** with explicit, informed consent and an obvious stop/kill control. The coordinator
  **never executes donor-supplied code** and never instructs a machine its owner didn't enroll. This
  is an explicit anti-botnet posture, documented in the M2 threat model.
- **Per-chunk + per-run caps.** Hard CPU-time and memory caps per work unit; a per-run total compute
  budget; a chunk that exceeds its cap aborts cleanly. Prevents runaway resource use on donor machines.
- **Tamper-evident provenance.** The ledger is **append-only and hash-chained** (each entry binds the
  prior entry's hash) and signed; releases carry a `ReleaseManifest` with `finalResultHash` and a
  repro command, so tampering is detectable and any result is independently re-derivable.
- **No secrets, ever.** No API keys/tokens/credentials in code, logs, ledgers, manifests, or
  artifacts (per CLAUDE.md). The coordinator's signing keys are held in a secret manager, never
  committed; the public ledger contains only hashes and pseudonymous ids.
- **No PII.** Pseudonymous, opt-in worker attribution; no behavioral telemetry; only technical repro
  metadata. (See *Data, licensing & compliance*.)
- **Supply chain:** pinned/locked dependencies (pnpm lockfile), minimal deps, dependency + license
  audit in CI, signed worker-client releases with published checksums; Subresource/binary integrity
  where applicable.
- **Funded-lane caps (if used).** If any *algorithm-work* task uses metered API (e.g., agent-assisted
  optimization), it runs only via `packages/runner` with a hard `fundedBudgetUsd` cap that must never
  be exceeded (per CLAUDE.md). **Computation runs are donated CPU/GPU, not API spend.**

## Sustainability & maintenance

- **Ownership after delivery:** maintainer owns code/releases; verification steward owns the gate;
  steward owns dataset distribution + the partner relationship; compute-donor coordinator supports
  donors. All currently **TBD** and must be named before M2 (steward/coordinator) and M4 (SME).
- **Reproducibility as durability:** because every release is reproducible from a public ledger +
  one-command repro, the project survives loss of any single operator or machine — anyone can re-derive
  and re-verify.
- **Toolchain reuse:** the bignum core, binary-splitting engine, verification harness, and
  coordination protocol are designed to be **reusable beyond π** (any binary-splitting series, any
  distributed-verified map-reduce), which is the main long-term public good.
- **Outcomes tracking:** verification integrity and reproducibility are tracked in-repo (ledger +
  CI); reuse and educational adoption are tracked by best-effort self-report (no telemetry).
- **Low lock-in:** MIT/CC0, commodity-hardware reference path, standard tooling — cheap to run and
  forkable, improving long-term survivability.

## Open questions

1. **Compute-donation lane gap (needs a human/governance decision).** Elyos lanes model **agent
   compute** (donated agent sessions / funded API with `fundedBudgetUsd`). bigger-pi's
   **computation-run family donates raw CPU/GPU cycles**, which the current Task schema doesn't model
   (`tokenEstimate` ∈ {small,medium,large} doesn't capture CPU-hours, and there's no compute-budget
   cap field). **Proposal:** treat computation-run tasks as `data`/`dataset` tasks, document the
   per-chunk compute/memory cap in `context` for now, and **extend the schema** with an optional
   `computeBudgetCpuHours` (analogous to `fundedBudgetUsd`). Decision needed from governance.
2. **Record-scale target & budget.** What total compute/energy budget (and therefore digit target) is
   justified for M4? This must be approved against the energy-cost disclosure, not chased open-ended.
3. **Final-assembly consolidation host.** Who provides the large-memory single host for the final
   multiply/divide/sqrt at scale? This is the hard bottleneck; the frontier target depends on it.
4. **Bignum backend choices (ADR #1).** Which vetted library + the scope of the clean-room NTT;
   LGPL-linking stance documented.
5. **Verification SME.** Who is the numerical-analysis SME for the medium-risk methodology sign-off?
6. **Partner org.** Which research/education or distributed-computing partner, if any?
7. **GPU scope.** Is a GPU path in-scope for M3, or backlog? (Verification gate is unchanged either
   way.)

## References

- Elyos work rules — `C:\code\elyos\CLAUDE.md`
- Good-deed definition & risk tiers — `C:\code\elyos\docs\good-deed-definition.md`
- Task schema — `C:\code\elyos\packages\schema\src\schemas.ts`
- Portfolio roadmap (bigger-pi entry) — `C:\code\elyos\planning\ROADMAP.md`
- Algorithms (cite primary papers in provenance): Chudnovsky brothers (1988/1989), Chudnovsky series;
  Bailey, Borwein & Plouffe (1995/1997), the BBP formula; F. Bellard, Bellard's formula; binary
  splitting for hypergeometric series; Schönhage–Strassen / NTT-based multiplication.
- Prior art for distributed digit extraction: the PiHex project (distributed BBP hex-digit computation).

---

## Appendix A — Improvements applied

The following 25 specific improvements were applied to the draft above (each notes where).

1. **Reframed the public good away from the digits themselves** to the open, verified, reproducible
   *toolchain + protocol + ledger* (Exec summary, Problem) — honest about a moonshot's real value.
2. **Made the HARD RULE the explicit identity** with a precise, multi-limb definition (dual
   computation + BBP + per-chunk recompute + reference cross-check + guard digits) rather than a
   vague "verify" (Exec summary, Quality gates).
3. **Required two *genuinely independent* bignum backends** (vetted library + clean-room NTT) so a
   correlated library bug cannot pass verification (Architecture, Risks).
4. **Distinguished BBP as an *independent formula* check**, not merely a re-run, strengthening the
   cross-method guarantee (Architecture, Risks).
5. **Surfaced the single-host final-assembly bottleneck as a named, benchmarked constraint** with its
   own M1 task and risk row — not buried (Architecture, Roadmap, Risks).
6. **Added a guard-digit / rounding policy** (publish precision minus `G`) as an ADR + SME-reviewed
   item, since final assembly is the only rounding step (Architecture, Quality gates, Risks).
7. **Added per-chunk CPU/memory caps and per-run compute budgets + energy disclosure** to honor the
   moonshot "budget/compute caps" guardrail (Goals, Architecture, Security, Risks).
8. **Wrote an explicit anti-botnet / consent posture** (signed client, donor's own machine, no
   coordinator-executed code) — the key abuse vector for distributed compute (Non-goals, Security, M2).
9. **Adopted "never trust a single worker"** as a security principle backed by mandatory 2nd-worker
   recompute + hash + spot-check (Security, Risks).
10. **Hash-chained, append-only, signed ledger** for tamper-evidence and full reproducibility
    (Architecture, Data model, Security).
11. **Content-addressed artifacts with two hashes + a one-command repro** so any published digit is
    independently re-derivable (Data/licensing, ReleaseManifest, Quality gates).
12. **Pinned licensing precisely**: MIT code, **CC0** for digit data (facts, not copyrightable),
    CC-BY for explainers, with an LGPL-linking stance for GMP and a clean-room MIT NTT fallback
    (Data, licensing & compliance).
13. **Added a patent/encumbrance check** for the algorithms before reliance, with primary-paper
    citations in provenance (Data, Risks).
14. **Flagged the Elyos schema gap** (agent-lane model vs. donated *CPU* compute) as a concrete
    governance decision with a proposed `computeBudgetCpuHours` extension (Open questions, Dependencies).
15. **Separated the verification steward role from the runner** ("author may not verify their own
    run") mirroring the no-self-approval pattern from sibling Elyos plans (Governance, Quality gates).
16. **Required a numerical-analysis SME sign-off on the *methodology*** (spot-check sufficiency,
    independence, guard digits) as the medium-risk gate before any record-scale publish (Quality, M4).
17. **Made success metrics integrity-first** (100% verified, 0 unverified published, 100%
    reproducible) and explicitly *demoted* raw digit count as a vanity metric (Success metrics).
18. **Defined a concrete BBP spot-check target** (≥ 64 random offsets + hex tail) tied to a stated
    probability bound, instead of "some spot checks" (Architecture, Quality gates).
19. **Specified the map-reduce equality gate** — merged chunk result must be *bit-identical* to the
    single-host computation — as an explicit, testable exit criterion (Roadmap M1, Success metrics).
20. **Added a "Publicly Shipped (open public good)" success state** with partner TO BE SECURED, so a
    finished, verified toolchain isn't stranded for lack of a partner (Quality gates, Roadmap M5).
21. **Encoded the alternating task-family loop** (algorithm-work ↔ computation-runs) into the roadmap
    phasing and the TASKS milestone structure (Exec summary, Roadmap, TASKS).
22. **Added a determinism contract ADR** defining what "bit-identical" means across backends/platforms
    so reproducibility is unambiguous (Architecture key decisions).
23. **Gated all perf work by reproducibility + verification** — no speed change ships if it breaks
    bit-reproducibility — preventing optimization from eroding trust (Roadmap M3, Risks).
24. **Made worker attribution pseudonymous + opt-in with zero PII / zero behavioral telemetry**, and
    barred secrets from ledgers/logs/artifacts per CLAUDE.md (Data, Security).
25. **Added decision ordering** (backend + guard-digit ADRs precede the verification harness;
    chunker/merge precede distribution) so dependencies are explicit and sequencing is realistic
    (Architecture decision ordering, Roadmap dependencies, TASKS depends-on).

## Review sign-off

**Reviewer:** senior staff engineer + TPM (self-review against PLAN_SPEC.md, CLAUDE.md, and the
good-deed definition). **Date:** 2026-06-28.

**Completeness.** All 17 required H2 sections present and in order. Metadata header matches the
global convention. TASKS.md exists with schema mapping, per-milestone tables, acceptance criteria,
DoD, backlog, and a schema-valid example Task JSON. Appendix A lists 25 applied improvements.

**Correctness checks performed & fixes made.**
- **Schema validity** of the example task confirmed field-by-field against
  `packages/schema/src/schemas.ts` (all required fields present; enums valid; donated lane so no
  `fundedBudgetUsd` needed; `verifiedNeed=false`, `requestor="TBD"`).
- **Math sanity:** binary-splitting recurrence and `N ≈ digits/14.18` stated correctly; final
  assembly `π ≈ (426880·√10005·Q)/T` identified as the single-host rounding step; BBP positioned as
  an *independent-formula* check (hex), not a re-run — fixed an earlier draft phrasing that implied a
  mere recomputation.
- **Guardrails:** no for-profit primary benefit; open/PD/CC-only licensing (CC0 for digit facts);
  zero PII; compute/energy caps + verification gates emphasized per the moonshot guardrails;
  anti-botnet/consent posture explicit; medium-risk SME methodology sign-off required before any
  record-scale publish.
- **Honesty:** partner and verified need marked **TO BE SECURED** / `verifiedNeed=false`; the
  Elyos-lane-vs-CPU-donation **schema gap** surfaced as an explicit governance decision rather than
  glossed; record target gated behind a justified, disclosed budget.

**Outstanding items requiring a human decision (carried to Open questions):** the compute-donation
schema extension; the M4 compute/energy budget + digit target; the final-assembly consolidation host;
the verification SME; and the partner org. None block M0–M1.

**Verdict:** Approved as a Draft (v0.1.0) baseline for review. The verification gate and
reproducibility posture are strong enough to begin M0; the named-human dependencies (steward, SME,
partner) and the compute-lane schema decision are the gating items before scale.
