# poc-ai-governance-standards

This repository is the authoritative source of the standards used by the AI-assisted
pull request evaluation pipeline. It does not run anything itself. Instead, a CI/CD
pipeline in a separate application repository reads the contents of this repository
and uses them to evaluate pull requests before they are merged.

Because the standards defined here directly shape automated decisions about production
code, this repository is treated as a policy artifact rather than a developer tooling
project. It contains no scripts, no bootstrap tooling, and no IDE configuration. Changes
to anything in this repository are subject to architecture review, and reviewers should
treat a change to a standard with the same seriousness as a change to the pipeline that
enforces it.

The repository separates four distinct concerns, each in its own directory.

The `standards/` directory holds the individual standards that a pull request is
evaluated against. Each standard describes a single expectation, the severity of a
violation, what the evaluator should look for, and — importantly — what does not count
as a violation. There are three standards today: AI-generated code provenance
(STD-001), test coverage signals (STD-002), and dependency hygiene (STD-003).

The `evaluator/` directory holds the prompt template that the pipeline uses when it
calls the language model to evaluate a pull request. The prompt is itself a policy
artifact: it determines how the standards are applied, so changes to it are reviewed
with the same rigor as changes to the standards themselves.

The `findings-schema/` directory defines the structure of a single evaluation finding.
Every finding the pipeline produces must conform to this schema, because findings are
written to the audit log of the target repository and must remain machine-readable and
durable over time.

The `auto-approve-rules/` directory defines narrow conditions under which the pipeline
may approve a pull request without invoking the language model at all. These rules exist
to avoid spending evaluation effort and token cost on changes that carry no meaningful
production risk.

This repository is versioned, and the pipeline pins to a specific git ref when it runs.
That means the exact standards, prompt, schema, and rules that were in effect at the
moment any pull request was merged can be reconstructed forensically from the ref
recorded alongside the merge. Treat every commit here as part of a permanent governance
record.
