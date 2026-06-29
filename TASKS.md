# TASKS — bigger-pi (distributed, verified, open computation of more digits of π)

> Status: Draft · Version: 0.2.0 · Last updated: 2026-06-29 · Owner: TBD (maintainer) · Lane: donated

## How these tasks map to Elyos

Each task below becomes an Elyos **Task JSON** validated against
`packages/schema/src/schemas.ts`. Field mapping:

- `id` — stable slug ID, e.g. `bigger-pi-core-002`.
- `title` — the task title from the table.
- `project` — `"bigger-pi"`.
- `type` — one of `code | research | writing | data | design-spec | maintenance` (the "Type" column).
- `lane` — `"donated"` for all tasks here (no escrow/API spend). A `funded` task (e.g. agent-assisted
  optimization via `packages/runner`) would require `fundedBudgetUsd` with a hard cap.
- `priority` — `high | medium | low`.
- `domain` — array, e.g. `["mathematics","distributed-computing","software","open-data"]`.
- `riskTier` — `low | medium | high` (the "Risk" column). Verification/computation-run and
  coordination/abuse tasks are **medium**; pure infra/teaching tasks are **low**.
- `urgent` — boolean (default `false`; this is a moonshot, not live response).
- `deliverable` — `pr | dataset | document | translation` (the "Deliverable" column).
- `tokenEstimate` — `small | medium | large` (the "Size" column). **Note:** for computation-run tasks
  this estimates the *agent/coordination* effort, **not** the donated CPU-hours — those are capped
  per-chunk in `context` (see PLAN *Open questions* on the proposed `computeBudgetCpuHours` field).
- `status` — `open | in-progress | review | delivered | done` (all start `open`).
- `context`, `objective`, `acceptanceCriteria[]`, `output` — task narrative + checkable criteria.
- `resources[]` — source/reference URLs or repo paths.
- `requestor` — partner/requestor (**TO BE SECURED**; `"TBD"` until a partner is named).
- `verifiedNeed` — **`false` until a named partner confirms need in writing** (honest default).
- `outputLicense` — `"MIT"` for code, `"CC0-1.0"` for digit data/datasets/ledgers,
  `"CC-BY-4.0"` for educational writeups.

**Reviewer column legend:** `maintainer` (code review), `algo` (numerical-methods/arbitrary-precision
reviewer), `verify-steward` (owns the HARD RULE gate; **independent of whoever ran the computation**),
`SME` (computational-mathematics / numerical-analysis expert; medium-risk methodology sign-off),
`steward` (last-mile dataset/partner owner).

**HARD RULE (binding on all computation-run + release tasks):** no digit result is accepted into the
ledger or published unless the **standard gate** passes: **(1)** one main Chudnovsky computation,
**(2)** independent-path **BBP/Bellard hex spot-checks** pass (≥ 64 random offsets + hex tail — note
these are *base-16, position-specific* checks, not a base-10 re-derivation), **(3)** every constituent
chunk was recompute-verified + hashed, **(4)** the published prefix matches a known reference, and
**(5)** the guard-digit analysis is recorded. **Dual *full* independent computation is reserved** for
small reference baselines or where BBP coverage is insufficient — it is *not* the default, because two
full computations per result double CPU-hours/energy (no-waste guardrail). The author of a run may
**not** verify their own run; `verify-steward` signs the `VerificationRecord`.

**Sequencing rule (M0):** the **bignum-backend ADR and guard-digit policy (arch-001) are decided
before** the verification harness (verify-004) is finalized — the harness's "bit-identical" contract
depends on both. The chunker (M1) depends on the verified single-host baseline (core-002) + ledger
schema (ledger-005).

---

## Milestone M0 — Spine: reference impl + verification harness + coordination protocol

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| bigger-pi-arch-001 | ADRs: dual bignum-backend strategy, (P,Q,T) artifact format, coordination+ledger schema, **guard-digit policy**, determinism contract | design-spec | small | low | document | — | maintainer, algo, verify-steward |
| bigger-pi-core-002 | Chudnovsky binary-splitting reference impl: (P,Q,T) per term-range + single-host final assembly to ≥ 1e6 digits | code | large | low | pr | arch-001 | maintainer, algo |
| bigger-pi-core-003 | BBP/Bellard hex-digit extractor (independent code path) | code | medium | low | pr | arch-001 | maintainer, algo |
| bigger-pi-verify-004 | Verification harness enforcing the HARD RULE: dual-backend bit-compare + BBP spot-check + known-reference check + guard-digit report | code | large | medium | pr | core-002, core-003 | verify-steward, algo |
| bigger-pi-ledger-005 | Coordination protocol + append-only **hash-chained** digit ledger + reproducibility manifest schema | design-spec | medium | low | document | arch-001 | maintainer, verify-steward |
| bigger-pi-ci-006 | CI: build/test/lint + **determinism test** (same input → same hash across 2 backends) + verification-gate test + dependency-license + no-secrets audit | code | medium | low | pr | core-002 | maintainer |

**Acceptance criteria — key tasks**

- **bigger-pi-core-002 (Chudnovsky binary-splitting reference impl):**
  - Computes exact integer `(P, Q, T)` for any term-range `[a, b)` using the associative recurrence;
    leaf terms follow the Chudnovsky definition; `N ≈ digits / 14.18` documented.
  - Single-host final assembly `π ≈ (426880·√10005·Q)/T` produces ≥ 1e6 correct digits.
  - Result **matches a published reference prefix** to the computed precision (minus declared guard
    digits).
  - Deterministic: identical input → identical artifact hash on repeat runs.
  - TypeScript/ESM, builds via pnpm; `pnpm build && pnpm test && pnpm lint` pass; MIT-licensed.
- **bigger-pi-verify-004 (verification harness — HARD RULE):**
  - **Standard gate:** runs ≥ 64 random BBP/Bellard **hex spot-checks** + the hex tail (an *independent
    formula*, base-16) against the assembled value; all must pass; the residual-error probability bound
    is documented. This is the default — a single main computation, not two.
  - **Reserved dual-full check:** for small reference baselines (or where BBP coverage is insufficient),
    re-derives the result with a **second independent bignum backend** and asserts **bit-identical**
    (excluding the declared `G` guard digits), failing loudly on mismatch — invoked deliberately, not by
    default (it doubles energy).
  - Cross-checks the published prefix against a known reference value.
  - Emits a `VerificationRecord` (hashes, backends, spot-check results, guard digits, verifier id);
    refuses to mark "verified" unless **every** limb passes.
  - An author cannot sign off their own run (role-separation enforced in process + record fields).
- **bigger-pi-ledger-005 (coordination protocol + ledger):**
  - Defines `WorkUnit`, `ChunkResult`, `VerificationRecord`, `LedgerEntry`, `ReleaseManifest` (per
    PLAN data model); ledger entries are **append-only and hash-chained** (each references the prior
    entry's hash) and signed.
  - A `ReleaseManifest` carries `finalResultHash`, `twoMethodAgreement`, spot-check counts, repro
    command, and compute/energy fields; no secrets appear anywhere in the schema.
  - Includes a documented one-command reproduction path from ledger → final hash.
- **bigger-pi-ci-006 (CI gates):**
  - CI fails on build/type/lint/unit errors, on any **determinism** failure (the reference path —
    **integer / fixed-modulus NTT, not floating FFT** — must produce identical hashes across backends/
    architectures), on any **verification-gate** test failure, on a **license-incompatible
    dependency**, and on any **secret** detected in code/artifacts/ledger fixtures.

**Definition of Done (M0):** π computed to ≥ 1e6 digits by Method A on a single host, **independently
re-derived bit-identical by a 2nd backend**, BBP-spot-checked, and matched to a published reference;
ADRs #1–#5 recorded (backend + guard-digit policy first); ledger + repro-manifest schema defined; CI
enforces build/test/lint + determinism + verification-gate + license + no-secrets. All MIT/CC0.

---

## Milestone M1 — Chunking & map-reduce merge

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| bigger-pi-chunk-007 | Work-unit manifest + credit-sized term-range chunker with per-chunk CPU/memory caps | code | medium | low | pr | core-002, ledger-005 | maintainer, algo |
| bigger-pi-merge-008 | Hierarchical (P,Q,T) map-reduce merge; merged result **bit-identical** to single-host | code | large | medium | pr | chunk-007 | algo, maintainer |
| bigger-pi-verify-009 | Per-chunk recompute verification (2nd worker/backend hash match) wired into ledger acceptance | code | medium | medium | pr | verify-004, merge-008 | verify-steward |
| bigger-pi-bench-010 | Benchmark + document the single-host final-assembly bottleneck (memory/time at scale) | research | small | low | document | merge-008 | algo |

**Acceptance criteria — key tasks**

- **bigger-pi-chunk-007 (chunker + manifest):**
  - Splits `[0, N)` into independent term-range work units, each with a **hard CPU-time + memory cap**
    sized to one donated session ("credit-sized"); a unit exceeding its cap aborts cleanly.
  - Emits a work-unit manifest (id, method, range, caps, algoVersion, backend) that the coordinator
    and ledger consume.
- **bigger-pi-merge-008 (map-reduce merge):**
  - Hierarchically merges accepted `(P, Q, T)` triples using the associative combine; the merged root
    triple and assembled result are **bit-identical** to a single-host computation of the same range
    (asserted in CI).
  - Merge is order-independent for a fixed partition (associativity verified by test).
- **bigger-pi-verify-009 (per-chunk verification):**
  - Each chunk is recomputed by a **second independent worker/backend**; acceptance into the merge
    requires a **hash match**; a mismatch rejects the chunk and never enters the ledger as verified.
  - Every accepted chunk has a `VerificationRecord` linked in the hash-chained ledger.

**Definition of Done (M1):** a target precision (baseline ≥ 1e8 digits) computed via ≥ K independent,
credit-sized chunks merged map-reduce; **merged == single-host (bit-identical)**; every chunk
recompute-verified + hashed + ledgered; chunk size tuned to a stated CPU/memory budget; the
final-assembly bottleneck benchmarked and documented.

---

## Milestone M2 — Distributed coordination + donated-compute worker client

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| bigger-pi-coord-011 | Coordinator service: assignment, collection, verification orchestration, public hash-chained ledger | code | large | medium | pr | ledger-005, verify-009 | maintainer, verify-steward |
| bigger-pi-worker-012 | Donated-compute worker client (consent, resource caps, signed, runs on donor's own machine) | code | large | medium | pr | coord-011 | maintainer |
| bigger-pi-abuse-013 | Anti-abuse threat model + guardrails (anti-botnet, consent, signed client, never-trust-one-worker, kill switch) | design-spec | medium | medium | document | worker-012 | maintainer, verify-steward |

**Acceptance criteria — key tasks**

- **bigger-pi-coord-011 (coordinator service):**
  - A documented **build-vs-reuse decision** (custom coordinator vs. running as a **BOINC project**) is
    recorded before implementation; BOINC already provides work distribution, redundant validation, and
    credit.
  - Assigns work units with a **tunable redundancy factor** (e.g., 2× quorum + tie-break recompute,
    reputation-weighted; no duplicate assignment beyond that), keeps upper merge levels on the
    coordinator/assembly host (bandwidth reality), collects results + checksums, dispatches the
    recompute, and records assign→result→verify→merge in the **public, append-only, hash-chained**
    ledger.
  - The coordinator **never executes donor-supplied code**; it only dispatches our defined work units.
  - The published ledger is reproducible: an independent party can re-derive the run's final hash.
- **bigger-pi-worker-012 (worker client):**
  - A **signed, open** client the donor installs and runs **on their own machine with explicit,
    informed consent** and an obvious stop/kill control; honors per-chunk CPU/memory caps.
  - **GIMPS-style checkpoint/restart** (periodic state save) so an interrupted laptop-class session
    resumes without losing work; **client-side self-checking arithmetic** (e.g., mod-p residue check)
    catches most bad results before they consume verification budget.
  - Submits result + checksum + technical env metadata only; **pseudonymous, opt-in** attribution; no
    PII, no behavioral telemetry; no secrets written to logs/artifacts.
- **bigger-pi-abuse-013 (anti-abuse threat model):**
  - Documents the anti-botnet posture (consent-first, no covert/non-consenting use, no
    coordinator-executed code), the never-trust-one-worker rule, DoS/poisoned-unit handling, and the
    kill switch; each mitigation maps to an implemented control.

**Definition of Done (M2):** ≥ 10 distinct external donors complete + verify chunks through the live
coordinator; every accepted chunk has a 2nd-worker recompute match; the ledger is public, append-only,
hash-chained, and independently reproducible; the anti-abuse threat model is implemented + documented.

---

## Milestone M3 — Algorithm/perf hardening (NTT/FFT, GPU optional, Bellard speedups)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| bigger-pi-perf-014 | NTT/FFT bignum multiplication backend (2nd independent backend); throughput up, determinism preserved | code | large | medium | pr | core-002, ci-006 | algo, maintainer |
| bigger-pi-perf-015 | Bellard extractor optimization + optional GPU path (verification gate unchanged) | code | medium | medium | pr | core-003 | algo, maintainer |
| bigger-pi-expert-016 | Expert review of verification methodology (guard-digit analysis, BBP spot-check sufficiency, two-method independence) | research | medium | medium | document | verify-004, merge-008 | SME, verify-steward |

**Acceptance criteria — key tasks**

- **bigger-pi-perf-014 (NTT/FFT backend):**
  - A clean-room NTT/FFT multiplication backend lands as a **second, genuinely independent** bignum
    path (independence preserved for verification).
  - Documented ≥ X× throughput improvement on a fixed benchmark; **passes the determinism +
    verification tests** — the PR does not merge if it breaks bit-reproducibility.
- **bigger-pi-expert-016 (SME methodology review):**
  - A computational-mathematics / numerical-analysis SME reviews and signs off the guard-digit
    derivation, the BBP spot-check count vs. its stated probability bound, and the genuineness of
    backend/two-method independence; sign-off recorded.
  - This sign-off is a **prerequisite for any large-scale publication (M4).**

**Definition of Done (M3):** documented ≥ X× throughput on a fixed benchmark; ≥ 2 genuinely
independent bignum backends maintained; every perf change passes determinism + verification gates;
SME methodology sign-off recorded.

---

## Milestone M4 — Verified moderate-scale run + open dataset release

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| bigger-pi-run-017 | Execute a verified extended-precision run (target sized to justified donated budget); full HARD-RULE verification | data | large | medium | dataset | coord-011, expert-016 | verify-steward, maintainer |
| bigger-pi-release-018 | Open dataset + ReleaseManifest + reproducibility package + compute/energy disclosure (CC0) | data | medium | low | dataset | run-017 | maintainer, verify-steward, steward |

**Acceptance criteria — key tasks**

- **bigger-pi-run-017 (verified moderate-scale run, within the ~1e8–1e10 ceiling):**
  - The run's digit target (within the distributed ceiling, never record-adjacent) and total
    compute/energy budget were **approved in advance** against the energy-cost disclosure, justified by
    education/verification outcomes (no open-ended chase).
  - The result passes the **full HARD RULE** (one main computation + independent-formula BBP spot-checks
    + every chunk recompute-verified + reference prefix match + guard-digit analysis; dual-full reserved
    for baselines), signed by the
    `verify-steward` (not the runner).
  - **Zero unverified digits are published.**
- **bigger-pi-release-018 (open dataset release):**
  - Publishes the digits (CC0), the `ReleaseManifest` (`finalResultHash`, two-method agreement,
    spot-check counts, repro command, compute CPU-hours + energy estimate), and a one-command
    reproduction package; an independent party re-derives the same `finalResultHash`.
  - No secrets/PII anywhere in the release; all artifacts content-addressed.

**Definition of Done (M4):** a new **verified** digit milestone published as a CC0 dataset +
ReleaseManifest + repro package, with two-method agreement, BBP spot-checks, per-chunk verification,
**SME-signed methodology**, and disclosed compute/energy cost. Satisfies the project Definition of
Shipped ("Publicly Shipped (open public good)").

---

## Milestone M5 — Research/education outputs + partner adoption

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| bigger-pi-edu-019 | Educational explainers (binary splitting, BBP, NTT) + reusable distributed-verified-map-reduce pattern writeup | writing | medium | low | document | release-018 | maintainer, algo |
| bigger-pi-partner-020 | Secure research/education partner (university math/CS dept or distributed-computing project) | research | medium | medium | document | — | maintainer, steward |

**Acceptance criteria — key tasks**

- **bigger-pi-edu-019 (educational outputs):**
  - Clear, source-cited explainers of binary splitting, BBP/Bellard extraction, and NTT
    multiplication; plus a reusable "distributed verified map-reduce" pattern writeup; CC-BY-4.0.
- **bigger-pi-partner-020 (secure partner):**
  - A named research/education or distributed-computing org confirms need/interest in writing.
  - On success, `verifiedNeed` flips to `true` and `requestor` is set to the named org across the
    project's tasks.

**Definition of Done (M5):** ≥ 1 educational artifact published + used; ≥ 1 external reuse of the
toolchain/dataset/protocol documented; partner outreach concluded (endorsement upgrades the project
status; absence does not strand the already-shipped public good).

---

## Backlog / future

| ID | Title | Type | Size | Risk | Deliverable | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| bigger-pi-feat-021 | Frontier final-assembly: large-memory / out-of-core consolidation strategy | research | large | medium | document | Out-of-scope frontier beyond the ~1e8–1e10 ceiling; documents the single-host bottleneck |
| bigger-pi-feat-022 | Normality / digit-statistics dataset + analysis notebooks (CC0) | data | medium | low | dataset | Research substrate from verified digits |
| bigger-pi-feat-023 | Schema extension: `computeBudgetCpuHours` for computation-run tasks | code | small | low | pr | Resolves the agent-lane vs. CPU-donation gap (governance) |
| bigger-pi-feat-024 | Worker reputation + pseudonymous credit ledger | code | medium | medium | pr | Strengthens never-trust-one-worker; no PII |
| bigger-pi-feat-025 | Cross-platform reproducibility matrix (OS/arch backends bit-identical) | maintenance | medium | low | document | Hardens the determinism contract |
| bigger-pi-feat-026 | Funded-lane agent-assisted optimization spike (capped) | code | small | low | pr | Would set `lane=funded` + `fundedBudgetUsd` cap |
| bigger-pi-feat-027 | Dependency / supply-chain audit + signed-release hardening | maintenance | small | low | pr | Recurring |
| bigger-pi-feat-028 | Build-vs-reuse spike: bigger-pi as a BOINC project vs. custom coordinator | research | medium | low | document | Resolve before M2; BOINC already solves distribution/validation/credit |
| bigger-pi-feat-029 | Standalone "verify someone else's claim" CLI (re-derives final hash from any dataset + manifest) | code | medium | low | pr | Seed of verification-as-a-service; reuses the harness |
| bigger-pi-feat-030 | Generalized verified map-reduce for other constants (e, ζ(3), Catalan) + CC0 constant corpus | code | large | low | dataset | Constant-agnostic spin-off; oracle/benchmark for GMP/MPFR/Arb |
| bigger-pi-feat-031 | Shared Elyos "independent-verification gate" primitive (runner-separated recompute + cross-formula check) | code | medium | low | pr | Reusable platform capability across Elyos projects |

---

## Example task JSON

Complete, schema-valid Task JSON for the first M0 task. `verifiedNeed` is `false` and `requestor` is
`"TBD"` because **no partner org is secured yet** (honest default per the plan).

```json
{
  "id": "bigger-pi-arch-001",
  "title": "ADRs: dual bignum-backend strategy, (P,Q,T) artifact format, coordination+ledger schema, guard-digit policy, determinism contract",
  "project": "bigger-pi",
  "type": "design-spec",
  "lane": "donated",
  "priority": "high",
  "domain": ["mathematics", "distributed-computing", "software", "open-data"],
  "riskTier": "low",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "small",
  "status": "open",
  "context": "bigger-pi is an open, distributed, verification-first effort to compute more digits of pi on donated compute, where the public good is the open, reproducible, independently-verified toolchain + chunk-coordination protocol + provenance ledger. The HARD RULE - no digit accepted or published without independent verification (one main Chudnovsky computation + independent-formula BBP/Bellard hex spot-check + per-chunk recompute; dual full computation reserved for small reference baselines to avoid doubling energy) - is the project's spine. This task records the foundational architecture decisions that everything else depends on. Verification independence requires two genuinely independent bignum backends, and the verification harness's bit-identical contract depends on both the backend choice and the guard-digit policy, so these ADRs must land before the harness is finalized.",
  "objective": "Record ADRs for: (1) a dual, independent bignum-backend strategy (a vetted open library plus a clean-room NTT/FFT path) and the FFI/WASM boundary; (2) the (P,Q,T) artifact serialization + hashing format; (3) the coordination protocol + append-only hash-chained ledger + reproducibility-manifest schema; (4) the guard-digit / rounding policy for the single-host final assembly; (5) the determinism contract defining what bit-identical means across backends and platforms.",
  "acceptanceCriteria": [
    "ADR selects two genuinely independent bignum backends with verification independence as the deciding criterion, and documents the LGPL-linking stance for any library used",
    "Artifact format ADR defines (P,Q,T) serialization plus SHA-256 and a second independent hash for content addressing",
    "Coordination + ledger ADR defines WorkUnit, ChunkResult, VerificationRecord, LedgerEntry, ReleaseManifest with an append-only hash-chained, signed ledger and no secrets anywhere in the schema",
    "Guard-digit policy specifies how G is chosen and how published precision excludes unstable trailing digits",
    "Determinism contract states that exact-integer (P,Q,T) results are always identical, that the reproducible reference path uses integer/fixed-modulus NTT (not floating FFT) so bit-identity holds across architectures, and that final-assembly rounding is pinned by the guard-digit policy",
    "ADRs are reviewed by maintainer, algo reviewer, and the verification steward before M0 implementation tasks proceed"
  ],
  "resources": [
    "C:\\code\\elyos\\planning\\projects\\bigger-pi\\PLAN.md",
    "C:\\code\\elyos\\planning\\ROADMAP.md",
    "C:\\code\\elyos\\CLAUDE.md",
    "C:\\code\\elyos\\docs\\good-deed-definition.md"
  ],
  "output": "A document recording ADRs #1-#5 (dual bignum-backend strategy, (P,Q,T) artifact format, coordination+ledger schema, guard-digit policy, determinism contract) that gate the M0 implementation and verification tasks.",
  "requestor": "TBD",
  "verifiedNeed": false,
  "outputLicense": "CC-BY-4.0"
}
```

---

## Generated task index

Every milestone/backlog row above now has a schema-valid Elyos Task JSON under `tasks/` (validated
against `packages/schema/src/schemas.ts`; `filename == id`; no duplicates; no extra keys). The seed
`bigger-pi-arch-001.json` is kept as-is.

**Fan-out:** none. This is a single-computation project with no plan-enumerated fan-out dimension
(no language set, no named dataset/source list, no document/topic set). Each backlog row maps to
exactly **one** representative task — no items were fabricated. Items that depend on unsecured
inputs (the M4 digit target/budget, the partner org, the verification SME) remain a single task each
and expand only on partner/scope/governance confirmation.

**Field policy applied** (per "How these tasks map to Elyos"): `lane=donated` for all tasks except
`bigger-pi-feat-026` (the explicit funded-lane spike — `lane=funded` with a hard
`fundedBudgetUsd` cap; the value is a **conservative placeholder pending governance approval**);
`verifiedNeed=false` and `requestor="TBD"` everywhere (no partner secured); `outputLicense` =
`MIT` (code), `CC0-1.0` (digit data/datasets/ledgers), `CC-BY-4.0` (design-spec/research/writing
docs); `riskTier` mirrors the Risk column (verification/computation-run/coordination/abuse =
`medium`; infra/teaching = `low`). No task authors refused content; the HARD RULE and
consent/anti-botnet/no-secrets guardrails are carried verbatim into the relevant `context` and
`acceptanceCriteria`.

**Acceptance criteria for rows without an explicit block above** (authored in the JSONs; summarized
here so `TASKS.md` stays authoritative):

- **bigger-pi-core-003 (BBP/Bellard hex extractor):** independent code path from Method A; extracted
  hex digits match a known reference for spot-checked offsets; deterministic; TS/ESM, pnpm
  build/test/lint pass; MIT.
- **bigger-pi-bench-010 (final-assembly benchmark):** reproducible memory/wall-time benchmark of the
  single-host final assembly across precision targets; quantifies the bottleneck + largest feasible
  precision; documents mitigations without overstating; CC-BY-4.0.
- **bigger-pi-perf-015 (Bellard optimization + optional GPU):** optional, off-by-default GPU path not
  required for correctness; verification gate unchanged and passing; determinism preserved; documented
  throughput gain; does not merge if it breaks determinism/verification; MIT.
- **bigger-pi-feat-021 (frontier final-assembly):** out-of-core/large-memory consolidation strategy
  addressing the single-host bottleneck; memory/time tradeoffs + reproducible plan; preserves
  determinism/verification contracts; CC-BY-4.0.
- **bigger-pi-feat-022 (normality/digit-statistics):** statistics derived only from already-verified
  published digits; reproducible notebooks + CC0 dataset; no claims beyond the data; no PII; CC0-1.0.
- **bigger-pi-feat-023 (computeBudgetCpuHours):** optional, backward-compatible schema field analogous
  to `fundedBudgetUsd`, governance-gated; existing tasks remain valid; schema tests; MIT.
- **bigger-pi-feat-024 (worker reputation/credit ledger):** pseudonymous credit/reputation
  strengthening never-trust-one-worker; no PII; opt-in; no secrets; MIT.
- **bigger-pi-feat-025 (cross-platform repro matrix):** OS/arch x backend matrix proving bit-identical
  (P,Q,T); hardens + documents the determinism contract; records deviations; CC-BY-4.0.
- **bigger-pi-feat-026 (funded-lane spike):** runs only via `packages/runner` under a hard
  `fundedBudgetUsd` cap (never exceeded) with a public cost ledger; capped, time-boxed; verification
  gate unchanged; no secrets in logs/receipts/artifacts; governance-approved before spend; MIT.
- **bigger-pi-feat-027 (supply-chain audit + signed releases):** dependency/license audit removing or
  flagging incompatible deps; signed-release hardening (pinned lockfile); no secrets; MIT.

**Index (id -> milestone):**

- M0: `bigger-pi-arch-001` (seed), `bigger-pi-core-002`, `bigger-pi-core-003`, `bigger-pi-verify-004`,
  `bigger-pi-ledger-005`, `bigger-pi-ci-006`
- M1: `bigger-pi-chunk-007`, `bigger-pi-merge-008`, `bigger-pi-verify-009`, `bigger-pi-bench-010`
- M2: `bigger-pi-coord-011`, `bigger-pi-worker-012`, `bigger-pi-abuse-013`
- M3: `bigger-pi-perf-014`, `bigger-pi-perf-015`, `bigger-pi-expert-016`
- M4: `bigger-pi-run-017`, `bigger-pi-release-018`
- M5: `bigger-pi-edu-019`, `bigger-pi-partner-020`
- Backlog: `bigger-pi-feat-021`, `bigger-pi-feat-022`, `bigger-pi-feat-023`, `bigger-pi-feat-024`,
  `bigger-pi-feat-025`, `bigger-pi-feat-026`, `bigger-pi-feat-027`

Total: **27** task JSONs (1 pre-existing seed + 26 generated).
