# orchestrator-economy

Planner/worker protocol for Cursor Agent: the **parent** (expensive picker model) plans and integrates; **research**, **implementation**, and **verification** run in subagents on **Composer 2.5 standard** (`composer-2.5`), not Composer Fast.

**Skill version:** 1.4 (see `SKILL.md` frontmatter)

## What you get

| Piece | Role |
|-------|------|
| `SKILL.md` | When to delegate, gates, stages, UI notes |
| `references/handoff-contracts.md` | Task envelopes and worker response templates |
| `rules/orchestrator-economy.mdc` (optional in repo) | Always-on rule pointing at the skill |
| `agents/*.md` (optional) | `researcher`, `implementer`, `verifier` with pinned `model: composer-2.5` |

Works alongside **Gentle AI** (`gentle-ai.mdc` legacy rule). Does not replace Judgment Day (`jd-*`).

## Prerequisites

- Cursor Agent with **Task** subagents (`researcher`, `implementer`, `verifier` if using custom agents)
- Valid **Task** model slugs: `composer-2.5` or `composer-2.5-fast` only — **do not** pass `composer-2.5[fast=false]` (Task rejects it)

## Install (Linux / WSL)

### 1. Clone this repo

```bash
git clone <your-repo-url> ~/cursor-orchestrator-economy
cd ~/cursor-orchestrator-economy
```

Expected layout (use this repo as **repository root**):

```text
README.md
SKILL.md
references/handoff-contracts.md
rules/orchestrator-economy.mdc
agents/researcher.md
agents/implementer.md
agents/verifier.md
```

### 2. Skill (required)

```bash
mkdir -p ~/.cursor/skills
ln -sfn "$PWD" ~/.cursor/skills/orchestrator-economy
```

(If you nest the package under `skills/orchestrator-economy/`, point the symlink at that subdirectory.)

### 3. Cursor **User Context → Skills** (UI catalog)

Symlink so the skill appears in **Customize → Skills**:

```bash
STORE="$HOME/.local/state/cursor/agent-stores/cursor_agent_stores/<YOUR_USER_STORE_ID>/files/skills"
mkdir -p "$STORE"
ln -sfn "$PWD" "$STORE/orchestrator-economy"
```

Find `<YOUR_USER_STORE_ID>`: create any skill once in the UI and inspect that path, or list `~/.local/state/cursor/agent-stores/cursor_agent_stores/`.

### 4. Rule — always on (pick one)

**A. Legacy file (WSL, works with Remote-WSL Agent)**

```bash
mkdir -p ~/.cursor/rules
ln -sfn "$PWD/rules/orchestrator-economy.mdc" ~/.cursor/rules/orchestrator-economy.mdc
```

Ensure frontmatter has `alwaysApply: true`.

**B. Customize → Rules → + New (portal)**

Create **Orchestrator economy**, turn **Always apply** on, paste the body from `rules/orchestrator-economy.mdc` (or the short rule text in this repo). The portal may not list legacy `~/.cursor/rules/` files; duplicating here is for visibility only — avoid two conflicting copies with different text.

### 5. Custom agents (recommended)

```bash
mkdir -p ~/.cursor/agents
for a in researcher implementer verifier; do
  ln -sfn "$PWD/agents/$a.md" ~/.cursor/agents/$a.md
done
```

### 6. Gentle AI registry (optional, per project)

If you use Gentle AI:

```bash
cd /path/to/your/project
gentle-ai skill-registry refresh --force
```

Confirm `orchestrator-economy` appears in `.atl/skill-registry.md`.

## Verify

1. **New Agent chat** — check **Rules** include orchestrator economy (portal or legacy).
2. Run a realistic task (web compare + repo read + optional doc). Parent should **not** use WebSearch/WebFetch (0 in parent trace).
3. Confirm subagent cards use **`composer-2.5`**.
4. After the task, use your **diagnostic prompt** — gates, spot-check only after researcher, no inline edits after implementer **complete**.

## Policy summary

- Multi-step web / large repo map → **researcher**
- Code / docs implementation → **implementer**
- `src/` or runtime changes → **verifier**; **docs-only** → verifier optional, short parent readback
- After **researcher complete**: at most one Read or Grep on a path from **## Repo map**
- After **implementer complete**: do not edit deliverables inline; follow-up Task to implementer
- UI: “Searching web…” under a completed Task card may be **subagent echo**, not parent tools

## Coexistence

| Component | Notes |
|-----------|--------|
| `subagent-composer-model` skill | Still sets Task model to `composer-2.5`; economy sets **when** and **how** to delegate |
| `gentle-ai.mdc` | Stays in `~/.cursor/rules/`; not shown in Customize portal but still injected on WSL |
| Judgment Day | When user invokes juzgar / `jd-*`, follow **judgment-day** skill |

## Upgrade

Replace symlinks after `git pull`. Bump version in `SKILL.md` `metadata.version` when you change behavior.

## License

Apache-2.0 (skill frontmatter). Adapt agents/rules as needed.
