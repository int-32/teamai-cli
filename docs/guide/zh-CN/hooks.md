# Hooks

> 本文是 [TeamAI CLI 使用指南](../../usage-guide.zh-CN.md) 的一部分。

---

`teamai init` 会装上这些内置 hook：

| 事件 | 做什么 |
|------|--------|
| `SessionStart` | 拉取团队资源，并记下会话开始 |
| `PostToolUse` | 记录 skill 使用 |
| `UserPromptSubmit` | 记录输入 |
| `Stop` | 检查 CLI 更新，并记下会话结束 |

```bash
teamai hooks list
teamai hooks inject --dry-run
teamai hooks inject
teamai hooks remove
```

只处理已经安装的工具。

## 团队 Hooks 声明

写在 `hooks/hooks.yaml`，提交进团队仓库。只给一部分人时写 `hooks/<ns>/hooks.yaml`，见 [Env、hooks 与 MCP server 按 namespace 划分](./sharing.md#envhooks-与-mcp-server-按-namespace-划分)。

```yaml
hooks:
  - id: block-secret
    description: 提交前扫描密钥
    event: PreToolUse
    matcher: Bash
    command: bash ~/.teamai/team-scripts/scan-secret.sh
    timeout: 15
    tools: [claude, cursor]
```

`id` 只用小写字母、数字和 `-`。`builtin` 只放在根文件里，用来关掉某条内置 hook，或改它的超时。

```yaml
sharing:
  hooks:
    autoApply: false          # pull 只提示，要自己跑 teamai hooks inject
    requireTeamScripts: true  # command 必须在 ~/.teamai/team-scripts/ 下
```

本机全部停用团队 hook：`TEAMAI_HOOKS_DISABLED=1`。
