# Coding Agent Lean Review

[中文](README.md) | English

A lightweight simplification skill for coding agents. After generating or modifying code, review redundant defenses and abstractions without a current purpose. Use targeted ablation only when necessary.

**One quick pass, with ablation on demand.** Preserve real requirements and necessary protections without pursuing a deletion target.

## When to Use

- Check a completed feature or fix for duplicate validation, fallbacks, or retries introduced in the current task.
- Simplify factories, wrappers, compatibility branches, and extension points without a current purpose.
- Evaluate a defensive unit that adds meaningful complexity when inspection cannot cheaply resolve its necessity.

## Workflow

```text
Code added or modified in this task
  → One quick review
  → Inspect contracts, call paths, or equivalent implementations
  → Simplify with evidence; preserve when uncertain
  → At most one local ablation experiment when needed
  → Include in normal validation and finish briefly
```

Ablation means temporarily removing or simplifying one unit in an isolated local variant, then running the smallest relevant check that exercises the protected behavior. Restore failed or inconclusive experiments. Passing tests alone do not prove redundancy.

## Scope and Limits

| Area | Default behavior |
| --- | --- |
| Reading | Current changes; direct dependencies only to answer a specific question |
| Review budget | One quick pass; stop when no clear candidate exists |
| Ablation budget | At most one experiment per task; expand only for a user-requested deeper review or correctness |
| Validation | Reuse existing results; add checks for unresolved regression risks; honor required repository checks |
| Output | Briefly mention meaningful simplifications and validation in the normal response; no separate review report |

Preserve authentication, authorization, data integrity, concurrency, idempotency, and resource cleanup by default. Match external-input validation to real boundary risks. Type annotations are not runtime validation. Combined removals must leave at least one effective layer for each required guarantee.

## Installation

Download this repository, name the directory containing `SKILL.md` as `coding-agent-lean-review`, and place it in your agent host's skill discovery directory.

For example, a host using `~/.agents/skills/` would discover:

```text
~/.agents/skills/coding-agent-lean-review/SKILL.md
```

Consult your host's documentation for its discovery directory and reload procedure. This repository contains only Markdown instructions and documentation; no runtime dependencies are required.

## Usage Examples

```text
Use $coding-agent-lean-review to quickly review the code changed in this task.
Preserve required behavior and error contracts. Stop if there is no clear
simplification candidate.
```

```text
After completing the fix, use $coding-agent-lean-review for one simplification
pass. Use local ablation on one complex defensive unit only if needed, and
include accepted changes in normal validation.
```

A host may select the skill based on its description. Installation alone does not guarantee execution after every code change. For explicit use, adapt the examples to your host's supported invocation syntax.

## Repository Structure

```text
coding-agent-lean-review/
├── SKILL.md       # Agent instructions (original English text)
├── README.md      # Chinese documentation
└── README.en.md   # English documentation
```

[SKILL.md](SKILL.md) is the source of truth for execution rules. This skill does not replace functional tests, security audits, or required repository validation.
