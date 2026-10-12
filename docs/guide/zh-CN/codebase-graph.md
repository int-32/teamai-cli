# 代码知识图谱

> 本文是 [TeamAI CLI 使用指南](../../usage-guide.zh-CN.md) 的一部分。

---

`teamai import` 把源码仓库写成团队仓库里的 `teamwiki/`，`teamai recall` 会一起搜。

```bash
teamai import --dir /path/to/project
teamai import --from-repo https://github.com/org/repo
teamai import --from-org myorg
teamai import --from-repo-list repos.yaml
teamai import --from-mr https://github.com/org/repo/pull/123
teamai import --from-repo https://github.com/org/repo --incremental
teamai import --from-repo https://github.com/org/repo --skip-enrich
```

已经提过一次之后，用 `teamai codebase` 刷新和检查：

```bash
teamai codebase --extract /path/to/repo --project my-service
teamai codebase --extract /path/to/repo --project my-service --incremental
teamai codebase --deep-enrich --project my-service --output /path/to/repo
teamai codebase --lint --output /path/to/repo
```

`--output` 指向仓库根目录。`--project` 的值会成为 `teamwiki/evidence/code/<slug>/`。

recall 结果里的 `Source freshness:`：`current` 是内容和上次扫描一致，`stale` 是变了，`missing` 是文件不在了，`unknown` 是对不上。`current` 也不代表文档本身正确。

只让一部分人看到某个代码库时：

```yaml
# manifest/projects.yaml
projects:
  - id: svc-a
    resources:
      wiki: [svc-a]
```

`svc-a` 是提取时用的 slug，不一定等于项目 id。
