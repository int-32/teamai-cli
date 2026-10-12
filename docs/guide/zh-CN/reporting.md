# 使用统计上报

> 本文是 [TeamAI CLI 使用指南](../../usage-guide.zh-CN.md) 的一部分。

---

`teamai pull` 默认把会话和使用次数提交进团队仓库。不想要这些提交时：

```yaml
# teamai.yaml
usageReport: false
```

本机覆盖写在该 scope 的 `config.yaml`：

```yaml
usageReportEnabled: false
```

`teamai doctor` 会说明当前开没开、设置来自哪里。

早期版本可能把别的项目的 skill 记进了你的 `stats/<user>.yaml`。在单独的 clone 里删掉那一条：

```bash
git clone --branch teamai-reports --single-branch <team-repo-url> teamai-reports
cd teamai-reports
# 删掉 stats/<user>.yaml 里 skills: 下的那条
git commit -am "stats: remove <skill> reported from another project"
git push origin teamai-reports
```
