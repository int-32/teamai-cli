# Usage reporting

> Part of the [TeamAI CLI Usage Guide](../usage-guide.md).

---

`teamai pull` commits session and usage counts into the team repo. Turn that off with:

```yaml
# teamai.yaml
usageReport: false
```

One machine overrides it in that scope's `config.yaml`:

```yaml
usageReportEnabled: false
```

`teamai doctor` says whether reporting is on and where that comes from.

An older release may have written another project's skill into your `stats/<user>.yaml`. Delete that entry from a separate clone:

```bash
git clone --branch teamai-reports --single-branch <team-repo-url> teamai-reports
cd teamai-reports
# delete the skill's entry under skills: in stats/<user>.yaml
git commit -am "stats: remove <skill> reported from another project"
git push origin teamai-reports
```
