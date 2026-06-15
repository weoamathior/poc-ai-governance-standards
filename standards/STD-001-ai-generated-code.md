# STD-001: AI-Generated Code Provenance

**ID:** STD-001
**Title:** AI-Generated Code Provenance
**Severity:** required

## Description

Code that was generated or substantially drafted with AI assistance must carry a
provenance comment identifying it as such. The provenance comment should indicate that
the code was AI-assisted, name the tool that produced it, and give a brief description of
what was generated. The purpose of this standard is to preserve a clear record of which
parts of the codebase originated from AI assistance, so that those sections can be
located, reviewed, and reasoned about deliberately rather than discovered by accident
later. A provenance comment is a small cost that protects the long-term auditability of
the codebase.

A conforming provenance comment, placed immediately above the generated block, names the
assistance, the tool, and the substance of what was generated — for example, a note that
a retry-with-backoff handler for the payment gateway client was AI-assisted using a named
tool.

## Evaluator Guidance

Flag a method under this standard, at the *required* severity, only when **both** of the
following hold:

1. **The body contains real business logic** — a conditional, a loop, a calculation, or
   multi-step processing. A trivial member does not qualify: an empty body, a one-line
   delegating call, a plain return of a constant or an empty collection, or a getter or
   setter is never a violation.
2. **No provenance comment sits immediately above the method.** If a provenance comment is
   present, the method is **not** a violation — that is the entire purpose of the standard,
   and it must not be flagged regardless of anything else about the code.

Both conditions must be true. If either fails — the body is trivial, or a provenance
comment is present — do not flag.

Do not judge whether the code "looks" AI-generated; that is guesswork and produces
inconsistent results. Judge only the two conditions above. Apply this to service-layer and
controller-layer classes, where business logic concentrates; code in test classes is out
of scope (see the non-violations below).

## What Does Not Constitute a Violation

The following do not require a provenance comment and must not be flagged under this
standard:

Test code of any kind — unit and integration test classes, individual test methods, test
data files, and fixtures. Provenance is required for production logic in the service and
controller layers, not for anything under a test source tree, so a change confined to a
test class is never a violation of this standard even when it contains assertions or setup
logic. Mock stubs used to stand in for real collaborators during testing. Generated output
from code generators such as protobuf or OpenAPI, which is mechanically produced from a
schema and is not authored logic in the sense this standard is concerned with.
