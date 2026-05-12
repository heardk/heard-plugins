# virtual-team

A Claude Code plugin (distributed via the `heard-plugins` marketplace) that installs four named subagents — `nate`, `priya`, `simone`, `dev` — covering full-stack, frontend, security, and quality. Each has a distinct voice, scope, and review protocol, and is told to stay in its lane and defer outside it.

## Roster

All four agents are advisory-only — they have read/search/Bash tools but no `Edit` or `Write`. They investigate and recommend; the root assistant implements after the user approves the plan.

| Agent | Role | Model | Tools | Use for |
|---|---|---|---|---|
| `nate` | Senior full-stack | sonnet | Read, Grep, Glob, Bash | Architecture, state management, type design, performance, refactoring, build tooling |
| `priya` | Senior frontend | sonnet | Read, Grep, Glob, Bash | UI/UX, accessibility, responsive design, component consistency, form UX, print layouts |
| `simone` | Senior security | opus | Read, Grep, Glob, Bash, WebFetch | Security audits, input validation, auth, secrets, dependencies, CORS/CSP |
| `dev` | Senior quality | sonnet | Read, Grep, Glob, Bash | Testing strategy, test design, edge cases, CI quality gates |

### Why these models

- **sonnet** for the generalist roles. It is the default workhorse — strong at code reasoning, fast enough for interactive review, cheaper than opus.
- **opus** for `simone`. Security findings have the highest cost of being wrong (false negatives ship vulnerabilities, false positives waste team time on theater). Worth the extra cost for deeper reasoning.

If you want to tune cost, downgrade `priya` and `dev` to haiku for narrower review work — the personas hold up at smaller sizes for opinion-style feedback. Keep `nate` on sonnet or higher because architecture review needs broader context.

### Why these tools

- **All four** get `Read`, `Grep`, `Glob` for codebase exploration and `Bash` for running tests, builds, typecheck, lint, and CLI tools (`gh`, `npm audit`, etc.) as part of investigation.
- **None** get `Edit` or `Write`. The agents are advisory — they return findings and recommended diffs; the root assistant applies them after the user approves the plan. This enforces "plan before building" structurally: a subagent literally cannot ship code without going through the root.
- **`simone`** additionally gets `WebFetch` for CVE and advisory lookups.

## Install

This plugin lives in the [`heard-plugins`](https://github.com/heardk/heard-plugins) marketplace.

### Recommended

```
/plugin marketplace add heardk/heard-plugins
/plugin install virtual-team@heard-plugins
```

For local development without GitHub:

```
/plugin marketplace add /path/to/heard-plugins
/plugin install virtual-team@heard-plugins
```

### Dev mode (single session, no install)

```bash
claude --plugin-dir /path/to/heard-plugins/plugins/virtual-team
```

### Manual symlink (bypass the plugin system)

```bash
mkdir -p ~/.claude/agents
for a in nate priya simone dev; do
  ln -sf /path/to/heard-plugins/plugins/virtual-team/agents/$a.md ~/.claude/agents/$a.md
done
```

### Per-project copy

Drop the agent files into a project's `.claude/agents/` directory. Use this when you want to extend an agent with project-specific context (see "Extending per-project" below).

## Usage

Invoke by name in any Claude Code session. Plugin-installed agents use the namespaced form `@<plugin>:<agent>`:

```
@virtual-team:nate review the state management approach in src/store/
@virtual-team:priya audit this form for keyboard accessibility
@virtual-team:simone audit input validation on the file upload endpoint
@virtual-team:dev what's the highest-risk thing to test first in this codebase?
```

Or ask for a multi-agent review:

```
Full team review of the auth refactor before I merge it.
```

The triggering assistant should fan out to the relevant agents based on what changed.

The `virtual-team:` prefix prevents collision with project-level agents that share a short name. Project agents in `<repo>/.claude/agents/<name>.md` are invoked as plain `@<name>` and take precedence for project-domain decisions; plugin agents are the fallback.

## Extending per-project

Project-level agents in `<repo>/.claude/agents/` are invoked as plain `@<name>` (e.g. `@nate`) and win over the plugin's `@virtual-team:nate` for project work. Use this to layer project-specific context onto a persona without forking the plugin:

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
   - Architecture / state / bugs / general questions -> Nate (`@virtual-team:nate`)
   - UI / accessibility / forms -> Priya (`@virtual-team:priya`)
   - Security / auth / input validation -> Simone (`@virtual-team:simone`)
   - Testing / quality / edge cases -> Dev (`@virtual-team:dev`)
3. Cross-cutting requests get multiple members in sequence. Surface
   disagreement plainly; don't synthesize it away.
4. Plan before code. Always present a plan and get confirmation before
   writing code, even if the approach seems obvious.
5. Project-level agents in `<repo>/.claude/agents/` (invoked as plain
   `@<name>`) win over the plugin's `@virtual-team:<name>` for project work.
```

## Design notes

- **Each agent stays in its lane.** Constraints sections explicitly tell the agent what to defer. This prevents Nate from doing a security audit poorly when Simone exists.
- **Disagreement is the point.** Nate will push back on Priya's UI complexity; Priya will push back on Nate's "ship it" instinct. Surface the disagreement, don't paper over it.
- **Agents recommend more than they decide.** Even the implementers (`nate`, `priya`, `dev`) are framed as reviewers first. The user should be the one merging the recommendations.

## License

MIT
