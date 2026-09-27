# 07 — Evaluation

Status: **implemented and run (Phase 11)** — see [[implementation-status]] for the module
table (`src/core/evaluation/`, `src/benchmark/`) and [[04-decisions]] #17 for why the
"agent alone" condition is a simulated naive-agent baseline rather than a live second LLM.

## Critical experiment (once something is built)

Strong AI agent alone **vs.** strong AI agent + ECC. Never compare against a weak baseline.

## Metrics to eventually track

- **Evidence**: critical evidence recall, irrelevant evidence rate, provenance completeness.
- **Judgment**: expert-rated usefulness, decision quality, risk detection, constraint
  recall, historical recall, verification quality.
- **Efficiency**: tokens, agent turns, latency, compute cost, time saved.
- **Trust**: unsupported claims, hallucination rate, correction rate, provenance accuracy.

## Benchmark (implemented, Phase 11)

Each benchmark task carries: a free-text request and hand-picked ground-truth relevant paths
(`src/core/evaluation/types.ts`'s `BenchmarkTask`). `src/benchmark/benchmarkTasks.ts` defines
three tasks against **this repository's own real codebase** (dogfooding — resolves the
"no target repository chosen" gap flagged since Phase 10), not only the tiny test fixture.
`npm run benchmark` runs `retrieveBaselineEvidence` ("agent alone": keyword grep, no
ranking/trust/compression) and the core pipeline ("agent+ECC") against each task and scores
both against the same ground truth. Future extensions (unseen repos, temporal holdouts,
adversarial/legacy/monorepo tasks) remain open — three same-repo tasks satisfy "at least one"
but are not exhaustive coverage.

## Current baseline

**Re-run 2026-09-27** (originally measured 2026-09-12) via `npm run benchmark` against this
repository (3 tasks; see [[implementation-status]] Phase 11 section for the exact tasks). The
benchmark dogfoods this repo's own codebase, so the absolute figures drift as the code grows —
the stable result is ECC reaching 100% recall on fewer tokens than the baseline. Average across
all three:

| Metric | Agent alone | Agent + ECC |
|---|---|---|
| Evidence recall | 50% | 100% |
| Irrelevant evidence rate | 89% | 91% |
| Provenance completeness | 0% | 100% |
| Estimated tokens | 3319 | 1439 |

ECC recalls all ground-truth-relevant files every time (this run the baseline misses relevant
evidence in two of three tasks — 0% recall on `trust-level-extension`, 50% on
`token-budget-fix`); every ECC item carries provenance/trust by construction (Phase 7), the
baseline has none; ECC's compression uses under half the tokens (1439 vs. 3319) the baseline
spends reading whole files. The one metric where ECC measures worse — irrelevant evidence rate — is honest,
not a flaw hidden by cherry-picked tasks: each task's ground truth is a narrow 2-path list,
and ECC's supplementary git/test evidence (which the baseline never retrieves at all) counts
as "irrelevant" under this metric's strict definition even though it's legitimate supporting
context. Recorded as-is per [[04-decisions]] #17.

## Golden examples as independent, reproducible measurements

`docs/examples/golden-example-01-debugging/` and `docs/examples/golden-example-02-refactoring/`
(see [[04-decisions]] #23) each ship a runnable `measure.mjs` calling this same
`retrieveBaselineEvidence`/`runEvaluation` methodology against a small, purpose-built fixture
repo, independent of the benchmark above. Results are consistent with this section's honest
framing rather than uniformly favorable: Example 1 shows a clear ECC win under a tight budget
(150 tokens: 33% vs. 100% recall, 0% vs. 100% provenance); Example 2 shows a real tie on file
recall between the naive baseline and ECC at a generous budget, with ECC's advantage coming
from provenance/memory/risk-scaled verification rather than recall. Neither result was
cherry-picked to only show ECC winning — both are reported in full in each example's own
README.
