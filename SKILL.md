---
name: code-trim
description: >
  Perform a lightweight simplification pass after generating or modifying
  code. Remove redundant defenses and speculative abstractions while
  preserving required behavior. Use targeted ablation only when justified.
---

# Code Trim

Prefer the smallest implementation that satisfies current requirements
and protects concrete failure modes. Do not add defenses for hypothetical
future needs or remove code merely to reduce line count.

## Scope and Budget

- Review only code added or modified in the current task and its direct
  dependencies when needed to resolve a specific question.
- Reuse existing context, diffs, and validation results for unchanged code.
  Do not rescan the repository or reread files without a concrete reason.
- Make one quick pass. Prioritize obvious, high-value simplifications;
  do not inventory or score every defensive statement.
- Do not create review documents, risk tables, or per-unit reports.
- If no clear candidate exists, stop without changing code.

## Quick Review

Look for:
- Repeated validation already guaranteed on every supported call path.
- Broad exception handling that hides defects, silent defaults, and
  overlapping fallback or retry layers.
- Compatibility branches, factories, wrappers, and extension points
  without a current requirement or meaningful repository role.

For each meaningful candidate, determine:
1. Which current failure mode or requirement does it address?
2. What would break if it were removed?
3. Is the same guarantee already provided elsewhere?

Simplify when a concrete contract, call path, or equivalent implementation
supports the change. A short guard is not automatically necessary;
missing tests are not evidence that it is unnecessary.

Type annotations alone are not runtime validation. Confirm upstream
guarantees cover supported callers and remain valid until use.

## Preserve Necessary Defenses

Keep authentication, authorization, data-integrity, concurrency,
idempotency, and resource-cleanup protections by default. Change them
only when required by the task or supported by direct invariant evidence.

Keep external-input validation and failure handling proportionate to
real boundary risks. Prefer specific errors over unjustified silent
fallbacks. Preserve required observable behavior and error contracts.

If evidence remains insufficient, leave the candidate unchanged rather
than expanding the review or guessing.

## Optional Module-Level Ablation

Use ablation only for a module or defensive unit that adds meaningful
complexity and whose necessity cannot be resolved cheaply by inspection.

- State one concrete removal hypothesis.
- Temporarily remove or simplify that unit in an isolated local variant.
- Run the smallest relevant check that exercises the protected behavior.
- Keep the simplification only with supporting behavioral or invariant
  evidence; a passing suite alone does not prove redundancy.
- Restore unsuccessful or inconclusive changes.

Default to at most one ablation experiment per task. Broaden only when
the user requests deeper review or correctness requires further work.
Never experiment against live data or irreversible side effects.

## Validate and Finish

- Include simplifications in the task's normal final validation.
- Add targeted checks only for a specific unresolved regression risk;
  do not rerun broad suites after every trivial edit.
- After accepting a removal, use the updated implementation for further
  decisions. Verify combined edits retain at least one effective layer
  for each required guarantee.
- Honor required repository checks. If execution is unavailable, state
  that limitation and do not claim runtime verification.
- Stop once relevant validation is sufficient. Do not iterate to meet
  a deletion target.
- In the normal final response, briefly mention meaningful simplifications
  and validation. If nothing changed, omit a separate review report.
