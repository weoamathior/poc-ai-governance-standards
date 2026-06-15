# Standards

This directory contains the individual standards that a pull request is evaluated
against. Each standard lives in its own Markdown file and is written to be understood
both by a human reviewer and by the language model that performs the evaluation at
runtime. A standard should be self-contained: a reader should not need to consult any
other document to understand what it requires.

Every standard follows the same format so that the evaluator can apply them
consistently. A standard begins with an **ID** (for example, STD-001) that uniquely and
permanently identifies it, and a short **title** that names the concern in plain
language. The **description** explains the expectation the standard encodes and why it
matters.

Each standard carries a **severity level**, which is one of three values. A severity of
*advisory* means a violation is worth surfacing but does not block a merge. A severity of
*required* means a violation represents a genuine gap that should be addressed. A
severity of *escalate* means a violation needs human judgment beyond the pipeline and
should be routed to a person rather than resolved automatically. The severity assigned to
a standard is the severity that appears on any finding the evaluator produces against it.

Each standard then provides **evaluator guidance**, which describes concretely what the
model should look for in a pull request diff. This guidance is written for the evaluator,
not for the developer, and should be specific enough to reduce guesswork about when the
standard applies.

Finally, each standard includes a section on **what does not constitute a violation**.
This section exists to reduce false positives. Because the evaluator is instructed to be
conservative, clearly naming the cases that look like violations but are not helps keep
the pipeline's output trustworthy and low-noise.
