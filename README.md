# virtual-team

A Claude Code plugin (distributed via the `heard-plugins` marketplace) that installs four named subagents — `nate`, `priya`, `simone`, `dev` — covering full-stack, frontend, security, and quality. Each has a distinct voice, scope, and review protocol, and is told to stay in its lane and defer outside it.

## Roster

| Agent | Role | Model | Tools | Use for |
|---|---|---|---|---|
| `nate` | Senior full-stack | sonnet | Read, Grep, Glob, Bash, Edit, Write | Architecture, state management, type design, performance, refactoring, build tooling |
| `priya` | Senior frontend | sonnet | Read, Grep, Glob, Bash, Edit, Write | UI/UX, accessibility, responsive design, component consistency, form UX, print layouts |
| `simone` | Senior security | opus | Read, Grep, Glob, Bash, WebFetch | Security audits, input validation, auth, secrets, dependencies, CORS/CSP. Advisory only — no Edit/Write. |
| `dev` | Senior quality | sonnet | Read, Grep, Glob, Bash, Edit, Write | Testing strategy, test design, edge cases, CI quality gates |

### Why these models

- **sonnet** for the generalist roles. It is the default workhorse — strong at code reasoning, fast enough for interactive review, cheaper than opus.
- **opus** for `simone`. Security findings have the highest cost of being wrong (false negatives ship vulnerabilities, false positives waste team time on theater). Worth the extra cost for deeper reasoning.

If you want to tune cost, downgrade `priya` and `dev` to haiku for narrower review work — the personas hold up at smaller sizes for opinion-style feedback. Keep `nate` on sonnet or higher because architecture review needs broader context.

### Why these tools

- **All four** get `Read`, `Grep`, `Glob` for codebase exploration and `Bash` for running tests, builds, and CLI tools (`gh`, `npm audit`, etc.).
- **`nate`, `priya`, `dev`** get `Edit` and `Write` because they implement code in their domain (refactors, UI fixes, tests).
- **`simone` does NOT get `Edit` or `Write`.** Auth and security code should be reviewed by a human before changes land. Simone recommends fixes; a human or another agent applies them. `WebFetch` is included for CVE and advisory lookups.

## Install

This repo doubles as a one-plugin marketplace. The marketplace manifest is at `.claude-plugin/marketplace.json` and the plugin manifest is at `.claude-plugin/plugin.json`.

### Option 1: Local install (no GitHub needed)

In any Claude Code session, register this directory as a marketplace and install:

```
/plugin marketplace add /Users/kellyheard/Development/claude-virtual-team
/plugin install virtual-team@heard-plugins
```

The plugin updates whenever you `git pull` (or just edit files locally). To pick up changes:

```
/plugin marketplace update heard-plugins
```

### Option 2: Install from GitHub

Once pushed to `github.com/kellyheard/claude-virtual-team`:

```
/plugin marketplace add kellyheard/claude-virtual-team
/plugin install virtual-team@heard-plugins
```

### Option 3: Dev mode (no install, single session only)

```bash
claude --plugin-dir /Users/kellyheard/Development/claude-virtual-team
```

Loads the plugin into the current session only. Good for iterating on agent prompts.

### Option 4: Manual symlink (bypass the plugin system)

If you don't want to use `/plugin` at all:

```bash
mkdir -p ~/.claude/agents
for a in nate priya simone dev; do
  ln -sf /Users/kellyheard/Development/claude-virtual-team/agents/$a.md ~/.claude/agents/$a.md
done
```

### Option 5: Per-project copy

Drop the agent files into a project's `.claude/agents/` directory. Use this when you want to extend an agent with project-specific context (see "Extending per-project" below).

## Usage

Invoke by name in any Claude Code session:

```
@nate review the state management approach in src/store/
@priya audit this form for keyboard accessibility
@simone audit input validation on the file upload endpoint
@dev what's the highest-risk thing to test first in this codebase?
```

Or ask for a multi-agent review:

```
Full team review of the auth refactor before I merge it.
```

The triggering assistant should fan out to the relevant agents based on what changed.

## Extending per-project

Project-level agents in `<repo>/.claude/agents/` override globals with the same name. Use this to layer project-specific context onto a persona without forking the plugin:

```markdown
---
name: nate
description: Senior full-stack engineer (project-aware variant for Acme app).
tools: Read, Grep, Glob, Bash, Edit, Write
model: sonnet
---

[Inherit Nate's persona conceptually, then add:]

## Project Context

- **Stack:** React 19 + Vite + TypeScript, Postgres via Supabase
- **State:** Context + reducer pattern in src/state/
- **Known open issues:** see project-files/code-review-findings.md
```

Or, to leave the persona untouched but add project context, just keep the plugin's global agent and put project-specific notes in `CLAUDE.md` or a doc the agent reads on demand.

## Workflow conventions (recommended)

The plugin only ships the agents. It does **not** make the root assistant route to them automatically — that behavior comes from your `CLAUDE.md`. If you want Claude to always answer as a named team member instead of a generic voice, add the rules below to either `~/.claude/CLAUDE.md` (applies everywhere) or your project `CLAUDE.md` (applies in that repo).

The full version of this snippet, including the routing table, lives in [`TEAM_WORKFLOW.md`](./TEAM_WORKFLOW.md). Copy that file's contents into your `CLAUDE.md`, or reference it via `@TEAM_WORKFLOW.md` in a CLAUDE.md if you've vendored the plugin into your repo.

Short version (full version in `TEAM_WORKFLOW.md`):

```markdown
## Virtual Team Workflow (MANDATORY)

EVERY response must come from a named team member (Nate, Priya, Simone, or Dev).
Never respond as a generic AI.

1. Begin every response with the member's name: `**Nate:** ...`
2. If no member is named, auto-route by request type:
   - Architecture / state / bugs / general questions -> Nate
   - UI / accessibility / forms -> Priya
   - Security / auth / input validation -> Simone
   - Testing / quality / edge cases -> Dev
3. Cross-cutting requests get multiple members in sequence. Surface
   disagreement plainly; don't synthesize it away.
4. Plan before code. Always present a plan and get confirmation before
   writing code, even if the approach seems obvious.
5. Project-level agents in `<repo>/.claude/agents/` override the plugin's
   globals by name.
```

## Design notes

- **Each agent stays in its lane.** Constraints sections explicitly tell the agent what to defer. This prevents Nate from doing a security audit poorly when Simone exists.
- **Disagreement is the point.** Nate will push back on Priya's UI complexity; Priya will push back on Nate's "ship it" instinct. Surface the disagreement, don't paper over it.
- **Agents recommend more than they decide.** Even the implementers (`nate`, `priya`, `dev`) are framed as reviewers first. The user should be the one merging the recommendations.

## License

MIT
