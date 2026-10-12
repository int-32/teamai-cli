# 跨团队 Skill 订阅

> 本文是 [TeamAI CLI 使用指南](../../usage-guide.zh-CN.md) 的一部分。

---

订阅别的团队公开的 skills。对方在自己的 `teamai.yaml` 里列出要公开的：

```yaml
publicSkills:
  - deploy-helper
```

```bash
teamai source add https://github.com/other-team/teamai-public.git --name other-team
teamai source list
teamai source browse other-team
teamai source remove other-team
teamai push
```

`pull` 会装上这些 skill。`push` 合入后，其他成员的 `pull` 也会订上。没写 `publicSkills` 时，这个源是空的。

## HTTP 源

```bash
teamai source add-http https://your-team-host/api --token <api-key>
teamai source list
teamai source remove-http
```

每个安装只能有一个 HTTP 源。主仓已经是 `init --http` 时，不能再 `add-http`。接口见 [HTTP 契约](./http-contract.md)。
