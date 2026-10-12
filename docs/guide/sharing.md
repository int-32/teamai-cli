# Sharing Team Resources

> Part of the [TeamAI CLI Usage Guide](../usage-guide.md).

---

## Skills

```bash
mkdir -p ~/.claude/skills/my-deploy-helper
cat > ~/.claude/skills/my-deploy-helper/SKILL.md << 'EOF'
---
name: my-deploy-helper
description: Deploy a service for the team
---
# Deploy Helper
When the user requests a deployment, follow these steps:
1. Check that the current branch is master
2. Run tests `npm test`
3. Build `npm run build`
4. Deploy `./deploy.sh`
EOF

teamai push
teamai push --role pm
```

`push` fills in `name` and `description` when the frontmatter is missing. The default path is `skills/<primary role>/<name>/`. `--role` writes to that role instead.

## Rules

```bash
cat > ~/.claude/rules/code-review-guide.md << 'EOF'
# Code Review Guidelines
- All functions must have JSDoc comments
- `any` type is not allowed
EOF

teamai push
```

A rule members cannot delete goes in `teamai.yaml`:

```yaml
sharing:
  rules:
    enforced: [code-review-guide]
```

A rule you wrote yourself stays. `push` writes back only the body of a rule the team already has.

## Env, hooks and MCP servers by namespace

The root file reaches everyone. A namespace declared under `resources:` reaches only members who have it active, and replaces the root entry of the same name.

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

## Env (environment variables)

```bash
teamai env add API_ENDPOINT https://api.example.com --description "Team API endpoint"
teamai env add API_ENDPOINT https://checkout.internal --project checkout
teamai env add GITHUB_TOKEN --secret -d "GitHub token with repo scope" --url https://github.com/settings/tokens
teamai push
```

```yaml
# env/env.yaml
variables:
  - key: API_ENDPOINT
    value: https://api.example.com
    description: Team API endpoint
```

```yaml
# env/secrets.yaml
secrets:
  - key: GITHUB_TOKEN
    description: GitHub token with repo scope
    url: https://github.com/settings/tokens
```

Secret values stay off the repo. On each machine:

```bash
teamai env set GITHUB_TOKEN
teamai env exec -- gh pr create
```

## Docs

Put files in the team repo's `docs/`. Members receive them on `pull`. To limit a directory, declare it under `resources.docs`:

```yaml
# manifest/projects.yaml
projects:
  - id: checkout
    resources:
      docs: [checkout]
```

## MCP servers

```yaml
# mcp/mcp.yaml
servers:
  - name: gpu-analysis
    description: GPU inventory and pricing
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

Write secrets as `${VAR}` and declare the key in `env/secrets.yaml`. A server for one project goes in `mcp/<ns>/mcp.yaml`.

```bash
teamai mcp list
teamai mcp inject
teamai mcp remove
```
