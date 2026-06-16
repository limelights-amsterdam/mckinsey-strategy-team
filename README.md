# mckinsey-strategy-team

A Claude Code **skill** that orchestrates a live *agent team* for strategic decisions. A team lead
runs intake, classifies the question, then runs the team in waves on McKinsey-style frameworks:
diagnose + map the market in parallel, generate **competing options from opposing mandates** and
filter them to a shortlist, synthesize a recommendation, then turn a **panel of verifiers** loose on
its load-bearing assumptions (a weakness ≥2 of them agree on doesn't survive) before it ships.
Output: a board-ready **decision memo + narrative** that survives a leadership or board meeting.

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

## Credits

The framework reference docs in `references/` are based on **Oria AI's "21 Strategy Skills for
Claude"** — [oria.one/resources/21-strategy-skills-for-claude](https://www.oria.one/resources/21-strategy-skills-for-claude).
This project adapts that material into a live agent-team orchestration; all the underlying
McKinsey-style methodology and framework write-ups originate from their work. Huge thanks to the
Oria team for making it available — bedankt! 🙏

## Prerequisites

- **Claude Code ≥ 2.1.32** (`claude --version`)
- **Agent teams enabled.** Quickest way — run this in your terminal before you start:
  ```sh
  export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
  ```
  That only applies to the Claude Code session you launch from that shell. To enable it
  everywhere (including the desktop/web app), set it in `~/.claude/settings.json` instead:
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

The lead runs a short intake and shows you an engagement plan (the wave plan, which frameworks,
standard vs deep mode) before any teammates are spawned. Then the pipeline runs:
**classify → diagnose + market → generate-and-filter options → synthesize → verifier-panel
pressure-test → decision memo**. Deep mode adds a tournament to rank surviving options and a
loop that keeps hunting risks until the panel runs dry.

A standard run is ~7 live sessions across its waves (more in deep mode) — noticeably more tokens
than an ordinary chat. Use it for real decisions, not quick questions.

## Not for this

- Applying a single framework on its own → just load that framework directly.
- A quick question that doesn't justify a multi-agent run → use a single session.

## License

MIT — see [`LICENSE`](LICENSE). This covers the orchestration and docs in this repo. The framework
reference material is adapted from Oria AI (see [Credits](#credits)); please keep that attribution
when reusing it.
