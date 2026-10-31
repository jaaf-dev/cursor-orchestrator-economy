---
name: orchestrator-economy
description: "Trigger: always on. Web search compare docs repo map delegate researcher implementer verifier. Composer 2.5 standard workers."
license: Apache-2.0
metadata:
  author: alexis
  version: "1.4"
---

## Activation Contract

Always active via user rule `orchestrator-economy.mdc` (`alwaysApply: true`). Read this skill before acting in Agent mode.

The parent decides **per turn** using Decision Gates below. When a **Delegate** row matches, launch the worker — do not do that work inline. When **Parent inline** matches, one bounded step in the main chat is OK.

## Hard Rules

- Parent: plan, decompose, integrate, report. Never multi-step WebSearch/WebFetch or large read-only exploration inline when **Delegate → researcher** applies.
- Every worker **Task** MUST set `"model": "composer-2.5"`. Valid Task slugs are only `composer-2.5` and `composer-2.5-fast` — **never** `composer-2.5[fast=false]` (the tool rejects it before the subagent starts).
- Never use `composer-2.5-fast` unless the user explicitly requested Fast in that message.
- Composer 2.5 standard is the economical worker tier; Composer 2.5 (Fast) costs more and is disallowed by default.
- Prefer custom `researcher` over built-in Explore when delegation is required and billed model must be Composer standard.
- Do not edit inline files already delegated to `implementer` for the same work unit.
- **After implementer `complete`:** do not edit those deliverable paths inline (titles, copy, index rows). Request a **follow-up Task → implementer** for doc tweaks, or list edits in **REPORT** for the user to apply.
- Workers must not sub-delegate.
- **After researcher `complete`:** do not re-map the repo (no multi-file Grep/Read/Glob rounds). At most **one** `Read` **or** **one** `Grep` on a path the researcher listed under **## Repo map**. If evidence is missing, Task → researcher again — do not explore inline.

## Decision Gates

| Situation | Route | Task `model` |
| --- | --- | --- |
| Multi-step web, external docs, compare sources online | **Delegate → researcher** | `composer-2.5` |
| Large read-only repo map (many files/grep rounds) | **Delegate → researcher** | same |
| Single known path read, one grep, one symbol (no prior researcher on this topic) | **Parent inline** | — |
| User question answerable from conversation only | **Parent inline** | — |
| Code edits, tests, authorized implementation | **Delegate → implementer** | same |
| Review after delegated changes under `src/`, runtime config, or tests | **Delegate → verifier** | same |
| Delegated changes **only** under `docs/**/*.md` (no `src/`) | **Verifier optional** — parent readback: diff summary + index coherence in ≤5 lines | — |
| User asked to skip verification or quick probe | Skip verifier; say so in PLAN | — |
| Judgment Day (`jd-*`) when user invoked juzgar | Follow **judgment-day** skill | per that skill |

When a delegate row matches, this skill owns **handoff shape**; `subagent-composer-model` owns worker model slug (standard Composer, no fast).

## UI interpretation (audit Grok vs Composer)

- Lines like **Searching web** or **Fetch** appearing **below** a **Completed** Task card may be **subagent tool echo** in the parent scroll, not WebSearch/WebFetch on the parent.
- Judge compliance by **parent tool trace** (parent WebSearch/WebFetch count) and the diagnostic prompt, not scroll layout alone.
- Stuck **Searching…** spinners the parent cannot see in trace may be UI lifecycle bugs (subagent finished, parent call id not closed).

## Execution Steps

1. **PLAN** — One short block before the first Task: which gates apply; **verifier at end** (yes / no / docs-only readback); **post-research** = at most one spot-check path from **## Repo map** (or N/A).
2. **RESEARCH** — If researcher gate matched, Task → `researcher`, foreground, wait. Else skip.
3. **INTEGRATE (research)** — Use researcher **Findings** + **Repo map**; optional single spot check only (Hard Rules).
4. **IMPLEMENT** — If implementer gate matched, Task with envelope; parallel only if edit surfaces disjoint.
5. **INTEGRATE** — Merge worker outputs; no duplicate inline work. Do not StrReplace/Write paths the implementer just delivered — follow-up implementer Task if copy must change.
6. **VERIFY** — Per Decision Gates (verifier Task or docs-only readback).
7. **REPORT** — Concise summary; no raw worker dumps.

Delegation envelope: `references/handoff-contracts.md`. Inject `## Skills to load before work` per `skill-resolver.md` for implementer.

## Output Contract

Parent final message: summary, failed/skipped checks, next step. Parse worker `## Status`, `## Needs your decision`, **## Repo map**.

## References

- `references/handoff-contracts.md` — delegation envelope and response templates
