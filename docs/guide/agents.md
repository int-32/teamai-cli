# Agents

> Part of the [TeamAI CLI Usage Guide](../usage-guide.md).

---

One file per agent in the team repo: `agents/<name>.yaml`. `name` matches the filename.

```yaml
# agents/code-reviewer.yaml
name: code-reviewer
description: Review a change before it is pushed
instructions: |
  Read the diff and comment on correctness, tests, and risks.
tools: [Read, Grep]
targets: [claude, cursor]
```

Leave `targets` out to send the agent to every installed tool. An agent for one role goes in `agents/<namespace>/`, declared under `resources.agents`.

```bash
teamai push
teamai push --role frontend
teamai remove agents code-reviewer
```

`pull` writes each tool's own format into that tool's agents directory: `<name>.toml` for the Codex family, `<name>.agent.md` for Copilot, `<name>.json` for Kiro, and `<name>.md` for the rest.

## Model aliases

A name such as `model: strong` is mapped per tool in `models/aliases.yaml`:

```yaml
aliases:
  strong:
    claude: { model: opus, effort: high }
    codex: { model: gpt-6-sol, effort: high }
  fast:
    claude: haiku
```

An alias starts with a lowercase letter, then lowercase letters, digits, or `-`. A namespace that needs its own mapping uses `models/<ns>/aliases.yaml`.
