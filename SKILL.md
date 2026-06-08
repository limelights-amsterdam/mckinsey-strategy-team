---
name: mckinsey-strategy-team
description: >
  Orchestrates a live agent team that pressure-tests a strategic question with McKinsey-style
  frameworks. A team lead runs intake, classifies the problem, then runs the team in waves:
  diagnose + map the market in parallel, generate competing strategic options from opposing mandates
  and filter them to a shortlist, synthesize a recommendation, and finally turn a panel of
  adversarial verifiers loose on the load-bearing assumptions before it ships. Output: a board-ready
  decision memo + narrative that survives the meeting. Use to prepare a leadership or board session, structure a merger/M&A or
  major strategic choice, or stress-test / war-game an existing strategy. Triggers on: strategy
  team, agent team strategy, pressure-test strategy, war game, prepare a board decision, stress-test
  our plan, merger/M&A structuring, where-to-play decision. NOT for applying a single framework on
  its own, or a quick question that doesn't justify a multi-agent run.
---

# Strategy Team

A live, steerable **agent team** for strategic decisions. The value isn't running frameworks in
parallel — it's the **structured adversarial pressure**: options generated from opposing mandates so
you don't get a false binary, then a panel of verifiers each attacking the draft from a different
angle, while you steer individual teammates. That's the difference between a tidy analysis and one
that survives the room.

This skill is a concrete wiring of the **Six Workflow Patterns** onto strategy work. By default it
uses four, in order: **① classify-and-act → ② fan-out-and-synthesize → ④ generate-and-filter → ③
adversarial verification**. A **deep mode** (for high-stakes, hard-to-reverse calls) adds **⑤
tournament** (rank the surviving options pairwise) and **⑥ loop-until-done** (keep hunting risks
until the panel runs dry).

Built on the 21 McKinsey-style frameworks in `references/`
(source: github.com/aapersh/strategy-skills-for-claude).

---

## When to use / not

**Use it for:**
- Preparing a leadership or board session where a real decision gets made.
- Structuring a merger/M&A or major strategic choice (where to play, build/buy/partner).
- Stress-testing / war-gaming an existing strategy before commitment.

**Don't use it for:** applying one framework on its own, or a quick question that doesn't justify a
team of live agents over several waves — a single session is cheaper there.

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
- The **stakes → run mode**. Default is a **standard** run. Offer **deep mode** when the decision is
  hard to reverse or bet-the-company: it adds a tournament to rank surviving options (⑤) and a loop
  that keeps the panel hunting risks until two rounds come up empty (⑥), at a real token premium.
  Scale depth to the stakes, not by habit — confirm the mode with the user.

### Step 1 — Classify-and-act (routing)
Classify the engagement type and pick the right frameworks per role (see **Framework index**).
Produce a short **engagement plan**: problem statement + decision question (1-2 lines each), the
**wave plan** (who runs in wave A, what option-mandates get generated in wave B, who sits on the
verifier panel), each teammate's frameworks + deliverable, and whether this is a **standard or
deep-mode** run.

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

### Step 3 — Open the team (mechanics)
Use the agent-team primitives — don't fall back to plain subagents:
1. `TeamCreate({ team_name: "strat-<slug>", agent_type: "team-lead", description: "<topic>" })` —
   opens the team + its shared task list.
2. One `TaskCreate({ subject, description })` per teammate as you spawn it.
3. Spawn each teammate with the **Agent tool, passing `team_name` + `name`** (the teammate path, not
   fire-and-forget):
   `Agent({ team_name: "strat-<slug>", name: "<role>", subagent_type: "general-purpose", prompt: <template> })`.
4. Steer teammates with `SendMessage({ to: "<name>", message, summary })`.
5. Teammate messages arrive automatically — don't poll. Idle = normal, not an error.

The team runs in **waves** on purpose: each wave's output feeds the next, so later teammates reason
on real findings instead of guessing. Spawn a wave, wait for it, then spawn the next.

**Spawn template** (every teammate inherits nothing — give it the whole picture):
```
You are the <role> teammate on a strategy team. Question: <one line>.
1. First read the shared brief: /tmp/strat-team-<slug>/brief.md
2. Also read (only if your wave depends on earlier work): <upstream files, e.g. diagnose.md, market.md>
3. Read your framework(s): <REFS>/<domain>/<file>.md  (one or more; <REFS> is absolute)
4. Apply the framework method strictly to THIS question. No generic theory — concrete findings,
   with explicit assumptions where data is missing.
5. Write your output to /tmp/strat-team-<slug>/<role>.md (you own this file — no conflicts).
   Follow your framework's "Output Format".
6. Go idle afterward; the lead is notified automatically. Optionally SendMessage "team-lead" your
   3 key conclusions in plain text.
Output language: <follows the deliverable>.
```

### Step 4 — Wave A: diagnose + map the market (pattern ② fan-out)
Spawn these two **in parallel** — they're independent, and both feed everything downstream:
| Teammate (`name`) | Frameworks |
|---|---|
| `diagnose` | `01-diagnosis-and-framing/situation-assessment` + `growth-barriers` or `assumption-audit` |
| `market` | `02-.../market-mapping` + `competitive-intel` (+ `profit-pool-analysis` / `customer-segmentation` if relevant) |

Wait for both. Their files (`diagnose.md`, `market.md`) are the factual floor the options stand on —
which is exactly why options are **not** generated in this wave. Letting the option work run blind to
the market analysis was the old flow's weak point; the wave split fixes it.

### Step 5 — Wave B: generate competing options, then filter (pattern ④ generate-and-filter)
A single "strategy" teammate tends to emit its first three ideas and a false binary. Instead, generate
**divergent** options from opposing mandates, then filter to a shortlist — the single biggest lever on
decision quality. (`03-.../strategic-options` warns against exactly the narrow option set this prevents.)

1. Spawn **2 option-generators in parallel**, each reading `brief.md` + `diagnose.md` + `market.md`
   + `<REFS>/03-strategic-choice-and-economics/strategic-options.md` (and `business-case-builder`
   for rough economics). Bias each one hard — divergence is the point:
   | Generator (`name`) | Mandate |
   |---|---|
   | `option-bull` | Maximize upside/growth. The boldest defensible move; assume resources can be found. |
   | `option-lean` | Maximize resilience/efficiency. Cheapest, lowest-risk, fastest-to-reverse path. |

   Each writes 2-3 distinct options (each with its **"what must be true"**) to its own file
   (`options-bull.md`, `options-lean.md`).
2. **The lead is the filter** (no extra teammate): read both files, **dedupe** overlapping options,
   drop dominated ones, and **score the survivors** against the decision criteria (attractiveness,
   feasibility, risk, economics, strategic fit). Keep the **2-3 strongest**, and write the shortlist
   + scoring to `/tmp/strat-team-<slug>/options-shortlist.md`.

> **Deep mode** adds a third generator `option-contrarian` (mandate: *do the opposite of the obvious
> play — what would we do if the consensus move were forbidden?*) for wider divergence.

### Step 6 — Synthesize the draft recommendation (lead)
With the shortlist in hand, pick the recommended option and build a draft with the Pyramid Principle /
SCQA (logic from `06-.../narrative-builder`). Write to `/tmp/strat-team-<slug>/draft-recommendation.md`:
governing thought → 3 supporting arguments → backing per argument → the recommended option + **the
load-bearing assumptions it rests on**, named explicitly. The panel attacks those next, so don't bury
them.

### Step 7 — Pressure-test: a panel of verifiers (pattern ③ adversarial verification)
The draft is the "worker"; spawn a **panel** that attacks it from independent angles — one lens each,
so failure modes a single red-teamer would miss get caught. Spawn **only now**, after
`draft-recommendation.md` exists (otherwise they idle or attack incomplete work). Default panel of 3
(drop to 2 for lower stakes, 4 in deep mode); each reads the draft + brief and writes to its own file:
| Verifier (`name`) | Lens / framework |
|---|---|
| `verify-assumptions` | `01-.../assumption-audit` — which load-bearing beliefs are weakest, and where does the logic break if one is wrong? |
| `verify-wargame` | `05-.../war-gaming` — war-game competitor moves, market shifts, customer reactions, regulation. |
| `verify-execution` | `04-.../operating-model-design` + `05-.../risk-and-mitigation` — can the org actually execute this, and do the economics survive a bad case? |

**Tally rule — this is the point of a panel, not a lone critic:** a claim or assumption that **≥2 of
the panel** independently flag as fatal does *not* survive — it must be fixed, hedged, or the
recommendation changes. The lead collects the surviving vulnerabilities + mitigations into
`vulnerabilities.md`.

> **Deep mode — two extras:**
> - **⑥ loop-until-done:** re-spawn the war-game lens for another round and keep going until **two
>   consecutive rounds surface no new fatal risk**. Stops the panel quitting at the first three
>   obvious risks.
> - **⑤ tournament:** if **3+ options survive** and the choice is genuinely close, run pairwise
>   judging — spawn judges to compare survivors two at a time on the decision criteria, advancing
>   winners until one stands. Beats averaging a committee's opinion.

### Step 8 — Finalize the leadership deliverable
Fold the surviving critique into the recommendation. Deliver per `06-.../decision-memo` +
`06-.../narrative-builder`:
- Decision question (the SCQA question).
- Recommendation (Pyramid: governing thought + 3 arguments).
- Options + trade-offs (table) and why the recommended one wins.
- Business-case summary (key numbers + sensitivities).
- Top risks + mitigations (straight from the pressure-test; flag the ones ≥2 of the panel hit).
- First 90 days / next decisions + owners.
- Open questions + explicit assumptions. War-game vulnerabilities as an appendix.
- 60-second spoken story + 3 hostile-Q&A answers (so it survives the room).

**Durable output:** working files can stay in `/tmp`, but `/tmp` is volatile — write the final memo
to a durable path (ask the user where, or default to `./strategy-sessions/<slug>/decision-memo.md`)
and show the core in chat.

### Step 9 — Cleanup
- Shut each teammate down: `SendMessage({ to: "<name>", message: { type: "shutdown_request", reason: "done" } })`.
- Clean up the team once all teammates are gone (only the **lead** runs cleanup).

---

## Guardrails
- **File ownership:** each teammate owns exactly one output file → no overwrite conflicts.
- **One team, run in waves — no nested teams** (hard limit). Spawn a wave, wait, spawn the next; the
  lead never builds ahead of a wave it's still waiting on.
- **Concurrency:** keep each wave to ~2-3 live teammates — that's the sweet spot. A standard run
  totals ~7 agents over its waves (2 diagnose/market + 2 generators + 3 verifiers); deep mode adds a
  few more. Say so up front.
- **Token-aware:** waves cost real tokens. Default to standard mode and only go deep when the stakes
  justify it — scale depth to the decision, not by habit.
- **The structure is the value, not the head-count:** the **generate-then-filter** step (so options
  aren't one teammate's first idea) and the **≥2 panel tally** (so no lone verifier vetoes, and no
  weak assumption survives) are what make the output robust rather than merely confident. Don't skip
  them to save a teammate.

---

## Framework index (21 frameworks → path → when to use)

**01 · Diagnosis & framing** — `references/01-diagnosis-and-framing/`
| Framework | When |
|---|---|
| `situation-assessment` | Factual baseline before any direction. Almost always wave 1. |
| `growth-barriers` | Growth is stuck; leadership debates symptoms. |
| `assumption-audit` | Strategy leans on possibly weak beliefs. Also the `verify-assumptions` panel lens. |

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
| `war-gaming` | Stress-test strategy before launch. The `verify-wargame` panel lens (loops in deep mode). |
| `risk-and-mitigation` | Strategic risk gets an owner + response plan. |
| `kpi-architect` | Metrics are noisy, lagging, or performative. |
| `value-realization` | Benefits must be tracked after launch. |

**06 · Alignment & executive communication** — `references/06-alignment-and-executive-communication/`
| Framework | When |
|---|---|
| `stakeholder-alignment` | Pre-wire the recommendation before the meeting. |
| `narrative-builder` | The story must land in 60 seconds (Pyramid/SCQA). The synthesis step. |
| `decision-memo` | A clear recommendation in writing. The deliverable. |
