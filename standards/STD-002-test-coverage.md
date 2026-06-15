# STD-002: Test Coverage Signals

**ID:** STD-002
**Title:** Test Coverage Signals
**Severity:** advisory

## Description

A pull request that introduces new service-layer or controller-layer logic should also
introduce the tests that exercise that logic. This standard does not attempt to measure a
coverage percentage or enforce a numeric threshold; it looks instead for the simple
signal that new behavior arrived together with tests for that behavior. The intent is to
catch the common case where logic is added and the corresponding tests are forgotten,
while remaining advisory so that legitimate exceptions do not block a merge.

## Evaluator Guidance

Examine the diff for new methods added to a service or controller class. If new methods
appear in such a class and no test file is added or modified anywhere in the same pull
request, flag the pull request under this standard at the *advisory* severity. The signal
is the absence of any test change accompanying new service or controller methods — you are
not assessing whether the tests that exist are good, only whether the pull request
brought tests along with the new logic at all.

## What Does Not Constitute a Violation

The following must not be flagged under this standard:

Pull requests that are purely configuration changes, which introduce no new logic to
test. Dependency version bumps, which change what the application depends on rather than
how it behaves. Stub updates and similar changes that add no new service or controller
behavior. In each of these cases there is no new logic, so the absence of new tests is
expected and correct.
