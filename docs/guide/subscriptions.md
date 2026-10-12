# Cross-team skill subscriptions

> Part of the [TeamAI CLI Usage Guide](../usage-guide.md).

---

Subscribe to skills another team has made public. That team lists them in its own `teamai.yaml`:

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

`pull` installs these skills. After the `push` merges, every teammate's `pull` subscribes too. A source with no `publicSkills` is empty.

## HTTP Source

```bash
teamai source add-http https://your-team-host/api --token <api-key>
teamai source list
teamai source remove-http
```

An install has one HTTP source. `add-http` is unavailable when the main repo is already `init --http`. The API is the [HTTP contract](./http-contract.md).
