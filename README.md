# mckinsey-strategy-team

A Claude Code **skill** that orchestrates a live *agent team* for strategic decisions. A team lead
runs intake, classifies the question, spawns 3-4 teammates that work McKinsey-style frameworks in
parallel, synthesizes a recommendation, then a **red-team** teammate attacks its own logic
(pressure-test) before it ships. Output: a board-ready **decision memo + narrative** that survives a
leadership or board meeting.

Use it to: prepare a leadership/board session, structure a merger/M&A question, or stress-test /
war-game an existing strategy.

## What's inside

```
mckinsey-strategy-team/
├── SKILL.md        # the team-lead recipe (the orchestration)
├── README.md       # this file
├── WHY.txt         # the design rationale (what, how, value)
└── references/     # 21 McKinsey-style frameworks, 6 domains (method docs)
```

The 21 frameworks come from github.com/aapersh/strategy-skills-for-claude. They are not installed as
separate skills — the teammates **read them as files**. That sidesteps the limitation that a
teammate does not inherit a subagent's skill frontmatter.

## Prerequisites

- **Claude Code ≥ 2.1.32** (`claude --version`)
- **Agent teams enabled** — in `~/.claude/settings.json`:
  ```json
  { "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
  ```
- Works in any terminal — the default `"teammateMode": "in-process"` is all you need.

## Install

**Option A — user skill (available everywhere):**
```sh
# put this folder wherever you like, then symlink it:
ln -s "$(pwd)/mckinsey-strategy-team" ~/.claude/skills/mckinsey-strategy-team
```

**Option B — per project:**
```sh
cp -R mckinsey-strategy-team .claude/skills/mckinsey-strategy-team
```

> At runtime the lead resolves its `references/` path, trying the user-level install
> (`~/.claude/skills/mckinsey-strategy-team/references`) first and falling back to a per-project
> install (`$PWD/.claude/skills/mckinsey-strategy-team/references`) — so **either install option
> above works**. The only requirement is that the install folder keeps the name
> `mckinsey-strategy-team`.

## Use

Start a new Claude Code session and say, for example:

> *"Use the strategy team to prepare a leadership decision on whether we do [X] or [Y]. Here's the
> context: …"*

The lead runs a short intake and shows you an engagement plan (which teammates, which frameworks)
before any teammates are spawned. Then the pipeline runs:
**classify → fan-out → synthesize → adversarial pressure-test → decision memo**.

A run = 3-4 live sessions (teammates) — noticeably more tokens than an ordinary chat. Use it for
real decisions, not quick questions.

## Not for this

- Applying a single framework on its own → just load that framework directly.
- A quick question that doesn't justify a multi-agent run → use a single session.
