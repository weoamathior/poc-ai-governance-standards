# Auto-Approve Rules

This directory defines the conditions under which the pipeline may approve a pull request
without invoking the language model evaluator at all. Each rule lives in its own Markdown
file with structured criteria.

These rules exist to reduce noise and token cost. Some pull requests carry no meaningful
production risk, and running a full AI evaluation on them spends effort and money to
confirm what a simple structural check already makes obvious. An auto-approve rule
captures one such case precisely, so the pipeline can recognize it and move on.

The pipeline consults the auto-approve rules before it invokes the evaluator. If a pull
request matches a rule, the pipeline short-circuits: it approves the pull request under
that rule and skips LLM evaluation entirely. Because a match means no AI evaluation
happens, rules are written to be narrow and unambiguous, matching only pull requests whose
lack of risk is structurally evident. If no rule matches, the pipeline proceeds to normal
evaluation as usual.
