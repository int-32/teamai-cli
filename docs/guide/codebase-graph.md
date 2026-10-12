# Codebase Knowledge Graph

> Part of the [TeamAI CLI Usage Guide](../usage-guide.md).

---

`teamai import` writes a source repo into the team repo's `teamwiki/`. `teamai recall` searches it too.

```bash
teamai import --dir /path/to/project
teamai import --from-repo https://github.com/org/repo
teamai import --from-org myorg
teamai import --from-repo-list repos.yaml
teamai import --from-mr https://github.com/org/repo/pull/123
teamai import --from-repo https://github.com/org/repo --incremental
teamai import --from-repo https://github.com/org/repo --skip-enrich
```

After the first import, refresh and check with `teamai codebase`:

```bash
teamai codebase --extract /path/to/repo --project my-service
teamai codebase --extract /path/to/repo --project my-service --incremental
teamai codebase --deep-enrich --project my-service --output /path/to/repo
teamai codebase --lint --output /path/to/repo
```

`--output` is the repository root. `--project` becomes `teamwiki/evidence/code/<slug>/`.

A recall hit may add `Source freshness:`: `current` matches the last scan, `stale` changed, `missing` is gone, `unknown` could not be checked. `current` does not mean the written knowledge is correct.

Limit one codebase to some members:

```yaml
# manifest/projects.yaml
projects:
  - id: svc-a
    resources:
      wiki: [svc-a]
```

`svc-a` is the slug used at extract time. It does not have to match the project id.
