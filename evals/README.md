# Evaluator Regression Suite

This suite is the regression safety net for the **policy in this repository** — the
standards and the evaluator prompt. It tests one thing: does the evaluator flag what the
standards say it should, and stay silent on what it shouldn't.

It lives here, with the standards, on purpose. Each fixture is the **executable form of a
standard's "what is / isn't a violation" prose** — the standard states the intent in words,
the fixture proves it in behavior. They evolve together: a change to a standard, its
fixtures, and the baseline all belong in the same pull request, reviewed together. And
because the suite is wired into this repo's CI (`.github/workflows/evals.yml`), a standards
or prompt change that regresses the corpus **blocks its own PR** — the policy is held to a
regression test before it can be pinned and rolled out to consuming repos.

The suite depends only on this repo (the prompt, the standards, and the fixtures' embedded
synthetic diffs). It does **not** depend on any application repository.

## Layout

```
evals/
├── fixtures/        labeled cases — should-flag (recall) and should-not-flag (precision)
├── baseline.json    last-accepted per-standard recall / false-positive rates
└── README.md
```

The **harness and the evaluator engine live in `governance-ci`** (the shared library), not
here — so the corpus is tested by exactly the engine that runs in production. CI runs it for
you: `.github/workflows/evals.yml` is a thin caller into
`governance-ci/.github/workflows/standards-evals.yml@v1`, which checks the engine out and
runs the corpus against this repo's candidate policy on every PR.

## Running it locally

Clone `governance-ci` alongside this repo, then point its harness at this corpus:

```bash
# Stub — deterministic self-test, no API key, no cost.
python3 ../governance-ci/scripts/run_evals.py \
  --fixtures-dir evals/fixtures --baseline evals/baseline.json --standards-dir .

# Live — the real regression test, against this repo's standards.
ANTHROPIC_API_KEY=sk-... python3 ../governance-ci/scripts/run_evals.py --live \
  --fixtures-dir evals/fixtures --baseline evals/baseline.json --standards-dir .

# Re-baseline after an intentionally accepted behavior change (add --update-baseline).
```

Scoring is by tolerance, not equality (the model is non-deterministic): an `expect` passes
if the standard fired in ≥ `passThreshold` of N runs; an `expectNot` passes if it fired in
≤ `tolerance`. The runner prints per-standard recall / false-positive rates, a per-fixture
breakdown, and a comparison to `baseline.json`, and exits non-zero on any regression so CI
can gate on it.

## The flywheel

Every false positive or false negative seen on a real PR becomes a new fixture. Over time
the corpus becomes the institutional memory of every mistake the evaluator has made, and no
future policy change can silently reintroduce one.

## Growing coverage (multi-language)

The seed fixtures are Java because the POC's demo app was Java, but the standards are
language-agnostic. As the fleet spans languages, grow the corpus to cover them — structure
fixtures by standard and language so coverage is legible (e.g. "STD-001 proven on Java and
Python, not yet Go").

## One engine, no drift

The evaluator (`evaluate.py`) lives once in `governance-ci` and is used by both the
production pipeline and this regression suite — so the corpus tests exactly what runs in
production. See `SCALING-ARCHITECTURE.md`.
