# 参考

> 本文是 [TeamAI CLI 使用指南](../../usage-guide.zh-CN.md) 的一部分。

---

## 命令参考

完整的命令与 flag 列表由 CLI 自身生成：见 [`skill-data/core/references/commands.md`](../../../skill-data/core/references/commands.md)，或运行 `teamai --help` 和 `teamai <command> --help`。

---

## 配置文件参考

### teamai.yaml（远端团队配置）

```yaml
team: my-team
description: 团队 AI 资源仓库
repo: https://github.com/your-org/your-repo.git
provider: github
# scope: 若存在则忽略——本机安装位置由 `teamai init --scope` 决定

reviewers:
  - reviewer1

packages:
  npm:
    - name: typescript
      version: "*"

sharing:
  rules:
    enforced: [code-review-guide]
  recall:
    enabled: false             # 可选；成员可在本地覆盖
  docs:
    localDir: ./.teamai/docs   # 默认 ~/.teamai/docs；project scope 下 ~/ 前缀解析为项目根目录
  env:
    injectShellProfile: true
  coAuthor:
    enabled: false             # 可选，为全团队去除 AI 工具提交尾注
  gitExclude:
    enabled: true              # 可选，让分发的文件不进入 git（init 为新团队写入 true）
  contributeHint:
    enabled: true              # 可选，false = 高摩擦 session 结束后不再提示 /teamai
  intervention:
    correctionKeywords: []     # 可选，额外的纠偏词，与内置中/英/日列表合并
  webhooks:                    # 可选，在团队事件发生时通知外部端点（见"Webhook 通知"）
    enabled: true
    endpoints:
      - url: https://example.com/hook
        type: json             # json | feishu | wecom
        events: ["*"]          # 可取：session-start、session-stop、skill-use、push、pull，或 "*" 表示全部
        secret: my-signing-key # 可选，设置后启用 X-TeamAI-Signature 头
        timeout: 5000          # 可选，单次请求超时（毫秒，默认 5000）
        retries: 3             # 可选，失败重试次数（默认 3）
```

`teamai pull` 把你收到的 `docs/` 同步到 `sharing.docs.localDir`（见 [Docs（文档）](./sharing.md#docs文档)）。团队删掉的文档，本地同一份也会删。你自己改过、对不上团队历史的文件会留下，pull 打出 `Kept <path>`。`localDir` 本身是链接时不往里写。`teamai pull --dry-run` 只预览。

### config.yaml（本地配置）

```yaml
repo:
  localPath: /path/to/.teamai/team-repo
  remote: https://github.com/your-org/your-repo.git
username: your-name
updatePolicy: auto
scope: project                 # project（init 默认）或 user
projectRoot: /path/to/project  # 仅 project scope
inheritUserScope: true         # 可选，仅 project scope，默认 false
coAuthorEnabled: true          # 可选，每机器的 co-author 覆盖
gitExcludeEnabled: true        # 可选，每机器对 sharing.gitExclude.enabled 的覆盖
contributeHintEnabled: false   # 可选，每机器覆盖 sharing.contributeHint.enabled
usageReportEnabled: false      # 可选，每机器覆盖 usageReport
codexTrustEnabled: false       # 可选，每机器，停止 teamai 信任它写入的 Codex hooks 与项目（见 Hooks）
toolRoots:                     # 可选，每机器的工具根目录（见下）
  claude: ~/.claude-work
  codex: ~/.codex-alt
```

#### 迁移后的工具根目录（`toolRoots`）

Claude Code 用 `CLAUDE_CONFIG_DIR`、Codex 用 `CODEX_HOME` 把配置放到别处时，在 `toolRoots` 里写那个目录。目前只认 `claude` 和 `codex`。

```bash
CLAUDE_CONFIG_DIR=~/.claude-work teamai init
```

`init` 会记下它指向的目录。换目录后再跑一次 `init`。清掉记录：

```bash
CLAUDE_CONFIG_DIR= teamai init
CODEX_HOME= teamai init
```

目录要在 home 下（`~/.claude-work`），或在 `~/.config/` 下面。Copilot 用自己的 `COPILOT_HOME`。改了环境变量之后 `teamai doctor` 会对不上，按它的提示再跑一次 `init`。

### Webhook 通知（`sharing.webhooks`）

在团队事件发生时通知外部端点。每个 endpoint 声明 `url`、`type`（`json`、`feishu` 或 `wecom`）以及订阅的 `events`；`secret`、`timeout`（默认 `5000` 毫秒）、`retries`（默认 `3`）均为可选。

**事件及触发时机：**

| 事件 | 触发时机 |
| --- | --- |
| `session-start` | AI session 开始 |
| `session-stop` | AI session 结束（含 Copilot 的 `SessionEnd`） |
| `skill-use` | 调用某个 skill |
| `push` | `teamai push` **真正完成一次推送**——`--dry-run`、取消选择、无变更、或 PR 创建失败都不触发 |
| `pull` | `teamai pull` 完成一次真实（非 `--dry-run`）同步——暂停了无法解析模型的 agent 时不触发 |
| `*` | 通配符——订阅以上全部事件 |

**载荷。** 仅发送白名单内的非敏感字段：`skill-use` 发送 `skillName`，session 事件发送 `sessionId`；`push`/`pull` 只带事件与元数据。原始工具入参与工具输出**绝不**外发，且整个请求体在离开本机前会经过 teamai 的密钥脱敏处理。

**签名。** 设置 `secret` 后，每个请求都会带上 `X-TeamAI-Signature: sha256=<hmac>`——对**实际发送的请求体**计算的 HMAC-SHA256，供接收端校验真实性。`teamai webhook list` 与 `teamai webhook test` 可查看和测试已配置的端点；`teamai webhook test --dry-run` 只预览将发送到多少个端点，不发送请求。

---

## 模型配置

模型配置让 Claude Code、Codex、OpenCode、CodeBuddy、WorkBuddy、Pi 和 OMP 使用同一个模型网关。只有执行 `teamai models switch` 才会修改 Agent 配置；切换之后，`teamai pull` 会让已切换的 Agent 跟随团队目录的最新内容。

配置有两个来源，格式完全相同：

- `team:<id>` 来自团队仓库的 `models/models.yaml`，以及你当前生效 namespace 的 `models/<ns>/models.yaml`（见[团队配置按 namespace 划分](#团队配置按-namespace-划分)），只包含 URL 和模型 ID，不包含密钥。
- `local:<id>` 是 `~/.teamai/models/models.yaml` 中的个人配置，只在本机可见。

ID 唯一时可直接写 `<id>`；团队和个人配置同名时，需写成 `team:<id>` 或 `local:<id>`。

### 团队目录

在团队仓库中创建 `models/models.yaml`：

```yaml
profiles:
  - id: tokenhub
    name: Tencent TokenHub
    base_url: https://tokenhub.tencentmaas.com
    api_key: ${API_KEY}          # 占位符；每位成员在本地配置真实密钥
    model_groups:
      - protocols: [anthropic, openai-chat-completions]
        models:
          - glm-5.3               # 第一个模型是默认模型
          - deepseek-v4-flash
```

- `base_url` 是网关根地址。`anthropic` 协议直接使用该地址，OpenAI 协议在后面加 `/v1`，与 [TokenHub](https://cloud.tencent.com/document/product/1823/130078) 一致。
- `protocols` 声明该组模型支持的协议：`anthropic`、`openai-chat-completions`、`openai-responses`。协议支持不同的模型放在不同分组，每个模型 ID 只出现一次。
- `api_key` 必须写成 `${API_KEY}`。未知字段、重复模型 ID，以及带凭证、查询参数或片段的 URL 都会被拒绝；`teamai push` 会拦截无效目录。

哪些 Agent 可以使用由协议决定：

| Agent | 需要的协议 | `switch` 写入的内容 |
| --- | --- | --- |
| Claude Code | `anthropic` | `~/.claude/settings.json`：`env` 中的网关地址和密钥；全部模型进入 `/model` 选择器；`opus`/`sonnet`/`haiku` 映射到网关中名称匹配的模型，否则映射到默认模型 |
| Codex | `openai-responses` | `~/.codex/config.toml`：默认模型和 `[model_providers.teamai]` 块 |
| OpenCode | 任意 | `opencode.json`：每种协议一个 provider，包含全部模型 |
| CodeBuddy / WorkBuddy | `openai-chat-completions` | `models.json`：每个模型一个条目 |
| Pi | 任意 | `~/.pi/agent/models.json`：一个以 profile 引用为键的 provider，包含全部模型。不改动 `settings.json`，默认模型由你用 `/model` 选择 |
| OMP | 任意 | `~/.omp/agent/models.yml`：一个以 profile 引用为键的 provider，包含全部模型——与 Pi 结构相同，只是 YAML |

上例没有 `openai-responses` 分组，因此不会修改 Codex；确认网关的 Responses 接口支持这些模型后，再加上该协议即可。

### 使用团队配置

```bash
teamai models list                     # 全部配置：来源文件、密钥来源、网关、模型、Agent 及生效位置
teamai models list tokenhub            # 只看一个配置
teamai models switch tokenhub          # 首次使用时提示输入密钥
teamai models switch                   # 列出全部配置，询问使用哪一个
```

`switch` 不带配置名时会列出全部配置（团队配置在前），并切换到你所选的那个；输入 `none` 可取消。它一次只接受一个配置，因此填了多个会重新询问，而不会静默取第一个。没有终端时无处可选，因此此时必须给出配置名。

`switch` 会更新所有已安装且兼容的 Agent。可以用 `--agent claude`（可重复）缩小范围，用 `--model deepseek-v4-flash` 指定默认模型，用 `--dry-run` 预览。

如果不想保存密钥，可以改为引用环境变量：

```bash
teamai models configure tokenhub --from-env TOKENHUB_API_KEY
printf '%s' "$TOKENHUB_API_KEY" | teamai models configure tokenhub --api-key-stdin
```

Codex、OpenCode、CodeBuddy 和 WorkBuddy 会自行读取该变量。Claude Code 不支持，所以 `switch` 会把解析后的密钥写入 `~/.claude/settings.json`。命令刻意不提供 `--api-key <值>`，因为命令参数会进入 shell 历史和进程列表。密钥文件权限为 `0600`。

团队修改目录后，`teamai pull` 会把新内容重新应用到已切换到该配置的 Agent。

### 团队配置按 namespace 划分

项目或角色可以为某个团队配置提供自己的版本，例如让 checkout 成员在同一个 `id` 下使用 checkout 网关。把它放在 `models/<ns>/models.yaml`，并在 `resources.models` 中声明该 namespace，方式与 env、hooks 和 MCP server 相同（见 [Env、hooks 与 MCP server 按 namespace 划分](./sharing.md#envhooks-与-mcp-server-按-namespace-划分)）：

```yaml
# manifest/projects.yaml
projects:
  - id: checkout
    resources:
      models: [checkout]
```

- **覆盖。** `checkout` 生效期间，`models/checkout/models.yaml` 中的配置整体替换根目录中 `id` 相同的配置。已切换到 `team:<id>` 的 Agent 在下次 pull 时跟随它；namespace 失效后回到根配置。只存在于你已离开的 namespace 中的配置不会从 Agent 中移除：pull 会提示它 `is no longer active in your namespaces`，可用 `teamai models restore` 撤销。
- **密钥只用于它所属的网关。** 团队配置的 API 密钥按配置 `id` 和 `base_url` 的 origin（协议、主机和端口）保存。覆盖把配置指向另一个 origin 时，pull 不会修改使用它的 Agent，并提示运行 `teamai models switch team:<id>`，该命令会询问新网关的密钥（也可以先运行 `teamai models configure team:<id>`）。原网关的密钥会保留，因此离开 namespace 时无需重新输入。团队把根配置改到另一个 origin 时同样如此。
- **冲突只停止 models，不影响整个 pull。** 两个生效 namespace 中出现同一个 `id`，或某个生效文件无法解析时，本次不会更新任何 Agent，警告会指出相关文件。`teamai push` 会拒绝任何无效的 models 文件。
- `teamai models list` 显示每个团队配置来自哪个文件、是否覆盖根配置；`teamai doctor` 把每个覆盖列为提示。

### 个人配置

```bash
teamai models add my-gateway --name "My gateway" \
  --protocol anthropic,openai-chat-completions \
  --base-url https://gateway.example.com \
  --model glm-5.3,deepseek-v4-flash \
  --from-env MY_GATEWAY_KEY
teamai models switch my-gateway
```

省略参数时会交互输入。用 `configure` 修改个人配置：`--name`、`--base-url`、`--model`（追加模型）和 `--protocol`（让模型额外支持某协议；配合 `--model` 可只作用于这些模型）。也可以直接编辑 `~/.teamai/models/models.yaml`。个人配置的 ID 不能与团队配置重名。

### 恢复

```bash
teamai models restore                  # 所有被 TeamAI 切换过的 Agent
teamai models restore --agent codex
```

TeamAI 只修改自己管理的字段和条目，并记录首次切换前的值，`restore` 会还原这些值。如果你自己改了受管字段（例如 Claude `env` 中的网关地址），之后的切换、pull 和恢复都会跳过该 Agent。在 Claude 中用 `/model` 选择其他模型不算接管。Codex 的 `~/.codex/auth.json` 永远不会被修改。

Claude 注意事项：`settings.json` 启用了 Bedrock、Vertex 或 Foundry 时，`switch` 会拒绝切换。当前 shell 导出的 `ANTHROPIC_*` 与 TeamAI 写入的值不一致时会给出警告，因为从该 shell 启动的会话仍会使用这些值。

其他命令：

```bash
teamai models remove local:my-gateway  # Agent 保留当前配置，restore 仍然可用
```

用户级完整 `teamai uninstall` 会先恢复受管的模型配置；如有无法恢复的配置，会停止卸载并保留恢复记录。项目级卸载不改动这些机器级配置。
