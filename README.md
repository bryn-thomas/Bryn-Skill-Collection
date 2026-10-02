# Bryn Skill Collection

Personal Claude Code skills.

| Skill | What it does |
|---|---|
| [`attribute-quality-build`](skills/attribute-quality-build/SKILL.md) | Builds the Attribute Quality Monitoring dbt models for one IDP attribute across the seven data-quality lenses. It follows the playbook in `airflow-verification/.../attribute_quality_monitoring/README.md`, with hard stop points for review. |

## Install

Link each skill into your user-level skills folder. Claude Code then loads it in every project:

```bash
git clone https://github.com/bryn-thomas/Bryn-Skill-Collection.git
mkdir -p ~/.claude/skills
ln -s "$PWD/Bryn-Skill-Collection/skills/attribute-quality-build" ~/.claude/skills/attribute-quality-build
```

Run `git pull` in the clone to pick up updates.
