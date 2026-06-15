# Evaluator Cost Model

This document estimates what it costs to run the LLM evaluator, and — more importantly —
gives you the assumptions to re-run the numbers as you learn your real workload. It is a
planning artifact, not a guarantee. Treat every number here as "edit these inputs, recompute."

## How pricing works

Models are priced per **token**. A token is roughly three-quarters of a word (about four
characters), so one million tokens is on the order of 750,000 words. You pay separately for:

- **Input** — everything you send the model: the prompt, the standards, and the PR diff.
- **Output** — everything the model generates back: the findings.

Output is priced about five times higher than input, because generation is the expensive
part. A price quoted as "$3 / $15 per 1M" means $3 per million input tokens and $15 per
million output tokens.

The current per-1M prices for the candidate models:

| Model | Model ID | Input ($/1M) | Output ($/1M) |
|---|---|---:|---:|
| Haiku 4.5 | `claude-haiku-4-5` | 1 | 5 |
| Sonnet 4.6 | `claude-sonnet-4-6` | 3 | 15 |
| Opus 4.8 | `claude-opus-4-8` | 5 | 25 |

## Token assumptions (edit these)

Every evaluation sends a fixed block of context plus the variable PR diff:

| Piece | Tokens (≈) | Notes |
|---|---:|---|
| Prompt template | 700 | Fixed |
| Three standards, concatenated | 1,500 | Fixed per pinned standards ref |
| Findings schema (detection slice) | 700 | Fixed |
| **Fixed context subtotal** | **~2,900** | |
| PR diff — small (~50 lines) | 700 | Variable |
| PR diff — medium (~300 lines) | 4,000 | Variable |
| PR diff — large (~1,500 lines) | 20,000 | Variable |

**Output baseline: 2,000 tokens.** This is anchored on a real measurement — the median
output across ~35 GitHub Copilot review sessions on Sonnet 4.6. Two caveats, in opposite
directions:

- This pipeline asks for a **bare findings array**, not conversational review prose, so a
  structured-output run may come in **lower** (perhaps 800–1,200 tokens).
- If extended **thinking** is enabled, thinking tokens bill as output and can push the
  number **at or above** 2,000 even though the final JSON is small.

The demo runs Haiku with thinking **off**, so expect real output below this baseline. We
keep 2,000 as the documented planning figure because planning high is how you avoid
surprises. Output length also varies by model — Opus tends to reason more, Haiku less — so
re-measure per model rather than assuming 2,000 across the board. The evaluator logs actual
`input_tokens` / `output_tokens` to the workflow run; reconcile against this table.

## The formula

```
cost_per_PR =  (input_tokens  / 1,000,000) * input_price
             + (output_tokens / 1,000,000) * output_price

monthly_cost = evaluated_PRs_per_month * cost_per_PR
```

`evaluated_PRs_per_month` is PRs that reach the evaluator — i.e. total PRs minus the ones an
auto-approve rule (e.g. RULE-001) short-circuits at zero LLM cost.

## Cost per PR (output = 2,000 tokens)

| Model | Small (3.6k in) | Medium (6.9k in) | Large (22.9k in) |
|---|---:|---:|---:|
| Haiku 4.5 | $0.014 | $0.017 | $0.033 |
| Sonnet 4.6 | $0.041 | $0.051 | $0.099 |
| Opus 4.8 | $0.068 | $0.085 | $0.165 |

## Monthly cost (medium PRs, output = 2,000)

| Evaluated PRs/month | Haiku 4.5 | Sonnet 4.6 | Opus 4.8 |
|---:|---:|---:|---:|
| 100 | $1.70 | $5 | $8.50 |
| 500 | $8.50 | $26 | $43 |
| 2,000 | $34 | $102 | $170 |

## Two things to internalize

**Output is the dominant lever above ~1.5k output tokens.** At 2,000 output tokens, output
is already the bigger half of every Sonnet/Opus PR (Opus medium: ~$0.035 input vs ~$0.050
output). If you ever need to cut cost, the move is reducing output — keep findings terse
(one-sentence descriptions, which the schema already enforces) and keep thinking off — not
trimming the diff. Trimming the diff only helps the runaway tail below.

**The surprise vector is one pathological diff, not throughput.** A regenerated lockfile, a
vendored dependency dump, or a 50k-line generated file can be ~400k tokens. That single PR
costs roughly $0.40 (Haiku) / $1.20 (Sonnet) / $2.00 (Opus). Normal volume stays in the
tens of dollars per month; a giant diff is what shows up unexplained. Two guardrails:

1. Route generated files and lockfiles to an auto-approve rule (or exclude them from the
   diff) so they never reach the model.
2. Cap the diff — above, say, 50,000 tokens, skip evaluation and flag the PR for manual
   review rather than paying to "read" a generated blob.

## Choosing the model

`EVALUATOR_MODEL` (a repository variable in the application repo) selects the model; it
defaults to `claude-haiku-4-5`. Start there for the demo — ~1–2¢ per PR, low enough to
ignore cost while getting the pipeline working, and it still supports structured outputs so
findings conform to the schema. Move to `claude-sonnet-4-6` or `claude-opus-4-8` for sharper
judgment in production; the tables above tell you what that upgrade costs at your volume.
