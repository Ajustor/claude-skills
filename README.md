# Development rules skills

Agent skills derived from the team's *Development Rules* document, one per work domain. Each folder under `skills/` holds a `SKILL.md` following the [Agent Skills](https://agentskills.io) format, deployable with [pollen](https://github.com/groupbees/pollen).

| Skill | Domain | Rules |
|---|---|---|
| [clean-architecture](skills/clean-architecture/SKILL.md) | Layers, dependency rule, SOLID, folder structure | R-ARCH-01..06 |
| [design-patterns](skills/design-patterns/SKILL.md) | When and which pattern, anti-patterns, YAGNI | R-PAT-01..04 |
| [clean-code](skills/clean-code/SKILL.md) | Naming, functions, errors, tests | R-CC-01..24 |
| [performance](skills/performance/SKILL.md) | Measurement, DB, caching, network, front-end | R-PERF-01..15 |
| [code-review](skills/code-review/SKILL.md) | PR process, review procedure, comment style, PR checklist | R-REV-01..07 |
| [quality-verification](skills/quality-verification/SKILL.md) | CI/hooks tooling, periodic audits, technical debt | R-VER-01..10 |

Rule IDs are kept identical to the source document so findings can be traced back to it. When the rules document changes, update the matching skill.

## Install with pollen

In a project's `pollen.yaml` (or the user-level one, `pollen -g`):

```yaml
repos:
  - repo: <git URL of this repository>
    revision: <full commit SHA>  # vX.Y.Z
    paths:
      - path: skills
```

Then run `pollen update`. To pick only some domains, add `exclude: '^(performance|quality-verification)$'` under the path entry.

Check the skills locally before releasing:

```sh
pollen validate   # with a pollen.yaml using `repo: local` and `path: skills`
```
