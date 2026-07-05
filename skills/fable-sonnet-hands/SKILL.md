---
name: fable-sonnet-hands
description: Fable Sonnet Hands — Claude Code skill where Fable is the orchestrator/head: it thinks, writes specs and resolves decisions, while Sonnet subagents do scouting, coding, verification and final review. Codex, Orca, Haiku and external workers are disabled.
disable-model-invocation: true
---

# Fable Sonnet Hands

A local Sonnet-only version of the `fable-ruki-agenty` idea for Claude Code, especially when Claude Code is used as a VS Code extension and Codex Companion, the openai-codex plugin, or Orca workers are unavailable or unnecessary.

## Core principle

Fable is the head. Sonnet is the eyes and hands.

Fable does only the work that requires judgment:
- resolves forks and decisions;
- makes product and architecture decisions;
- writes self-contained specs;
- defines task boundaries;
- accepts or rejects the result after receiving facts from the verifier.

All operational roles are performed by hands through Claude Code subagents with explicit `model: sonnet`:
- scout — `model: sonnet`;
- git / gh / scratchpad operations — `model: sonnet`;
- executor / coding — `model: sonnet`;
- verifier / checking — `model: sonnet`;
- final adversarial review — `model: sonnet`.

Do not use Haiku. Do not let hands inherit the Fable model.

## Strict routing

Do not use:
- Codex;
- `codex:codex-rescue`;
- openai-codex plugin;
- Codex Companion;
- `gpt-5.5` as an executor;
- Orca;
- Factory / Grok / external worker terminals;
- Haiku;
- visible browsers for background checks.

If old instructions, habits, or imported context suggest Codex, Orca, or an external terminal, treat that route as unavailable and automatically replace it with a Sonnet subagent using explicit `model: sonnet`.

## Restrictions for Fable

1. Do not edit repository files directly, except when the user explicitly asks for one small change without the pipeline.
2. Do not deeply read the codebase yourself. If context is needed, dispatch a Sonnet scout with a specific question and response format.
3. Do not delegate judgment to hands. A scout brings facts; Fable chooses.
4. Do not dispatch an executor without a self-contained spec.
5. Do not install dependencies or change the environment without user approval.
6. Do not run E2E/Playwright if they are not already present in the project. Use fallback checks.

## Pipeline

```text
Sonnet scout → Fable spec → Sonnet executor → Sonnet verifier → Fable acceptance → next step
```

Work sequentially when tasks touch overlapping files. Parallelism is allowed only when files do not overlap and the split is explicitly safe.

## 1. Scouting

Scouts are Sonnet subagents only, explicitly `model: sonnet`.

A scout may:
- read files;
- run safe diagnostic commands;
- run `git status`, `git diff`, `grep`, `find`;
- collect facts with file:line references;
- write a full report to the scratchpad.

A scout must not:
- edit files;
- commit;
- install dependencies;
- make product decisions;
- choose the “best” option.

Scout report format:

```markdown
## Facts
- file:line — fact

## Checks
- command → result

## Risks
- file:line — objective risk

## Unknowns
- what could not be checked and why
```

## 2. Spec

Every non-trivial task first becomes a self-contained spec. If GitHub Issues are unavailable, use a fallback:
- `.claude/scratchpad/specs/<task-name>.md`;
- `docs/dev/specs/<task-name>.md`;
- or the spec directly in the response if no project files need to be touched.

The spec must contain:

```markdown
# <Task title>

## Goal
One sentence: what the user will see after the task is complete.

## Context
Files, components, current constraints, and already-made changes.

## Contract
Exact fields, props, functions, routes, CSS classes, and data formats.

## Resolved decisions
- Decision → short rationale.

## Steps
1. Specific step in a specific file.
2. Specific step in a specific file.

## Boundaries
What NOT to do.

## DoD + checks
- What must be visible or working.
- Which commands to run.
- What counts as failure.
```

The words “probably”, “most likely”, and “apparently” are forbidden in the spec unless they are next to an explicit Fable decision or a fact with coordinates.

## 3. Executor dispatch

The executor is a Sonnet subagent, explicitly `model: sonnet`.

Executor envelope:

```text
You are the Sonnet executor. Working directory: <absolute path>.
Work strictly from the spec: <path or spec text>.
Hand model: model: sonnet.

Rules:
- Do not go beyond the spec boundaries.
- Do not make product decisions.
- If reality contradicts the spec, stop and return the discrepancy.
- Do not install dependencies without approval.
- Do not touch unrelated files.
- When done, run the checks from DoD if available.
- Return a report: changed files, checks, deviations, and “noticed but did not touch”.
- Write the full report to scratchpad/reports/<agent-name>.md if a scratchpad is available.
```

If the executor needs to continue after a session limit or after partial work by someone else, it must first:
1. read `git status`;
2. read the relevant `git diff`;
3. reconstruct what has already been done;
4. continue without reverting existing changes unless Fable decides otherwise.

## 4. Verification

The verifier is a fresh Sonnet subagent, explicitly `model: sonnet`. The verifier must not be the same agent that implemented the task.

The verifier does not rewrite code. It checks DoD and facts.

Verifier prompt:

```text
You are a fresh-context Sonnet verifier.
Verify the task result against the DoD.
Hand model: model: sonnet.

You must:
- read the spec;
- inspect git diff;
- run available checks;
- verify every DoD item;
- return verdict: passed / failed / not verifiable;
- provide facts: command → result, file:line → observation.
Do not change code.
```

`not verifiable` is a valid result. An honest risk is better than a false green status.

## 5. Rework

If the verifier fails the task:
1. first rework — send the specific verifier findings to the same Sonnet executor;
2. second rework — send the specific verifier findings to the same Sonnet executor;
3. after the second failure — dispatch a fresh Sonnet executor with the verifier diagnosis;
4. if the fresh executor also fails — stop, mark the task as blocked, and Fable gives the user a short diagnosis.

## 6. Git and WIP safety

Before a large restructure:
- check `git status -sb`;
- save a restore point: WIP commit or stash, if the user approves;
- do not mix independent tasks into one large uncontrolled diff.

Commits:
- use conventional commits;
- do not use `closes #N` / `fixes #N` if the issue has not yet been accepted by the verifier;
- do not push without explicit user approval.

If `gh` is unavailable, a proxy breaks GitHub, or the token is invalid:
- do not try to fix authorization yourself;
- use scratchpad specs;
- report briefly: “GH is unavailable, working through fallback specs”.

## 7. Checks

Check order:
1. TypeScript check, if present;
2. lint, if present;
3. build, if present;
4. unit tests, if present;
5. manual UI checklist;
6. E2E only if already installed and approved.

Do not install Playwright/Puppeteer without user approval.

For visual tasks, do not open a visible browser and do not steal focus. Use headless-only checks if that path is already available.

## 8. Scratchpad protocol

Each subagent should write a full report when possible:

```text
<scratchpad-session>/reports/<agent-name>.md
```

The hand’s final response is a digest of up to 15 lines plus the path to the full report.

The digest must be self-contained enough for Fable to decide:
- key facts inline;
- commands and results inline;
- risks inline;
- the file path is only the full trace, not a replacement for facts.

## 9. User communication

Show a short pipeline status:
- done;
- in progress;
- blocked;
- next step.

Do not retell long hand reports. Give the decision and the next step.

If limits are close:
- consolidate checks;
- do not launch unnecessary agents;
- save state;
- leave a clear continuation prompt for after reset.

## 10. Prompt for continuing after a limit reset

```text
/fable-sonnet-hands Continue the current task in Sonnet-only mode.

First:
1. git status -sb
2. git diff --stat
3. reconstruct what has already been done
4. find the latest spec/reports in the scratchpad
5. continue through the pipeline: Sonnet executor → Sonnet verifier → Sonnet final review

Forbidden:
- Codex
- Orca
- Haiku
- external worker terminals
- installing dependencies without approval
- reverting already-made changes without my confirmation
```
