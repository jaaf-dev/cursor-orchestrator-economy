# cursor-orchestrator-economy

Protocolo **planner/worker** para Cursor Agent: el modelo **caro del picker** (Grok, Opus, etc.) planifica e integra; la investigación web, la implementación y la verificación corren en **subagentes** con **Composer 2.5 estándar** (`composer-2.5`).

**Versión de la skill:** 1.4 (`SKILL.md`)

---

## Qué problema resuelve

En un chat único con modelo premium suele pasar esto:

- **WebSearch, lectura masiva y edits** comparten el mismo contexto → el padre se llena de ruido (HTML, greps, diffs) y **empeoran** las decisiones siguientes.
- Pagás **tarifa de orquestador** por trabajo que no necesita razonamiento frontier (buscar docs, mapear repo, escribir un markdown).
- Sin reglas claras, el agente **no delega** o **re-explora** después de un subagente (tokens duplicados).

**orchestrator-economy** impone **cuándo** delegar, **qué** mandar a cada worker y **qué formato** devolver — para que el padre haga pocas cosas caras y bien.

---

## Para qué ayuda

| Beneficio | Efecto |
|-----------|--------|
| **Contexto limpio en el padre** | Resúmenes estructurados del worker, no dumps de búsqueda |
| **Modelo acorde a la tarea** | Investigación/implementación en Composer; Grok/Opus en plan y tradeoffs |
| **Política de coste** | Workers en **Composer 2.5 standard**, no **Composer Fast** (en la grilla de Cursor, Fast suele ser **más caro** en input, no más barato) |
| **Repetibilidad** | Mismas gates en todos los repos (skill + rule always-on) |
| **Depuración** | Contratos `## Repo map`, spot-check único post-research, reglas anti re-edición post-implementer |

No reemplaza **Gentle AI** ni **Judgment Day** (`jd-*`).

---

## Qué incluye (funcionalidad)

| Componente | Función |
|------------|---------|
| **`SKILL.md`** | Gates (web → researcher, código → implementer, verify según tipo), etapas PLAN→RESEARCH→IMPLEMENT→VERIFY, notas de UI (Grok) |
| **`references/handoff-contracts.md`** | Plantillas de delegación y respuesta (Status, Findings, Repo map, etc.) |
| **`rules/orchestrator-economy.mdc`** | Rule **alwaysApply**: cargar y obedecer la skill |
| **`agents/*.md`** | Subagentes `researcher`, `implementer`, `verifier` con `model: composer-2.5` |

**Task tool:** usar siempre `"model": "composer-2.5"`. No usar `composer-2.5[fast=false]` (el tool lo rechaza). Evitar `composer-2.5-fast` salvo pedido explícito del usuario.

---

## Ahorro estimado (tokens y gasto)

Son **órdenes de magnitud**, no promesas fijas: dependen del picker, de cuánto web/repo/implementación tenga la tarea y de si antes todo lo hacía el padre inline.

### Referencia de precios (lista Cursor, indicativa)

| Modelo | Input (ref.) | Uso típico aquí |
|--------|----------------|-----------------|
| Grok 4.7 (padre) | ~$2 | Plan, integración, respuesta al usuario |
| Composer 2.5 **standard** | ~$0.5 | Workers (research, implement, verify) |
| Composer 2.5 **Fast** | ~$3 input | **No usado** por esta skill (más caro y peor calidad en la práctica) |

### Tokens

- Cursor documenta que en flujos multi-agente **la mayoría de los tokens** suelen consumirse en **workers**, no en el planner — pero los tokens del planner **cuestan más** por unidad ([agent swarms / model economics](https://cursor.com/blog/agent-swarm-model-economics)).
- **Aislar** búsquedas en el researcher evita que **decenas de miles de tokens** de HTML/snippets entren al contexto del padre y se **re-envíen** en cada turno siguiente.
- **Regla post-research (v1.3+):** como mucho **1 Read/Grep** de spot-check → evita un segundo “mini-mapeo” inline (ahorro variable, a menudo **10–30 %** del trabajo read-only duplicado en tareas medianas).

### Gasto en dinero (escenario ilustrativo)

Tarea realista: **2 búsquedas web + mapa de repo + doc en `docs/`** (~15–40 min de agente).

| Enfoque | Comportamiento | Gasto relativo (orientativo) |
|---------|----------------|------------------------------|
| **Todo en padre Grok** | Web + grep + write en un hilo | **Alto** — todo a ~$2 input + contexto hinchado |
| **Economy** | Grok planifica; ~2 searches + repo en **Composer**; Grok integra | **Medio-bajo** — bulk del input en ~$0.5; Grok solo en plan/resumen |
| **Padre Grok + Fast en workers** | Delegar mal con Fast | **Peor** — Fast ~$3 input vs $0.5 standard |

En tareas con **mucho** research/implementación, quienes pasan de “100 % padre premium” a “padre corto + workers Composer standard” suelen ver **reducciones importantes en coste del padre** (a veces **~40–70 %** del spend de esa sesión, según cuánto ruido dejaba de meterse en Grok/Opus). Medí en tu cuenta: trace del padre **WebSearch = 0** y tarjetas worker **composer-2.5**.

### Qué no ahorra

- Tareas **triviales** (un read, una pregunta corta): delegar añade overhead de Task → a veces conviene **inline** (la skill lo permite).
- **Doble rule** legacy + portal con el mismo texto → pagás contexto duplicado; elegí un canal.
- **Paralelar** muchos subagentes sin necesidad → más tokens totales, aunque sean baratos.

---

## Instalación

### 1. Clonar

```bash
git clone https://github.com/jaaf-dev/cursor-orchestrator-economy.git ~/project/cursor-orchestrator-economy
cd ~/project/cursor-orchestrator-economy
```

Layout del repo (raíz):

```text
README.md
SKILL.md
references/handoff-contracts.md
rules/orchestrator-economy.mdc
agents/researcher.md
agents/implementer.md
agents/verifier.md
```

### 2. Skill

```bash
mkdir -p ~/.cursor/skills
ln -sfn "$PWD" ~/.cursor/skills/orchestrator-economy
```

### 3. User Context → Skills (panel Customize)

```bash
STORE="$HOME/.local/state/cursor/agent-stores/cursor_agent_stores/<TU_USER_STORE_ID>/files/skills"
mkdir -p "$STORE"
ln -sfn "$PWD" "$STORE/orchestrator-economy"
```

`<TU_USER_STORE_ID>`: listá `~/.local/state/cursor/agent-stores/cursor_agent_stores/` o creá una skill en la UI y mirá la ruta.

### 4. Rule always-on

**A. Archivo (WSL / Remote-WSL)**

```bash
mkdir -p ~/.cursor/rules
ln -sfn "$PWD/rules/orchestrator-economy.mdc" ~/.cursor/rules/orchestrator-economy.mdc
```

**B. Portal Customize → Rules → + New**

Creá **Orchestrator economy**, **Always apply** ON, pegá el cuerpo de `rules/orchestrator-economy.mdc`. No dupliques textos distintos en legacy y portal.

### 5. Agents

```bash
mkdir -p ~/.cursor/agents
for a in researcher implementer verifier; do
  ln -sfn "$PWD/agents/$a.md" ~/.cursor/agents/$a.md
done
```

### 6. Gentle AI (opcional)

```bash
cd /path/to/tu/proyecto
gentle-ai skill-registry refresh --force
```

---

## Verificación rápida

1. Chat Agent nuevo → rule economy activa.
2. Tarea con web + repo → padre **sin** WebSearch/WebFetch (0 en trace).
3. Subagentes en **`composer-2.5`**.
4. Post-research: como mucho **un** spot-check; post-implementer: **sin** edits inline al doc.

---

## Convivencia

| Pieza | Notas |
|-------|--------|
| `subagent-composer-model` | Fija slug `composer-2.5` en Task; economy fija **cuándo** delegar |
| `gentle-ai.mdc` | Rule legacy; puede no listarse en Customize pero inyectarse en WSL |
| Judgment Day | Prioridad del skill **judgment-day** cuando el usuario juzga |

---

## Actualizar

```bash
cd ~/project/cursor-orchestrator-economy && git pull
```

Re-apuntá symlinks si cambiaste rutas. Bump `metadata.version` en `SKILL.md` cuando cambies comportamiento.

---

## Licencia

Apache-2.0 (ver frontmatter de `SKILL.md`).
