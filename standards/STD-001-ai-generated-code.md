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

Look for substantial blocks of logic that have been added without any accompanying
provenance comment, paying particular attention to service-layer and controller-layer
code, where business logic concentrates and where missing provenance carries the most
risk. A "substantial block" means meaningful logic — a method body, a non-trivial branch
of control flow, an algorithm — rather than a single line or a trivial accessor. When such
a block appears in the diff with no provenance comment and its shape suggests it may have
been generated, flag it under this standard at the *required* severity.

## What Does Not Constitute a Violation

The following do not require a provenance comment and must not be flagged under this
standard:

Test data files and fixtures, which contain data rather than production logic. Mock stubs
used to stand in for real collaborators during testing. Generated output from code
generators such as protobuf or OpenAPI, which is mechanically produced from a schema and
is not authored logic in the sense this standard is concerned with.
