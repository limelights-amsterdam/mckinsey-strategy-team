---
name: mckinsey-strategy-team
description: >
  Orchestrates a live agent team that pressure-tests a strategic question with McKinsey-style
  frameworks. A team lead runs intake, classifies the problem, spawns 3-4 teammates that work the
  right frameworks in parallel, synthesizes a recommendation, then a red-team teammate attacks the
  load-bearing assumptions before it ships. Output: a board-ready decision memo + narrative that
  survives the meeting. Use to prepare a leadership or board session, structure a merger/M&A or
  major strategic choice, or stress-test / war-game an existing strategy. Triggers on: strategy
  team, agent team strategy, pressure-test strategy, war game, prepare a board decision, stress-test
  our plan, merger/M&A structuring, where-to-play decision. NOT for applying a single framework on
  its own, or a quick question that doesn't justify a multi-agent run.
---

# Strategy Team

A live, steerable **agent team** for strategic decisions. The value isn't running frameworks in
parallel — it's the **adversarial debate**: teammates attacking each other's logic while you steer
individual teammates. That's the difference between a tidy analysis and one that survives the room.

Three patterns, in order: **classify-and-act → fan-out-and-synthesize → adversarial verification**.

Built on the 21 McKinsey-style frameworks in `references/`
(source: github.com/aapersh/strategy-skills-for-claude).

---

## When to use / not

**Use it for:**
- Preparing a leadership or board session where a real decision gets made.
- Structuring a merger/M&A or major strategic choice (where to play, build/buy/partner).
- Stress-testing / war-gaming an existing strategy before commitment.

**Don't use it for:** applying one framework on its own, or a quick question that doesn't justify
3-4 live agents — a single session is cheaper there.

---

## Prerequisites
- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` (env or settings.json)
- Claude Code ≥ 2.1.32
- `teammateMode` set to `in-process` (the default) — works in any terminal.

If agent teams are off, say so and offer to set the env var rather than silently degrading to plain
subagents.

---

## The pipeline

### Step 0 — Intake (lead + user, short)
Better context in = better output out. Pull actively:
- The problem in one line, and the real **decision question** (what must be decided, by whom?).
- The audience (leadership team? board? a single decision-maker?) and the output language.
- Constraints (time, money, politics), what's already known/researched, available data/sources.
- The problem type (growth stalled? merger? pricing? new market? portfolio allocation?).

### Step 1 — Classify-and-act (routing)
Classify the engagement type and pick the right frameworks per role (see **Framework index**).
Produce a short **engagement plan**: problem statement + decision question (1-2 lines each),
3-4 teammates with their role + which framework files they read, and each one's deliverable.

→ **Checkpoint:** show this plan to the user ("I'll open a team and spawn these teammates with these
frameworks — OK?") so they can steer before tokens burn.

### Step 2 — Workspace + resolve skill root
1. Slug the topic and make a workspace: `/tmp/strat-team-<slug>/`.
2. Write `brief.md` with ALL intake context + the engagement plan. This is the shared source every
   teammate reads (teammates do NOT inherit the lead's conversation).
3. **Resolve the references directory absolutely** (critical for portability). The skill may be
   installed user-level *or* per-project, so try both and keep the first that resolves:
   ```
   realpath ~/.claude/skills/mckinsey-strategy-team/references 2>/dev/null \
     || realpath "$PWD/.claude/skills/mckinsey-strategy-team/references"
   ```
   Call the result `<REFS>` — it already ends in `/references`. Hand teammates paths of the form
   `<REFS>/<domain>/<file>.md` (never `<REFS>/references/...` — that double-counts the folder).

> ⚠️ **Never hardcode a user-specific path** in spawn prompts. Teammates are fresh,
> cwd-independent sessions; a hardcoded path breaks the moment the folder lives somewhere else.
> Always resolve `<REFS>` at runtime as above and pass the absolute result.

### Step 3 — Fan-out: open team + spawn teammates (wave 1)
Use the agent-team primitives — don't fall back to plain subagents:
1. `TeamCreate({ team_name: "strat-<slug>", agent_type: "team-lead", description: "<topic>" })` —
   opens the team + its shared task list.
2. One task per teammate: `TaskCreate({ subject, description })`.
3. Spawn each teammate with the **Agent tool, passing `team_name` + `name`** (this is the teammate
   path, not fire-and-forget):
   `Agent({ team_name: "strat-<slug>", name: "<role>", subagent_type: "general-purpose", prompt: <template> })`.
4. Steer teammates with `SendMessage({ to: "<name>", message, summary })`.
5. Teammate messages arrive automatically — don't poll. Idle = normal, not an error.

**Default 3 roles in wave 1** (scale to the problem — sometimes 2, sometimes 4):
| Teammate (`name`) | Frameworks (read from `references/`) |
|---|---|
| `diagnose` | `01-diagnosis-and-framing/situation-assessment` + `growth-barriers` or `assumption-audit` |
| `market` | `02-.../market-mapping` + `competitive-intel` (+ `profit-pool-analysis` / `customer-segmentation` if relevant) |
| `strategy` | `03-.../strategic-options` + `business-case-builder` (+ `pricing-strategy` / `portfolio-review` if relevant) |

**Spawn template** (give each teammate enough context — they inherit nothing):
```
You are the <role> teammate on a strategy team. Question: <one line>.
1. First read the shared brief: /tmp/strat-team-<slug>/brief.md
2. Read your framework(s): <REFS>/<domain>/<file>.md  (one or more; <REFS> is absolute)
3. Apply the framework method strictly to THIS question. No generic theory — concrete findings,
   with explicit assumptions where data is missing.
4. Write your output to /tmp/strat-team-<slug>/<role>.md (you own this file — no conflicts).
   Follow your framework's "Output Format".
5. You'll go idle afterward; the lead is notified automatically. Optionally SendMessage the
   "team-lead" with your 3 key conclusions in plain text.
Output language: <follows the deliverable>.
```

### Step 4 — Synthesize
Wait for the wave-1 teammates (don't build ahead of them). Read all role files and build a draft
recommendation using the Pyramid Principle / SCQA (logic from `06-.../narrative-builder`). Write to
`/tmp/strat-team-<slug>/draft-recommendation.md`: governing thought → 3 supporting arguments →
backing per argument → the recommendation + the key assumptions it rests on.

### Step 5 — Adversarial pressure-test (wave 2)
Spawn the **`red-team` teammate only now** — after `draft-recommendation.md` exists (otherwise it
idles or attacks incomplete work). Brief:
```
You are the red team. Read /tmp/strat-team-<slug>/draft-recommendation.md and the brief.
Read your frameworks: <REFS>/05-risk-performance-and-value-governance/war-gaming.md and
<REFS>/01-diagnosis-and-framing/assumption-audit.md.
Your job is NOT to confirm — it's to REFUTE the recommendation. Attack the load-bearing
assumptions: what must be true for this to hold, and where does it break? War-game competitor
moves, market shifts, customer reactions, execution failure, regulation. Write the surviving
vulnerabilities + mitigations to /tmp/strat-team-<slug>/vulnerabilities.md. Message me when done.
```
For high-stakes questions: spawn a second teammate and have them challenge each other via
`SendMessage` (scientific-debate style) — the assumption that survives is robust.

### Step 6 — Finalize the leadership deliverable
Fold the surviving critique into the recommendation. Deliver per `06-.../decision-memo` +
`06-.../narrative-builder`:
- Decision question (the SCQA question).
- Recommendation (Pyramid: governing thought + 3 arguments).
- Options + trade-offs (table) and why the recommended one wins.
- Business-case summary (key numbers + sensitivities).
- Top risks + mitigations (straight from the pressure-test).
- First 90 days / next decisions + owners.
- Open questions + explicit assumptions. War-game vulnerabilities as an appendix.
- 60-second spoken story + 3 hostile-Q&A answers (so it survives the room).

**Durable output:** working files can stay in `/tmp`, but `/tmp` is volatile — write the final memo
to a durable path (ask the user where, or default to `./strategy-sessions/<slug>/decision-memo.md`)
and show the core in chat.

### Step 7 — Cleanup
- Shut each teammate down: `SendMessage({ to: "<name>", message: { type: "shutdown_request", reason: "done" } })`.
- Clean up the team once all teammates are gone (only the **lead** runs cleanup).

---

## Guardrails
- **File ownership:** each teammate owns exactly one output file → no overwrite conflicts.
- **One team at a time, no nested teams** (hard limit).
- **Wait for your teammates** before synthesizing as the lead.
- **Token-aware:** 3-4 teammates is the sweet spot; more is linearly costlier, not linearly better.
- A run = 3-4 live sessions in panes. Say so up front.

---

## Framework index (21 frameworks → path → when to use)

**01 · Diagnosis & framing** — `references/01-diagnosis-and-framing/`
| Framework | When |
|---|---|
| `situation-assessment` | Factual baseline before any direction. Almost always wave 1. |
| `growth-barriers` | Growth is stuck; leadership debates symptoms. |
| `assumption-audit` | Strategy leans on possibly weak beliefs. Also red-team input. |

**02 · Market & competition** — `references/02-market-and-competitive-intelligence/`
| Framework | When |
|---|---|
| `market-mapping` | Size/segment the market, find white space. |
| `competitive-intel` | Predict what rivals will do. |
| `customer-segmentation` | Sharper customer groups for the decision. |
| `profit-pool-analysis` | Where is value created and captured? |

**03 · Strategic choice & economics** — `references/03-strategic-choice-and-economics/`
| Framework | When |
|---|---|
| `strategic-options` | Alternatives + criteria before commitment. Core for merger/choice. |
| `pricing-strategy` | Pricing power, discounting, monetization unclear. |
| `business-case-builder` | Decision needs economics, sensitivities, risks. |
| `portfolio-review` | Allocate resources across bets. |

**04 · Operating model & execution** — `references/04-operating-model-and-execution/`
| Framework | When |
|---|---|
| `operating-model-design` | Translate strategy into how work runs. |
| `initiative-prioritizer` | Too many initiatives competing for attention. |
| `transformation-roadmap` | Strategy becomes phased execution with owners. |

**05 · Risk, performance & value governance** — `references/05-risk-performance-and-value-governance/`
| Framework | When |
|---|---|
| `war-gaming` | Stress-test strategy before launch. Core of the red team. |
| `risk-and-mitigation` | Strategic risk gets an owner + response plan. |
| `kpi-architect` | Metrics are noisy, lagging, or performative. |
| `value-realization` | Benefits must be tracked after launch. |

**06 · Alignment & executive communication** — `references/06-alignment-and-executive-communication/`
| Framework | When |
|---|---|
| `stakeholder-alignment` | Pre-wire the recommendation before the meeting. |
| `narrative-builder` | The story must land in 60 seconds (Pyramid/SCQA). The synthesis step. |
| `decision-memo` | A clear recommendation in writing. The deliverable. |
