# Team Knowledge

> Part of the [TeamAI CLI Usage Guide](../usage-guide.md).

---

Write down what this session learned. Search it next time with `teamai recall`.

## Knowledge Capture & Retrieval

### Contributing knowledge

When you correct the AI, deny a tool call, or the AI retries a failing command, the session ends with a prompt to share it:

```text
[teamai] This session may contain a problem worth documenting: you interrupted the AI twice.

Consider running `/teamai share what this session taught me` to summarize what you learned and share it with your team (or run `teamai skill get share`).
```

Or hand in a note yourself:

```bash
teamai contribute --file /tmp/session.md
teamai contribute --file /tmp/session.md --namespace payments
```

`--namespace` must be a learnings namespace this scope declares in `manifest/projects.yaml`.

To stop the prompt, set this in `teamai.yaml`:

```yaml
sharing:
  contributeHint:
    enabled: false
```

On one machine, set `contributeHintEnabled: false` in `~/.teamai/config.yaml`, or `TEAMAI_CONTRIBUTE_HINT_DISABLED=1`.

### Searching knowledge

```bash
teamai recall "API timeout"
teamai recall --check "API timeout"
```

`--check` answers only `RELEVANT` or `NOT_RELEVANT`. A codebase hit may add a `Source freshness:` line. The statuses are in the [Codebase Knowledge Graph](./codebase-graph.md).

### Recall adoption and upvotes

Opening a document that recall returned counts as one upvote for this session. You can also record it by hand:

```bash
teamai recall feedback --positive <docId>
teamai recall feedback --negative <docId>
```

When the log has runs, `teamai stats` lists the last 10 sessions that ran recall.

### Enabling / Disabling Recall

Recall is off until the team turns it on:

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

One machine overrides this with `recallEnabled` in `~/.teamai/config.yaml`. `TEAMAI_RECALL_DISABLED=1` forces it off. While it is off, pull does not install recall. `teamai recall` still searches.

### Knowledge Base Maintenance

```bash
teamai recall maintenance --prune --dry-run
teamai recall maintenance --prune --archive
teamai recall maintenance --update-quality
```

`--archive` puts aside learnings whose confidence is below 0.15. `--update-quality` writes a `.draft.md`. Rename it to `.md` when the draft is right.

### Promoting Learnings

A learning can become a skill, rule, or doc when its confidence is at least 0.90, it has at least 5 upvotes, at least 2 contributors, and it has existed for at least 14 days.

```bash
teamai recall promote
teamai recall promote <learningId> --category skills
teamai recall promote <learningId> --dry-run
```

## Knowledge Base Health Report

```bash
teamai dashboard
```

Open Team Context, or go to `http://localhost:3721/kb-report`.
