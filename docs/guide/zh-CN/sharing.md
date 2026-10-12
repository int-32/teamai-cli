# 共享团队资源

> 本文是 [TeamAI CLI 使用指南](../../usage-guide.zh-CN.md) 的一部分。

---

## Skills（技能）

```bash
mkdir -p ~/.claude/skills/my-deploy-helper
cat > ~/.claude/skills/my-deploy-helper/SKILL.md << 'EOF'
---
name: my-deploy-helper
description: 帮助团队部署服务
---
# Deploy Helper
当用户请求部署时，按以下步骤执行：
1. 检查当前分支是否为 master
2. 运行测试 `npm test`
3. 构建 `npm run build`
4. 部署 `./deploy.sh`
EOF

teamai push
teamai push --role pm
```

没有 frontmatter 时，`push` 会补上 `name` 和 `description`。默认写到 `skills/<主角色>/<name>/`，`--role` 改写到指定角色。

## Rules（规则）

```bash
cat > ~/.claude/rules/code-review-guide.md << 'EOF'
# 代码审查规范
- 所有函数必须有 JSDoc 注释
- 禁止使用 `any` 类型
EOF

teamai push
```

成员不能删的 rule，写在 `teamai.yaml`：

```yaml
sharing:
  rules:
    enforced: [code-review-guide]
```

你自己的 rule 会留下。`push` 只把已有团队 rule 的正文写回仓库。

## Env、hooks 与 MCP server 按 namespace 划分

根目录的文件给所有人。`resources:` 里声明的 namespace 只给激活了它的成员，并替换根目录里的同名条目。

```text
env/env.yaml              hooks/hooks.yaml              mcp/mcp.yaml
env/<ns>/env.yaml         hooks/<ns>/hooks.yaml         mcp/<ns>/mcp.yaml
```

```yaml
# manifest/projects.yaml
projects:
  - id: checkout
    resources:
      env: [checkout]
      hooks: [checkout]
      mcp: [checkout]
```

## Env（环境变量）

```bash
teamai env add API_ENDPOINT https://api.example.com --description "团队 API 地址"
teamai env add API_ENDPOINT https://checkout.internal --project checkout
teamai env add GITHUB_TOKEN --secret -d "GitHub token with repo scope" --url https://github.com/settings/tokens
teamai push
```

```yaml
# env/env.yaml
variables:
  - key: API_ENDPOINT
    value: https://api.example.com
    description: 团队 API 地址
```

```yaml
# env/secrets.yaml
secrets:
  - key: GITHUB_TOKEN
    description: GitHub token with repo scope
    url: https://github.com/settings/tokens
```

密钥的值不进仓库。每人在自己机器上：

```bash
teamai env set GITHUB_TOKEN
teamai env exec -- gh pr create
```

## Docs（文档）

把文件放进团队仓库的 `docs/`，提交后成员 `pull` 时同步。某个目录只给一部分人时，在 `resources.docs` 里声明：

```yaml
# manifest/projects.yaml
projects:
  - id: checkout
    resources:
      docs: [checkout]
```

## MCP Server

```yaml
# mcp/mcp.yaml
servers:
  - name: gpu-analysis
    description: GPU 存量与价格查询
    transport: http
    url: https://example.com/api/mcp
    headers:
      Authorization: Bearer ${GPU_ANALYSIS_TOKEN}
    timeout: 600000
  - name: local-formatter
    transport: stdio
    command: npx
    args: ["-y", "@acme/formatter-mcp"]
    env:
      FORMATTER_MODE: strict
    requires: [npx]
    tools: [claude, cursor]
```

密钥写 `${VAR}`，并在 `env/secrets.yaml` 里声明。只给一个项目时，写到 `mcp/<ns>/mcp.yaml`。

```bash
teamai mcp list
teamai mcp inject
teamai mcp remove
```
