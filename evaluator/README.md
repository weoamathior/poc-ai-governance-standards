# Evaluator

This directory contains the prompt template that the pipeline uses when it asks a
language model to evaluate a pull request. The template lives in
`pr-evaluation-prompt.md`.

The prompt is a policy artifact, not a piece of tooling. It determines how the standards
in this repository are interpreted and applied to real pull requests, which means a change
to its wording can change the pipeline's behavior as surely as a change to a standard
itself. For that reason, changes to the prompt are subject to architecture review.

At runtime the pipeline supplies the prompt with the material it needs to do its work. It
injects the pull request diff being evaluated, the contents of the active standards files
concatenated together as context, and the findings schema that the output must conform
to. The prompt template marks the places where this material is inserted. Because the
standards and schema are passed in at runtime from whatever ref the pipeline has pinned,
the prompt stays general and does not restate the standards inline.
