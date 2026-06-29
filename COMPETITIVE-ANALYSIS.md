# Competitive & Improvement Analysis — bigger-pi

> Analyst review of `PLAN.md` (Draft v0.1.0, 2026-06-28). Web-researched, cited. Focus: technical
> correctness, competitive landscape, and where Elyos + Claude can credibly win.
> Analysis date: 2026-06-29.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually self-aware: it already concedes the single-host final-assembly bottleneck, demotes raw digit count as a vanity metric, and reframes value toward the toolchain. That honesty is its biggest strength. But several technical claims and framing choices need tightening before M0.

**1.1 The headline premise ("extend pi beyond the record via donated chunks") is, at the frontier, not feasible — and the strongest external authority says so explicitly.** Alexander Yee's own y-cruncher FAQ states that distributed computing "cannot be used to set records" because of verification and practical constraints, and the current record (314 trillion digits, StorageReview, Dec 2025) ran on a **single** Dell PowerEdge R7725 with 1.5 TB RAM and ~2 PB of SSD for ~110 days ([y-cruncher](https://www.numberworld.org/y-cruncher/); [StorageReview 314T](https://www.storagereview.com/review/storagereview-sets-new-pi-record-314-trillion-digits-on-a-dell-poweredge-r7725); [Tom's Hardware](https://www.tomshardware.com/pc-components/storage/pi-calculating-record-shattered-at-314-trillion-digits-with-a-four-month-run-on-a-single-server-storagereview-retakes-the-crown-thanks-to-storage-bandwidth)). The plan's exec summary still leads with "compute more digits of π" and a "verified precision milestone… M4 run sized to donated budget." **Recommendation: explicitly retire any beat-the-record ambition** and state up front that the deliverable is an open, verifiable, educational/moderate-scale pipeline. The plan is ~80% of the way there; it should finish the pivot rather than leave a foot in both camps.

**1.2 "Clean, associative map-reduce" oversells distributability; the merge tree is the catch the plan underweights.** Binary splitting's leaves are independent, but it is **not embarrassingly parallel**: the upper levels of the `(P,Q,T)` merge tree perform multiplications of integers approaching full target precision, which only pay off with sub-quadratic multiplication (Toom–Cook / Schönhage–Strassen / NTT) and are memory- and bandwidth-bound ([Wikipedia: Binary splitting](https://en.wikipedia.org/wiki/Binary_splitting)). Critically for a *donated* model: the `T`/`Q` triples near the root grow to roughly the full digit size. At 1e8 digits a triple is tens of MB (fine to ship to volunteers); at 1e12+ it is hundreds of GB to TB — you cannot transit that across consumer uplinks. So the distributed-donation model is genuinely useful **only up to ~1e8–1e10 digits**; beyond that the network transfer of near-root triples and the single-host final assembly dominate. The plan should state this scale ceiling numerically as a first-class design fact, not just a risk row.

**1.3 BBP/Bellard is base-16, position-specific, and not free — the plan is mostly right but a reader could over-read it.** BBP/Bellard extract **hexadecimal** digits and cannot directly produce base-10 digits ([Wikipedia: BBP](https://en.wikipedia.org/wiki/Bailey%E2%80%93Borwein%E2%80%93Plouffe_formula)). The plan correctly uses BBP as an *independent-formula spot-check*, which mirrors real practice (y-cruncher verifies via a BBP-style check of the trailing hex). Two clarifications to add: (a) BBP is "embarrassingly parallel" per-position but extracting the digit at position *n* still costs roughly O(n log n) work — it is cheap relative to a full computation but not constant-time, so a "spot-check at the hex tail of a 1e12-digit number" is itself a non-trivial job; (b) the cross-check compares **hex** digits of the assembled value, so the harness must convert/extract consistently — worth making explicit in the determinism contract.

**1.4 "Two independent full computations agree bit-for-bit" doubles compute and contradicts the energy-budget goal.** Real record runs do **one** main Chudnovsky computation plus a cheap independent **BBP final-digit verification** — not two full runs. Requiring dual *full* computation as the default doubles CPU-hours and energy for every published result, directly in tension with the plan's "no compute for compute's sake" guardrail and per-run energy disclosure. **Recommendation:** make the standard gate = one full computation + BBP/Bellard independent-path verification + per-chunk recompute; reserve dual-full-computation for small reference baselines or when BBP coverage is insufficient. Also note "bit-for-bit" only holds for *exact integer* stages; the final assembly (one big divide + one big sqrt) is the sole rounding step, so cross-backend agreement there is "equal after dropping G guard digits," which the plan does handle.

**1.5 Determinism / "bit-identical merged == single-host" is sound but platform-fragile.** Exact-integer `(P,Q,T)` are reproducible across platforms; the FFT/NTT multiply path is where non-determinism creeps in (floating-NTT rounding, SIMD reassociation). The plan's "two backends, one clean-room NTT" is good, but the determinism ADR must specify integer-only or fixed-modulus NTT for the reproducible reference path, or the bit-identical claim will quietly fail in CI across architectures.

**1.6 Minor / completeness.** (a) `N ≈ digits/14.18` is correct (Chudnovsky yields ~14.18 digits/term). (b) Success-metrics table still lists a "verified precision milestone (≥1e8 baseline, larger M4 run)" — fine as an *educational* target but should not be labeled in record-adjacent language. (c) The Elyos schema gap (CPU-hour donation vs. `tokenEstimate`/`fundedBudgetUsd`) is correctly surfaced; this is a real governance blocker for M2 and should be resolved before, not during, the live-coordinator milestone. (d) No mention of **checkpoint/restart** for long chunks (GIMPS saves state every 30 min) — essential for donated sessions that get interrupted; add to the worker-client spec. (e) No **anti-Sybil / result-trust economics** detail beyond "2nd-worker recompute" — at scale, redundant verification is your dominant cost; specify the redundancy factor and how disputed units are adjudicated.

**Verdict:** Technically literate and refreshingly honest, but the framing must complete its pivot. The realistic, defensible goal is **education + independent verification + reproducible open pipeline + donated-compute coordination at moderate scale**, explicitly *not* a digit record. As written, the plan can ship something genuinely novel; as titled ("bigger-pi", "extend digits"), it invites the one comparison it cannot win.

---

## 2. Competitive landscape

### Record-setters (the "scale" frontier — bigger-pi should NOT compete here)

- **y-cruncher (Alexander J. Yee)** — the de facto engine behind essentially every modern π record; Chudnovsky for computation + an independent formula for verification. *Strengths:* extreme optimization, swap-mode (disk-backed) scaling, decades of tuning, built-in verification. *Weaknesses/gaps:* **closed source**, single-node only (author states distributed computing "can't set records"), not a teaching artifact, not reproducible from a public ledger. ([numberworld.org/y-cruncher](https://www.numberworld.org/y-cruncher/))
- **StorageReview 314T (Dec 2025)** — current record, single Dell R7725, 1.5 TB RAM, ~2 PB SSD, 110 days, ~4,305 kWh (~13.7 kWh/trillion digits). *Strength:* energy-efficient single-server result that beat cloud. *Gap:* hardware-heroics, not open/reproducible/educational. ([StorageReview](https://www.storagereview.com/review/storagereview-sets-new-pi-record-314-trillion-digits-on-a-dell-poweredge-r7725); [TechRadar](https://www.techradar.com/pro/anyone-for-a-slice-of-record-pi-new-landmark-sees-314-trillion-digits-calculated-as-news-site-trounces-google-cloud-for-now))
- **Google Cloud 100T (Emma Haruka Iwao, 2022)** — 100 trillion digits, 157 days, y-cruncher on cloud VMs, ~82 PB of data processed. Prior 31.4T (2019) by the same author. *Strength:* cloud reproducibility writeups, strong public communication. *Gap:* still single-(big)-instance y-cruncher; closed engine; cloud cost gatekeeps participation. ([Google blog](https://blog.google/innovation-and-ai/infrastructure-and-cloud/google-cloud/new-digit-pi-2022/); [Google Cloud blog](https://cloud.google.com/blog/products/compute/calculating-100-trillion-digits-of-pi-on-google-cloud); [Wikipedia: Iwao](https://en.wikipedia.org/wiki/Emma_Haruka_Iwao))

### Distributed / donated-compute models (the real peer group — copy these patterns)

- **GIMPS / PrimeNet** — the gold standard for volunteer math compute: central server hands out work units, Prime95 client runs in background, checkpoints every 30 min, reports hourly, PRP test with a cheap verification (Gerbicz/proof) and Lucas–Lehmer recheck. *Strengths:* 25+ year sustained community, robust trust model, work that **is** naturally distributable (one exponent per unit). *Weakness for us:* prime-search is independent per candidate — π's merge tree is not, so we can't copy GIMPS's trust model wholesale. ([mersenne.org/works](https://www.mersenne.org/various/works.php); [Wikipedia: GIMPS](https://en.wikipedia.org/wiki/Great_Internet_Mersenne_Prime_Search))
- **BOINC** — the generic volunteer-computing platform: server splits datasets into work units, schedules by device capability, **validates via redundant computation**, awards credit; ~30 active science projects; volunteers attach to many at once. *Strength:* exactly the coordinator/redundancy/credit infrastructure bigger-pi proposes to rebuild. *Gap/opportunity:* bigger-pi could be a **BOINC project** rather than a from-scratch coordinator (build vs. reuse decision). ([BOINC Berkeley](https://boinc.berkeley.edu/); [Anderson 2019 PDF](https://boinc.berkeley.edu/boinc_a_platform_for_volunteer_computing.pdf))
- **Folding@home** — GPU-heavy volunteer computing; first exascale system (2020); embarrassingly-parallel independent simulations. *Strength:* proof that a million volunteers + GPUs = supercomputer-class throughput. *Relevance:* its workloads are independent (no global merge), which is why it scales — a cautionary contrast for π. ([foldingathome.org](https://foldingathome.org/2020/07/26/citizen-scientists-create-an-exascale-computer-to-combat-covid-19/); [arXiv 2303.08993](https://arxiv.org/abs/2303.08993))
- **PiHex (Colin Percival, 1998–2000)** — the *direct historical precedent*: distributed **Bellard/BBP** computation of specific π **bits** (5T, 40T, quadrillionth bit), 1,734 computers, 56 countries. *Strength:* proved BBP-style position extraction distributes beautifully because each position is independent. *Gap (our opening):* it computed isolated **bits**, never a contiguous decimal range, was not reproducible from a public ledger, and is long dormant. ([Wikipedia: PiHex](https://en.wikipedia.org/wiki/PiHex); [SFU announce](https://wayback.cecm.sfu.ca/projects/pihex/announce1q.html))
- **Bellard's formula / Fabrice Bellard** — faster BBP variant; Bellard also held a 2009 Chudnovsky record on a single desktop. *Relevance:* the independent-formula verification path bigger-pi relies on. ([Wikipedia: BBP](https://en.wikipedia.org/wiki/Bailey%E2%80%93Borwein%E2%80%93Plouffe_formula))

**Landscape summary:** Two non-overlapping worlds — closed single-node record engines (y-cruncher and its operators) and open volunteer-compute platforms (GIMPS/BOINC/FAH/PiHex). **No one occupies "open, reproducible, ledger-backed, independently-verified, educational π pipeline."** That gap is real and bigger-pi-shaped.

---

## 3. Gaps we can fill (honest framing — not a record)

1. **Open, reproducible toolchain.** Every record runs on closed y-cruncher; there is no MIT-licensed, end-to-end, reproducible-from-a-public-ledger Chudnovsky+BBP pipeline. This alone is a genuine, citable public good.
2. **Independent verification as a shared, reusable harness.** Guard-digit analysis, BBP spot-check sufficiency, dual-backend independence are expert folklore. Packaging them as an open "verify-any-long-computation" library is novel and reusable beyond π.
3. **Donated-compute coordination *with a documented merge*.** PiHex distributed independent *bits*; nobody has published an open, ledgered map-reduce that distributes a *contiguous* Chudnovsky series and merges to a bit-identical single-host result. Even capped at ~1e8–1e10 digits, that's a first.
4. **Education.** Binary splitting, NTT multiplication, and BBP are superb teaching artifacts; classroom-ready explainers + runnable reference code fill a real pedagogical gap.
5. **A CC0 verified-digit corpus + reproducibility manifest** as an oracle/benchmark for testing other arbitrary-precision libraries.

---

## 4. Differentiators to win

- **Reproducibility-first, ledger-backed provenance.** "Any outsider can re-derive every published digit from a public hash-chained ledger + one command." No record holder offers this. This is the single strongest wedge.
- **Verification-as-identity, not as a footnote.** Two independent code paths + BBP cross-check + per-chunk recompute, with the steward separated from the runner. Trustworthiness, not scale, is the product.
- **Open + MIT/CC0 end-to-end** vs. closed y-cruncher — forkable, auditable, teachable.
- **Genuinely participatory at human scale.** A student's laptop contributes a *real* verified chunk and gets ledger credit — the GIMPS/BOINC emotional hook, applied to π, with a transparent merge.
- **Honest scoping as a feature.** Publicly stating "we are not chasing the record; we are making π computation verifiable and learnable" is itself differentiating in a field full of headline-grabbing claims.

---

## 5. Claude API leverage

**Where Claude clearly helps (build/explain/orchestrate):**
1. **Implementing & optimizing the algorithms** — generating the clean-room NTT/FFT bignum backend, the binary-splitting recurrence, Bellard extractor; profiling hot loops; proposing Toom/SS/NTT trade-offs; porting inner loops to WASM/native. (Use a coding agent in the donated lane, or `packages/runner` with a hard `fundedBudgetUsd` cap for funded optimization work.)
2. **Writing the verification harness and test suite** — guard-digit analysis tooling, BBP spot-check selection, determinism/repro CI tests, dependency-license and no-secrets audits, property-based tests for the merge associativity.
3. **Educational content & explainers** — classroom-ready writeups of binary splitting, NTT, and BBP; annotated reference walkthroughs; the "distributed verified map-reduce" pattern doc. This is where an LLM's strength (clear exposition) and the project's goal (education) align perfectly.
4. **Orchestration & coordination glue** — work-unit manifest generation, ledger schema, coordinator service code, chunk-sizing heuristics, anomaly/abuse triage on submitted results.
5. **Literature/landscape synthesis** — keeping the provenance citations and patent-encumbrance review current.

**Where Claude must NOT decide (hard wall):**
- **No numeric result is ever asserted by the LLM.** Digits are accepted only when produced and verified by independent *algorithms/code* (dual path + BBP + per-chunk recompute). An LLM "believing" a digit is correct has zero evidentiary weight. **No hallucinated digits** ever enter the ledger.
- **Verification gates live in code, not in a prompt.** Claude may *write* the gate; it may never *be* the gate. The reproducibility check, hash comparison, and bit-identity test must be deterministic code runnable by anyone.
- **No LLM in the trust path of result adjudication.** Disputed work units are resolved by recompute + hash, not by model judgment.
- **Guard back-pressure:** because LLM-generated math code can be subtly wrong, every Claude-authored numeric routine must pass the same determinism + cross-backend + known-reference gates before use — treat Claude's code as an untrusted contributor.

---

## 6. Ten concrete optimizations

1. **Build on BOINC instead of a from-scratch coordinator** (or at minimum, adopt its work-unit/redundant-validation/credit model). Saves a milestone of infra and inherits a proven trust model. ([BOINC](https://boinc.berkeley.edu/))
2. **Adopt GIMPS-style checkpoint/restart** (state saved every N minutes) so interrupted donated sessions don't lose work — essential for laptop-class donors. ([mersenne.org/works](https://www.mersenne.org/various/works.php))
3. **Replace default dual-full-computation with one computation + BBP/Bellard verification** to halve energy per result; reserve dual-full only for small reference baselines.
4. **Integer/NTT-modular multiplication for the reference path** (fixed prime moduli + CRT) to guarantee cross-platform bit-identity instead of fragile floating-FFT.
5. **Cap the realistic scale ceiling explicitly** (~1e8–1e10 digits for true distribution) and route anything larger to a documented single-host assembly path — design to the ceiling rather than against it.
6. **Stream/compress `(P,Q,T)` triples and keep upper merge levels on the coordinator** (or a designated assembly host), so volunteers only ever handle small leaf/low-level ranges — matches the bandwidth reality.
7. **Redundancy factor as a tunable** (e.g., 2x quorum with tie-break recompute) à la BOINC validation, with reputation-weighted assignment to cut wasted duplicate compute.
8. **GPU path for the NTT multiply (optional, M3)** following Folding@home's evidence that volunteer GPUs dominate throughput — gated by the determinism contract.
9. **Self-checking arithmetic** (e.g., mod-p residue checks on each chunk before submission) so most bad results are caught client-side, before they consume verification budget. (Analogous to GIMPS's Gerbicz error-check.)
10. **Ship a "verify someone else's claim" CLI** that takes any published π dataset + manifest and re-derives the final hash — turns the verification harness into a standalone, reusable product (see §7).

---

## 7. Parallel & perpendicular spin-offs

- **Generalized donated-compute coordinator** — the work-unit/ledger/verification layer is constant-agnostic. A reusable "verified map-reduce for any binary-splitting hypergeometric series" (e, ζ(3), Catalan, log 2) is a bigger public good than π itself, and a natural BOINC-adjacent contribution.
- **Verification-as-a-service** — "give us your long-computation result + repro command, we independently re-derive and certify it." Directly serves other arbitrary-precision projects as an oracle and reuses the entire harness.
- **Other constants & a CC0 constant corpus** — verified digit datasets for several constants as a benchmark/test oracle for arbitrary-precision libraries (GMP, MPFR, Arb).
- **Normality / statistics research substrate** — a clean, provenance-tagged CC0 digit corpus for normality studies.
- **Education track as its own product** — an interactive "compute π and prove it" course; the alternating algorithm-work ↔ computation-run loop is a great teaching narrative.
- **Tie to atltuae's loop/verification architecture** — bigger-pi's HARD RULE (independent recompute + cross-formula check, runner separated from verifier) is the same shape as atltuae's loop/verify pattern. Factor a **shared Elyos "independent-verification gate" primitive** both projects consume — verification-first becomes a reusable Elyos platform capability, not a per-project reinvention.

---

## 8. Open questions for the maintainer

1. **The core reframe:** Will you formally retire "bigger/beat-the-record" framing and rename/re-scope to an **open, verifiable, educational π pipeline**? The plan is 80% there; finishing the pivot removes the one comparison you can't win and sharpens every metric. (Strong recommendation: yes.)
2. **Build vs. reuse:** Coordinator from scratch, or **bigger-pi as a BOINC project**? This is a milestone-sized decision and BOINC already solves work distribution, redundant validation, and credit.
3. **Scale ceiling:** Will you publicly commit to a realistic distributed ceiling (~1e8–1e10 digits) and treat frontier assembly as out of scope, rather than leaving an implied record ambition?
4. **Verification cost model:** Default to dual-full-computation (2x energy) or one-computation + BBP (record-industry standard)? This materially changes your energy disclosures and budget.
5. **Determinism contract:** Integer/modular-NTT only for the reference path, to make "bit-identical" real across architectures?
6. **Elyos schema gap:** Resolve `computeBudgetCpuHours` (CPU-hour donation) before M2, or block the live coordinator on it?
7. **Partner:** Is the most natural first partner a **university math/CS department** (education) or an **existing volunteer-compute org / BOINC** (distribution)? Pursuing both dilutes; pick the wedge.
8. **Energy justification:** What public-good threshold justifies any compute spend at all, given that "more π digits" has near-zero intrinsic value and the honest product is the *pipeline*? Frame the budget against education/verification outcomes, not digit counts.
