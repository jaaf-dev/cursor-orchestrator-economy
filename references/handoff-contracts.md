# Handoff contracts (orchestrator-economy)

Copy sections into parent Task prompts and require workers to reply using the matching response template.

## Delegation envelope (parent → worker)

```markdown
## Objective
One sentence: what this worker must achieve.

## Scope
In scope:
Out of scope (do not expand):

## Allowed edit surfaces
- Exact paths or globs (implementer only)
- "read-only" for researcher and verifier

## Context from parent
Facts, URLs, errors, decisions already made — do not re-discover if listed here.

## Definition of done
Verifiable checklist.

## Skills to load before work
- /absolute/path/to/SKILL.md

## Response format
Use the worker response template from orchestrator-economy handoff-contracts.

## Constraints
- Task model: `composer-2.5` only (never `composer-2.5-fast` or `composer-2.5[fast=false]`)
- No composer-2.5-fast or (Fast) variants unless the user explicitly requested in this chat turn
- No sub-delegation
- Parent must not edit deliverable paths after implementer completes; copy/title fixes → follow-up implementer Task
- Blockers: single ## Needs your decision item
```

## Researcher response template

```markdown
## Status
complete | partial | blocked

## Findings
- Verifiable fact (evidence inline)

## Repo map
- path:line or path — what it shows (concrete files for parent spot-check only)

## Sources
- URL or path:line — what it supports

## Uncertainties
- What could not be confirmed

## Recommended next step
One concrete action for implementer or parent

## Needs your decision
(Optional; at most one question)
```

## Implementer response template

```markdown
## Status
complete | partial | blocked

## Spec coverage
Which DoD items are done

## Changes
- path — one line per meaningful change

## Verification
`command`: observed result (exit code, pass/fail)

## Risk
Risk: none | item N (reason)

## Handoff for verifier
Bullets of what to check

## Needs your decision
(Optional; at most one question)
```

## Verifier response template

```markdown
## Verdict
pass | fail | inconclusive

## Criteria
| Criterion | Result | Evidence |

## Regressions noted
(Omit if none)

## Blockers
For implementer or parent

## Needs your decision
(Optional; at most one question)
```
