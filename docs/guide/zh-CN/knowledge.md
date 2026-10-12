# 团队知识

> 本文是 [TeamAI CLI 使用指南](../../usage-guide.zh-CN.md) 的一部分。

---

记下这次会话学到的东西。下次用 `teamai recall` 搜。

## 知识沉淀与检索

### 贡献知识

会话里纠正过 AI、拒绝过工具调用，或 AI 反复重试失败的命令时，结束时会提示把这次经验交给团队：

```text
[teamai] This session may contain a problem worth documenting: you interrupted the AI twice.

Consider running `/teamai share what this session taught me` to summarize what you learned and share it with your team (or run `teamai skill get share`).
```

也可以直接交一份笔记：

```bash
teamai contribute --file /tmp/session.md
teamai contribute --file /tmp/session.md --namespace payments
```

`--namespace` 必须是当前 scope 在 `manifest/projects.yaml` 里声明的 learnings namespace。

不想要这条提醒时，在 `teamai.yaml` 里写：

```yaml
sharing:
  contributeHint:
    enabled: false
```

本机覆盖：`~/.teamai/config.yaml` 的 `contributeHintEnabled: false`，或 `TEAMAI_CONTRIBUTE_HINT_DISABLED=1`。

### 搜索知识

```bash
teamai recall "API 超时"
teamai recall --check "API 超时"
```

`--check` 只回答 `RELEVANT` 或 `NOT_RELEVANT`。代码库结果可能多一行 `Source freshness:`，状态含义见[代码知识图谱](./codebase-graph.md)。

### Recall 采纳与 upvote

这次会话打开了 recall 返回的文档，就计一次 upvote。也可以手动记：

```bash
teamai recall feedback --positive <docId>
teamai recall feedback --negative <docId>
```

`teamai stats` 在有记录时，会列出最近 10 个跑过 recall 的会话。

### 开启 / 关闭 Recall

默认关。团队打开：

```yaml
sharing:
  recall:
    enabled: true
```

```bash
teamai recall enable
teamai recall disable
teamai recall status
```

本机覆盖写在 `~/.teamai/config.yaml` 的 `recallEnabled`。`TEAMAI_RECALL_DISABLED=1` 强制关掉。关掉之后 pull 不再安装 recall；直接运行 `teamai recall` 仍然可以搜。

### 知识库维护

```bash
teamai recall maintenance --prune --dry-run
teamai recall maintenance --prune --archive
teamai recall maintenance --update-quality
```

`--archive` 收走置信度低于 0.15 的 learning。`--update-quality` 写出 `.draft.md`，改好后把文件名改成 `.md`。

### 晋升 Learnings

置信度至少 0.90、至少 5 次 upvote、至少 2 个贡献者、存在至少 14 天，可以升成 skill、rule 或 doc。

```bash
teamai recall promote
teamai recall promote <learningId> --category skills
teamai recall promote <learningId> --dry-run
```

## 知识库健康报告

```bash
teamai dashboard
```

打开后进 Team Context，或直接访问 `http://localhost:3721/kb-report`。
