# Contributing

Thank you for improving `originplot`.

## Scope

This repository contains a Codex skill, not a standalone Python package. Changes
should improve how agents design, explain, or generate Origin/OriginPro external
Python automation workflows.

Good contributions include:

- Clearer `originpro` automation patterns.
- Better FigureSpec schema fields or examples.
- More precise guidance for workbooks, matrices, graph pages, graph layers,
  templates, exports, and OPJU archives.
- Small example specs that demonstrate real plotting workflows.
- Documentation fixes that reduce ambiguity.

Avoid committing:

- Large generated figures.
- `.opju` project files.
- Raw experiment datasets.
- Machine-specific paths, credentials, or local Origin installation details.

## Skill Design Rules

- Keep `originplot-skill/SKILL.md` focused on essential agent instructions.
- Move long schemas, examples, and detailed reference material into separate
  files linked from `SKILL.md`.
- Prefer concise examples over broad explanations.
- Preserve the external Python + OriginPro separation:

```text
computation outside Origin
data normalization in Python
visual style mostly in Origin templates
automation and batching in Python
final rendering in OriginPro
```

## Validation

Before opening a pull request, check:

```bash
python -m pip install pyyaml
python scripts/validate.py
```

The validation script checks the skill frontmatter, required repository files,
and YAML examples.

## Pull Requests

Please include:

- What changed.
- Why the change helps users or agents.
- Any Origin/OriginPro version assumptions.
- Any validation performed.
