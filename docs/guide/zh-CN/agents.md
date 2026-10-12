# Agents

> 本文是 [TeamAI CLI 使用指南](../../usage-guide.zh-CN.md) 的一部分。

---

团队仓库里一个 agent 一个文件：`agents/<name>.yaml`。`name` 和文件名一致。

```yaml
# agents/code-reviewer.yaml
name: code-reviewer
description: Review a change before it is pushed
instructions: |
  Read the diff and comment on correctness, tests, and risks.
tools: [Read, Grep]
targets: [claude, cursor]
```

`targets` 不写就发给所有已安装的工具。只给一个角色时放到 `agents/<namespace>/`，并在 `resources.agents` 里声明。

```bash
teamai push
teamai push --role frontend
teamai remove agents code-reviewer
```

`pull` 按工具自己的格式写入它的 agents 目录：Codex 系列是 `<name>.toml`，Copilot 是 `<name>.agent.md`，Kiro 是 `<name>.json`，其余是 `<name>.md`。

## 模型别名

`model: strong` 这类名字在 `models/aliases.yaml` 里按工具对应到具体模型：

```yaml
aliases:
  strong:
    claude: { model: opus, effort: high }
    codex: { model: gpt-6-sol, effort: high }
  fast:
    claude: haiku
```

别名以小写字母开头，后面可以是小写字母、数字或 `-`。某个 namespace 要换一套时，写 `models/<ns>/aliases.yaml`。
