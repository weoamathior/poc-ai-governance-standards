# STD-003: Dependency Hygiene

**ID:** STD-003
**Title:** Dependency Hygiene
**Severity:** escalate

## Description

A new dependency should not duplicate functionality that the existing dependency tree
already provides. The classic example is adding a second HTTP client to a project that
already has one: the result is two ways to do the same thing, more surface area to secure
and maintain, and inconsistency in how the codebase makes network calls. Because deciding
whether a new dependency genuinely earns its place requires judgment about the existing
architecture, violations of this standard are routed to a person rather than resolved
automatically.

## Evaluator Guidance

Watch for modifications to `pom.xml` or `build.gradle` that introduce a dependency under
a new `groupId` not already present in the project. When a new `groupId` is introduced,
flag the pull request under this standard at the *escalate* severity and name the specific
`groupId` and artifact in the finding, so that the reviewer who picks up the escalation
can immediately see which dependency is in question and assess whether it overlaps with
something the project already has.

## What Does Not Constitute a Violation

The following must not be flagged under this standard:

Version bumps to dependencies that are already present in the tree, since they introduce
no new `groupId` and no new capability. Test-scoped dependencies, which do not become part
of the production dependency surface and are evaluated under different considerations. A
change that only adjusts an existing dependency, rather than introducing a new `groupId`,
is outside the scope of this standard.
