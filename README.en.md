# Coding Agent Lean Review

[中文](README.md) | English

A code simplification skill for coding agents, intended for a lightweight review after a feature or bug fix.
It helps identify duplicate validation, overlapping fallbacks, abstractions without a current purpose,
and error handling that conceals defects, while preserving required behavior and error contracts.

**One quick pass, with ablation on demand: inspect once, investigate one complex unit when needed, and stop after sufficient validation.**

The repository contains one skill with two working modes and shared protection boundaries:

| Skill / mode | Use it for | Main output |
| --- | --- | --- |
| `coding-agent-lean-review` · Quick simplification | Reviewing newly added or modified code for clear redundancy | Evidence-backed simplifications and validation notes in the normal task response |
| `coding-agent-lean-review` · Targeted ablation | Investigating a complex defensive unit whose necessity is unclear | One isolated experiment; retain supported simplifications and restore failed or inconclusive changes |

Install once to use both modes. Start with quick inspection and enter ablation only when its conditions apply.
If no clear candidate exists, or uncertainty does not justify more investigation, preserve the implementation and finish.

## Contents

- [Workflow](#workflow)
- [Repository conventions](#repository-conventions)
- [Features](#features)
- [Installation](#installation)
- [Usage examples](#usage-examples)
- [Local review quickstart](#local-review-quickstart)
- [Verification](#verification)
- [GitHub Pages](#github-pages)
- [Data and operation boundaries](#data-and-operation-boundaries)
- [Frequently asked questions](#frequently-asked-questions)

## Workflow

```text
Code added or modified in the current task
  └─ coding-agent-lean-review
       ├─ Reuse context, diffs, and validation results
       ├─ Make one quick pass
       │    ├─ Duplicate validation and overlapping protections
       │    ├─ Broad exceptions, silent defaults, and fallback layers
       │    └─ Wrappers, factories, and compatibility without a current purpose
       ├─ Inspect contracts, call paths, or equivalent implementations
       │    ├─ Clear evidence → simplify
       │    └─ Insufficient evidence → preserve
       ├─ Run one local ablation if necessary
       │    ├─ State a concrete removal hypothesis
       │    ├─ Check an isolated local variant
       │    └─ Accept supported changes; restore failed or inconclusive ones
       └─ Include in normal validation and finish briefly
```

Quick inspection identifies candidates and resolves obvious redundancy.
Ablation answers one question that cannot be resolved cheaply by inspection.
Interpret the result using behavior or invariant evidence; a green test suite alone does not justify removal.

After accepting a removal, base subsequent decisions on the updated implementation.
Two validation layers may each appear redundant when considered separately,
but deleting both can eliminate the guarantee entirely.
Combined changes must leave at least one effective layer for every required guarantee.

## Repository conventions

- Root [SKILL.md](SKILL.md) is the source of execution rules and retains the English instructions.
- [README.md](README.md) and this file provide complete Chinese and English documentation.
- Use `coding-agent-lean-review` for both the directory and skill name.
- Documentation examples explain decisions; they do not require an inventory or table in every task.
- Keep both language versions aligned when changing budgets, validation, or stopping conditions.
- Add scripts, references, or other resources only when an actual need emerges.

```text
coding-agent-lean-review/
├── SKILL.md       # Agent execution rules
├── README.md      # Complete Chinese documentation
└── README.en.md   # Complete English documentation
```

This is a Markdown-only skill. It includes no analysis engine, database, automated test service, or deployment pipeline.
The host agent performs reading, editing, and checks inside the target project.

## Features

### Quick simplification

- Review only code added or modified in the current task.
- Read direct dependencies only to answer a specific question and reuse existing context.
- Identify validation already guaranteed along every supported call path.
- Inspect broad exception handling for hidden defects and defaults that swallow required errors.
- Check whether fallback and retry layers serve overlapping purposes.
- Identify compatibility branches, factories, wrappers, and extension points without a current requirement or meaningful repository role.
- For meaningful candidates, ask what they protect, what removal breaks, and whether another layer provides the same guarantee.
- Simplify with concrete contract or call-path evidence; missing tests do not prove redundancy.

### Targeted module-level ablation

- Reserve experiments for units that add meaningful complexity and cannot be assessed cheaply by inspection.
- State one concrete removal hypothesis before changing the unit.
- Temporarily remove or simplify it in an isolated local variant.
- Select the smallest relevant check that exercises the protected behavior.
- Combine observations with input contracts and invariants rather than relying only on a passing suite.
- Restore experiment changes when they fail or remain inconclusive.
- Default to at most one experiment per task; expand only for a requested deeper review or correctness.
- Never experiment against live data or irreversible side effects.

Ablation means removing a unit to investigate its necessity.
It does not mean repeatedly deleting code until tests fail.
Skip the experiment when inspection already answers the question.

### Necessary protections and execution budget

| Area | Default rule |
| --- | --- |
| Identity and permissions | Preserve authentication and authorization unless the task or direct invariant evidence supports a change |
| Data and execution | Preserve integrity, concurrency, idempotency, and resource cleanup |
| Input boundaries | Keep validation proportionate to actual external-input risks |
| Runtime guarantees | Annotations are not runtime checks; upstream guarantees must remain valid until use |
| Reading | Avoid repository rescans and unexplained rereading |
| Tests | Reuse results and add checks only for concrete unresolved risks |
| Output | No review documents, risk tables, or per-unit reports |
| Stopping | Finish when no clear candidate exists, evidence is insufficient, or validation is sufficient |

## Installation

### Get the repository

```bash
git clone https://github.com/XIIRUAN/coding-agent-lean-review.git
cd coding-agent-lean-review
```

The skill has no runtime dependencies. Cloning requires Git.
Prepare the target project's test tools, compilers, and package managers according to that project's requirements.

### Codex: user-level installation

The following commands clone directly into the user skill directory for use across projects.
Choose this option or the ordinary clone above; both are not necessary.

macOS / Linux:

```bash
mkdir -p "$HOME/.agents/skills"
git clone https://github.com/XIIRUAN/coding-agent-lean-review.git \
  "$HOME/.agents/skills/coding-agent-lean-review"
```

Windows PowerShell:

```powershell
$skillRoot = Join-Path $HOME '.agents\skills'
New-Item -ItemType Directory -Force -Path $skillRoot | Out-Null
git clone https://github.com/XIIRUAN/coding-agent-lean-review.git `
  (Join-Path $skillRoot 'coding-agent-lean-review')
```

Codex uses `~/.agents/skills/` for user skills and `.agents/skills/` for repository skills.
See the [official OpenAI Skills documentation](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills).

### Project-level installation and other hosts

For one project, create `.agents/skills/coding-agent-lean-review/` at the target repository root
and copy this repository's `SKILL.md` into it.
Optionally copy the README files for teammates; do not copy this repository's `.git` directory.

Other hosts should use the discovery location and invocation syntax documented by that host.
The resulting layout should contain `coding-agent-lean-review/SKILL.md`.

### Updates and discovery checks

For a clone-based installation, run inside its installation directory:

```bash
git pull --ff-only
```

For a copied installation, replace the corresponding `SKILL.md` and update any retained documentation.
If the host cannot discover it, inspect directory nesting, the filename, and the skill selector;
start a fresh session if necessary.
Installation makes the skill discoverable; this repository registers no mandatory after-edit hook.

## Usage examples

### Simplify after completing a feature

A natural-language request:

```text
The feature is implemented. Quickly check the new code for duplicate validation,
overlapping fallbacks, or wrappers without a current purpose.
Preserve required error behavior and keep the review within this task.
```

Explicit invocation:

```text
Use $coding-agent-lean-review on the changes made in this task.
Reuse existing context and test results. Make one quick simplification pass.
Stop if there is no clear candidate and include meaningful changes in the normal summary.
```

### Investigate a complex defensive unit

```text
Use $coding-agent-lean-review on the retry wrapper added in this task.
First determine whether the underlying client already covers the same failures
and error contract. If inspection cannot resolve this cheaply and the wrapper
adds meaningful complexity, run one ablation in an isolated local variant.
Use the smallest check covering retry behavior. Restore inconclusive changes,
and do not operate against production services or real business data.
```

### Combine with a bug fix

```text
Fix the current parsing error while preserving existing error types and public interfaces.
Then use $coding-agent-lean-review for one simplification pass.
Include accepted changes in normal validation and state which checks could not run.
```

“Current task” means the changes belonging to this request, not automatically the entire branch history,
repository, or another person's edits.
When only a diff is available, work within that evidence and state dependency-related limitations.

## Local review quickstart

### 1. Identify the target changes

These optional commands help a person locate changes.
An agent that already has the same information should reuse it.

```bash
git status --short
git diff --stat
git diff --cached --stat
```

Select files belonging to the task before inspecting detailed differences.
Ordinary `git diff` omits untracked files; read relevant new files using task context.

### 2. Supply existing validation results

Provide the checks actually run and their outcomes, such as “parser tests passed; integration tests have not run.”
Use the target repository's established commands rather than treating an example as a universal test entry point.

### 3. Invoke the skill

```text
Use $coding-agent-lean-review.
Scope: the parser changes just completed, plus direct dependencies needed to answer specific questions.
Reuse this task's validation and add checks only for unresolved regression risks.
```

### 4. Decide from evidence

| Example | What to establish |
| --- | --- |
| A private helper repeats input validation | Check all supported callers, mutable state, and error contracts |
| A public function also has direct callers | One validated caller cannot justify removing its boundary checks |
| Two layers retry the same request | Check attempt counts, retryable failures, idempotency, and final errors |
| A one-line cleanup inside `finally` | Brevity does not remove the need to release resources |

This table is instructional, not a required output inventory.
The default deliverable remains code changes and a brief explanation.

## Verification

### Documentation and skill structure

There is no application to install or execute and no bundled `tests/verify-all.sh`.
When maintaining documentation, run from the repository root:

```bash
git diff --check
```

This detects whitespace issues in changes; it does not evaluate skill behavior.
Also check README links, installation paths, and consistency with the skill name and description.
The current `SKILL.md` passed the creation environment's structural validator;
that validator is not distributed in this repository.

### Target-project behavior

- Include accepted simplifications in the original task's final validation.
- Add targeted checks only for a specific unresolved regression risk.
- For error handling, check error types, propagation, and caller-observable behavior.
- For retries, check the protected failure modes, idempotency, and side-effect boundaries.
- Reassess necessary guarantees after combining multiple removals.
- Honor required repository checks and state execution limitations when checks cannot run.

### When to finish

Stop when required behavior has sufficient validation and no new concrete risk remains.
Making no deletion can be a valid outcome; do not seek extra edits to demonstrate activity.
Expected behaviors in these examples explain the decision criteria.
They are not claims of completed cross-project benchmarks or effectiveness evaluations.

## GitHub Pages

The documentation entry point is the [GitHub repository](https://github.com/XIIRUAN/coding-agent-lean-review).
No GitHub Pages site is currently configured, and the skill does not automatically generate or publish review reports.

| Content | Current location / behavior |
| --- | --- |
| Chinese documentation | `main:README.md` |
| English documentation | `main:README.en.md` |
| Agent execution rules | `main:SKILL.md` |
| Review results | Code changes in the target project and the normal task response |
| Pages deployment | Not configured; no published site URL |

A static documentation site can be configured separately if needed later.
Website presentation should not change the skill's default rule against additional review reports.

## Data and operation boundaries

- Limit review to the current task and direct dependencies needed for specific questions.
- Invoking the skill does not grant extra authorization to upload code, publish repositories, or call external services.
- Run ablation only in isolated local variants, never against production data or irreversible side effects.
- Restore only experiment changes and preserve the user's existing, unrelated work.
- Preserve authentication, authorization, integrity, concurrency, idempotency, and cleanup by default.
- Keep external-input validation and error handling proportionate to actual risks.
- Missing tests, passing tests, type annotations, and short code cannot independently establish redundancy.
- The host controls execution permissions and data handling; a Markdown skill provides no sandbox of its own.

## Frequently asked questions

### Why provide detailed documentation for a lightweight workflow?

The README helps people install and understand the skill; [SKILL.md](SKILL.md) guides execution.
Detailed documentation does not require rereading this file, scoring every guard, or producing a report on each run.

### Does it automatically delete defensive code?

No. It requires a concrete understanding of requirements and failure modes before accepting simplifications.
Necessary protections and candidates with insufficient evidence remain in place by default.

### Does every run require ablation or new tests?

No. Ablation has complexity and evidence thresholds, and additional tests must address unresolved regression risks.
Otherwise, one quick inspection and normal validation are sufficient.

### Which languages are supported, and how much does it save?

The rules are language-independent, but analysis depends on the host agent, project context, and available tools.
The repository publishes no language coverage matrix or quantified token, time, or line-count savings.

### How can I report a problem?

Open an [issue](https://github.com/XIIRUAN/coding-agent-lean-review/issues) with a minimal shareable example:
the requirement, changed scope, incorrect removal or unnecessary retention, and relevant validation evidence.
Improvements should address observed failures while preserving the quick-review and on-demand-ablation boundaries.
