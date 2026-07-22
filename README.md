# bigger-pi

> **bigger-pi** is an open, distributed, **verification-first** effort to compute more digits of π on donated compute. The intrinsic value of "more digits" is modest; the public good is the **open, repr  ·  **Risk tier:** med  ·  **Status:** planning

**bigger-pi** is an open, distributed, **verification-first** effort to compute more digits of π on donated compute. The intrinsic value of "more digits" is modest; the public good is the **open, reproducible, independently-verified toolchain + chunk-coordination protocol + provenance ledger** that produces them — most record-scale π software today is closed, bespoke, or unreproducible. bigger-pi turns a famously single-host, heroic-effort computation into a **citizen-science map-reduce** that ordinary donated machines can meaningfully contribute to, and proves every digit it publishes.

**Definition of shipped:** (1) was produced by independent, credit-sized chunks merged map-reduce to a **bit-identical** single-host result; (2) passes the full **HARD RULE** (dual computation + BBP spot-check + per-chunk recompute + reference cross-check + guard-digit analysis); (3) is published as an **o

This is an **Hee-Lee Oss** good-deed project. Contributors pull a task, do it with their own coding agent, and open a PR. Platform: https://github.com/jdev1977/hee-lee-oss

## Plan
- [PLAN.md](./PLAN.md) — robust enterprise plan (vision, architecture, roadmap, risks; includes an applied-improvements appendix + review sign-off)
- [TASKS.md](./TASKS.md) — schema-mapped task backlog
- [tasks/](./tasks/) — ready-to-pull task JSON(s)

## Contribute
```bash
hee-lee-oss browse
hee-lee-oss next --repo Hee-Lee-Oss-Projects/bigger-pi --no-fork
```

## Licensing & review
- Open license (see PLAN.md).
- Risk tier **med** — deeds are *delivered, not merged*; a domain reviewer (and expert sign-off for any high-stakes content) must approve before merge.

> Planning stage; no adopting partner secured yet (`verifiedNeed: false` on delivery-dependent tasks).
