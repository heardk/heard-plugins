# heard-plugins

A Claude Code plugin marketplace by Kelly Heard.

## Plugins

| Plugin | Description |
|---|---|
| [`virtual-team`](./plugins/virtual-team) | Four named subagents (full-stack, frontend, security, quality) for code reviews, architecture feedback, and implementation. |

## Install

In any Claude Code session:

```
/plugin marketplace add heardk/heard-plugins
/plugin install virtual-team@heard-plugins
```

For local development without GitHub:

```
/plugin marketplace add /path/to/heard-plugins
/plugin install virtual-team@heard-plugins
```

To pick up changes after editing files locally:

```
/plugin marketplace update heard-plugins
```

## Layout

```
heard-plugins/
├── .claude-plugin/
│   └── marketplace.json         ← lists every plugin in this marketplace
├── plugins/
│   └── virtual-team/            ← one plugin
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── agents/
│       ├── README.md
│       └── TEAM_WORKFLOW.md
└── README.md                    ← this file
```

## Adding a new plugin

1. Create `plugins/<name>/` with its own `.claude-plugin/plugin.json` and any `agents/`, `commands/`, `hooks/` directories the plugin needs.
2. Add an entry to `.claude-plugin/marketplace.json`'s `plugins` array:
   ```json
   {
     "name": "<name>",
     "source": "./plugins/<name>",
     "description": "..."
   }
   ```
3. Commit and push. Existing installs pick up the new plugin via `/plugin marketplace update heard-plugins`.

## License

MIT
