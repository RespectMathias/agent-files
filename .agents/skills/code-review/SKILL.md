---
name: code-review
description: Review code changes for correctness, regressions, security, and other concrete problems. Use when the user asks for a code review, PR review, commit review, or review of current changes. Run the review in fresh subagents so the reviewer does not inherit the implementation context.
---

# Review

Review the requested changes. Do not modify code unless the user explicitly asks for fixes.

## 1. Determine scope

Use the target the user gave.

If no target was given, review current uncommitted changes.

Read repository instructions and the relevant diff before dispatching the review. Include enough surrounding code to understand the change, not only isolated diff hunks.

## 2. Independent review

Spawn a fresh subagent with no authoring context.

Give it the review target and instruct it to act as a skeptical reviewer who did not write the code and assumes the change may contain defects.

It must:

* Read the complete diff.
* Read surrounding implementation where needed.
* Trace callers, callees, types, invariants, and related tests when they affect changed behavior.
* Check deleted or replaced logic and determine where its previous guarantees are now enforced.
* Focus on correctness, regressions, security, data loss, concurrency, API contracts, error handling, meaningful performance problems, and structural regressions.
* Check test coverage when missing coverage could let a concrete regression escape.
* Run existing focused tests, type checks, linters, or builds when useful and reasonably scoped.
* Report only problems introduced or exposed by the reviewed change.
* Prefer root causes over symptoms.

Also review the implementation aggressively for unnecessary complexity and structural regressions.

Look for a simpler framing that deletes branches, special cases, wrappers, duplicated logic, or layers rather than merely rearranging them. Be suspicious of feature logic leaking into shared paths, bespoke helpers that duplicate canonical utilities, unnecessary optionality or casts, thin abstractions, ad-hoc conditionals, and logic living in the wrong layer.

Prefer direct, boring code and existing architecture over new machinery. When a meaningful "code judo" move can make the implementation substantially simpler without changing behavior, report it.

Structural findings still need a concrete cost: increased coupling, duplicated behavior, obscured invariants, unnecessary states or branches, wrong ownership, harder future changes, or another demonstrable maintainability problem. Do not report a refactor solely because it is more elegant.

For every finding require:

* severity
* file and line
* concrete problem
* failure scenario or structural consequence: specific input, state, timing, condition, or code evolution that exposes the problem and its concrete effect
* evidence from code or a deterministic check
* smallest reasonable fix
* test or check that would catch the regression when applicable

Do not report:

* formatter or style issues
* subjective refactors or alternative designs with no concrete structural cost
* generic best-practice advice
* missing comments or documentation without a concrete consequence
* pre-existing problems unrelated to the change
* descriptions or praise of what the change already does
* speculative suggestions without a concrete failure scenario or cost

Silence is better than noise. A clean review may contain no findings.

Do not suppress a suspected merge-blocking defect solely because confidence is incomplete. Mark it low confidence for verification.

## Examples

Good findings are specific, reproducible, and tied to changed behavior.

**Bad**

> `src/posts.ts:42` - Fix this query. It may be inefficient.

No concrete failure or evidence.

**Good**

> **[Medium]** **`src/posts.ts:42`** **- Loads each author with a separate query**
>
> * Problem: The new loop fetches each post's author individually.
> * Failure scenario: Listing 100 posts performs 101 database queries instead of a bounded query count, increasing request latency with result size.
> * Evidence: `getAuthor(post.authorId)` executes inside the loop and performs a database query on every call.
> * Suggested fix: Load the required authors in one query or use the repository's existing eager-loading path.
> * Regression check: Query-count test for a multi-post response.

Do not manufacture a finding merely to produce feedback.

**Bad**

> `src/parser.ts:18` - This could be refactored into a helper for readability.

**Good**

No finding if the changed code is correct and the proposed refactor has no concrete behavioral, security, reliability, or meaningful performance consequence.

A verifier must challenge rather than restate a candidate finding.

**Candidate**

> `src/cache.ts:61` can return stale data after an update because the cache is not invalidated.

**Confirmed**

> The update path writes to storage and returns without deleting or replacing the matching cache entry. The next read hits that entry until its TTL expires.

**Rejected**

> `updateRecord()` calls `invalidateRecordCache(id)` through `afterRecordWrite()` before returning. This covers the stated update path, so the claimed stale-read scenario does not occur.

## 3. Verify findings

If the reviewer reports no findings, skip verification.

Otherwise spawn a second fresh subagent. Give it the review target and the candidate findings, but not the first reviewer's reasoning beyond what each finding states.

Its job is adversarial: assume each finding may be a false positive and try to disprove it.

For every candidate:

1. Trace the stated failure scenario or structural consequence through the actual code.
2. Inspect relevant surrounding code, callers, tests, and invariants.
3. Check whether the diff already prevents the claimed failure or structural problem.
4. Run a focused deterministic check when that can settle the question.
5. Return one verdict:

   * `confirmed`
   * `rejected`
   * `needs human review`

Reject only with concrete counter-evidence, such as a guard that covers the stated trigger, an impossible state proven by the code, a misread API contract, evidence that the structural consequence does not exist, or evidence that the issue is outside the reviewed change.

Failure to reproduce or inability to find evidence is not by itself grounds for rejection. Use `needs human review` when evidence remains insufficient.

Build, test, type-check, or lint failures caused by the change are already deterministic evidence and do not need a second opinion.

## 4. Report

Deduplicate findings with the same root cause.

Report confirmed findings first, ordered by severity.

For each include:

**[Severity]** **`file:line`** **- short title**

* Problem
* Failure scenario or structural consequence
* Evidence
* Suggested fix
* Regression check

Put unresolved findings under `Needs human review`.

Do not include rejected findings except for a short verification count such as:

`4 candidates reviewed, 2 confirmed, 1 rejected, 1 needs human review.`

If nothing survives verification, say no actionable issues were found.

Keep the final review concise. Do not pad it with positive observations or generic commentary.