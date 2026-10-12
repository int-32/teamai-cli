# Hooks

> Part of the [TeamAI CLI Usage Guide](../usage-guide.md).

---

`teamai init` installs these built-in hooks:

| Event | What it does |
|-------|----------------|
| `SessionStart` | Pull team resources and record the session start |
| `PostToolUse` | Record skill use |
| `UserPromptSubmit` | Record the prompt |
| `Stop` | Check for a CLI update and record the session end |

```bash
teamai hooks list
teamai hooks inject --dry-run
teamai hooks inject
teamai hooks remove
```

Only tools that are already installed are touched.

## Team Hooks Declaration

Commit `hooks/hooks.yaml` in the team repo. A hook for some members only goes in `hooks/<ns>/hooks.yaml`. See [Env, hooks and MCP servers by namespace](./sharing.md#env-hooks-and-mcp-servers-by-namespace).

```yaml
hooks:
  - id: block-secret
    description: Scan for secrets before a commit
    event: PreToolUse
    matcher: Bash
    command: bash ~/.teamai/team-scripts/scan-secret.sh
    timeout: 15
    tools: [claude, cursor]
```

`id` is lowercase letters, digits, and `-`. `builtin` belongs only in the root file. Use it to turn a built-in hook off, or to change its timeout.

```yaml
sharing:
  hooks:
    autoApply: false          # pull only prints a hint; run teamai hooks inject
    requireTeamScripts: true  # command must live under ~/.teamai/team-scripts/
```

Turn team hooks off on this machine with `TEAMAI_HOOKS_DISABLED=1`.
