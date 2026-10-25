---
name: zero-bug-developement
description: Defect-prevention software development workflow inspired by NASA practices. Use when building, reviewing, or refactoring code where bugs are costly (money, auth, data, safety, production releases). Covers requirements traceability, risk-scaled rigor, bounded and simple code rules, static analysis gates, test-first development, independent review checklists, and release criteria. Triggers include "zero bug", "bug-free", "production ready", "harden this", "review before merge", "make this reliable", "NASA-style", or any request to reduce defects in a feature or codebase.
---

# Zero-Bug Development

Goal: ship with zero *known* defects and remove the classes of bugs that cause real damage. Literal zero bugs is not a realistic target. The method is layered prevention so that whatever slips through is rare, small, and caught early.

## Core principles

1. **Prevent before you detect.** Make wrong code hard to write (types, narrow APIs, invalid states unrepresentable) before adding checks after the fact.
2. **Every requirement traces to a test, and every test traces to a requirement.** Unmapped items are gaps.
3. **Rigor scales with risk.** Money, auth, personal data, and irreversible actions get the full process. A throwaway script gets a lighter one.
4. **The author is not the only reviewer.** Something independent checks the work: a second person, a checklist, or a tool.
5. **Warnings are errors.** Anything the compiler, linter, or analyzer flags must be fixed or explicitly justified in writing.

## Step 1: Classify risk

Before writing code, assign a tier and state it in one line.

| Tier | Examples | Required rigor |
|---|---|---|
| Critical | payments, auth, permissions, personal data, deletes, money math, safety | Everything below, plus independent review and failure-mode analysis |
| High | core business logic, data migrations, public APIs, integrations | Everything below, plus checklist review |
| Standard | internal features, UI flows, reports | Requirements trace, tests, static analysis, self-review checklist |
| Low | scripts, prototypes, one-off tools | Static analysis and smoke tests; mark clearly as non-production |

## Step 2: Write the requirements

For the feature, write:
- **Requirements**: numbered, testable statements. "REQ-03: A refund cannot exceed the original charge." Vague wording like "should be fast" must be rewritten with a measurable threshold.
- **Non-goals**: what this does not do.
- **Invariants**: rules that must always hold (e.g., "balance never goes negative", "every order has exactly one owner").
- **Failure modes**: what happens on network loss, duplicate input, partial writes, timeouts, and bad data. Each gets a defined behavior.

Do not write code until the requirements are numbered and each has a planned test.

## Step 3: Design for simplicity and bounded behavior

Apply these rules to all new code, adapted to the language:

- No unbounded recursion. Use iteration with an explicit limit.
- Every loop has a known upper bound or a timeout.
- Every external call has a timeout, a retry limit, and an idempotency key when it writes.
- Check every return value and every error. Do not swallow exceptions.
- Keep functions short enough to read in one screen. Split by responsibility.
- Keep scope tight: declare variables at the smallest scope, avoid shared mutable state.
- Validate inputs at the boundary. Inside the core, assume validated data.
- Make illegal states unrepresentable: use enums, tagged unions, or value objects instead of loose strings and booleans.
- Prefer pure functions for logic. Isolate side effects (I/O, clock, randomness) behind small interfaces that tests can replace.
- Avoid clever code. If a reviewer needs more than 30 seconds to follow a function, rewrite it.

## Step 4: Test first

For each requirement, in this order:

1. Write a failing test that encodes the requirement, including the failure-mode and invariant cases.
2. Confirm it fails for the right reason.
3. Write the minimum code to pass.
4. Refactor with tests green.

Required test categories:
- Happy path for each requirement
- Boundary values (zero, one, max, max+1, empty, null)
- Each invariant, tested by trying to break it
- Each failure mode from Step 2
- For Critical/High: property-based or fuzz tests on parsers, money math, and state transitions
- Regression test for every bug fixed, written before the fix

Coverage is a floor, not a proof. Track branch coverage on Critical code and treat untested branches as risk.

## Step 5: Static analysis and automated gates

Every change must pass these before review:

- Compiler or type checker in strict mode, warnings treated as errors
- Linter with the rules above enabled (unused variables, unchecked returns, unreachable code, shadowing)
- Security scanner for injection, hardcoded secrets, unsafe deserialization, and dependency CVEs
- Formatter, so review focuses on logic
- Full test suite in CI, with no skipped tests in the merge branch

A failing gate blocks the merge. Do not disable a rule to pass a gate. If a rule is wrong for one line, use a narrow, commented suppression that a reviewer can see.

## Step 6: Independent review

Reviewer must not be the author. If no second person exists, run the checklist as a separate pass and record it.

Review checklist:

- [ ] Every requirement has a test, and the test asserts the requirement (not the implementation)
- [ ] All inputs are validated at the boundary
- [ ] Every error path is handled and tested
- [ ] No unbounded loops, recursion, or external calls without timeouts
- [ ] Writes are idempotent or protected against duplicates
- [ ] No secrets, tokens, or personal data in logs or source
- [ ] Authorization is checked on the server, not only in the UI
- [ ] Money and numeric code uses exact types (integer minor units or decimal), not floats
- [ ] Concurrency: shared state is protected, and races are considered on read-then-write
- [ ] Time and timezone are explicit and injectable for tests
- [ ] Migrations are reversible or have a tested rollback
- [ ] Dependencies are pinned and the new ones are justified
- [ ] Failure modes from Step 2 each have a defined, tested response

Reviewers write findings as: `file:line | severity (blocker / major / minor) | problem | required change`. Blockers must be resolved before merge. No "LGTM" without the checklist filled in for Critical and High tiers.

## Step 7: Verify beyond unit tests

- **Integration tests** against real databases and services (or faithful containers), not only mocks, for anything touching persistence or external APIs.
- **End-to-end tests** for each user-facing requirement's main flow.
- **Chaos checks** for Critical and High: kill the dependency mid-request, duplicate the request, send malformed and oversized input, and confirm the defined failure mode happens.
- **Staging soak** for anything with money or data: run realistic traffic and compare outputs against an independent calculation.

## Step 8: Release criteria

Ship only when all are true:

- Every requirement is traced to passing tests
- Zero open blockers; majors have an owner and a date
- All gates green on the release commit
- Failure-mode tests pass
- Rollback path is written down and tested
- Monitoring and alerts exist for the invariants (e.g., alert on negative balance, failed payment spikes, error rate)
- Known limitations are documented for users or support

## Step 9: Post-release defect loop

Every production bug gets:
1. A regression test that reproduces it
2. A root cause classified as: requirement gap, design gap, test gap, review miss, or tooling gap
3. A process fix aimed at that class, not only the instance

Track defect escape rate (bugs found in production per release) and aim for it to fall over time.

## Output formats

When applying this skill, produce these artifacts as needed:

**Feature brief:** tier, requirements (numbered), invariants, failure modes, non-goals, open questions.

**Traceability table:**

| Req | Test(s) | Status | Notes |
|---|---|---|---|
| REQ-01 | test_x, test_y | passing | |

**Review report:** checklist results plus findings in the `file:line | severity | problem | fix` format.

**Release sign-off:** release criteria with pass/fail per item, and the list of known risks.

## Anti-patterns to refuse or flag

- Declaring code "bug-free" without tests tied to requirements
- Disabling a lint or security rule to get a green build
- Floats for money
- Auth checks only in the frontend
- Catch-all exception handlers that log and continue
- Retries on non-idempotent writes
- Tests that mock the thing under test
- Shipping Critical code without an independent review
- Fixing a production bug without a regression test

## Adapting to small teams

If there is one developer, the minimum that removes most damage is: strict compiler and linter in CI, requirement-to-test mapping in a single table, a written checklist run before every merge to Critical or High code, and a regression test for every bug. Add the rest as the product grows.
