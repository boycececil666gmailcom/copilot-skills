---
name: make-skill
description: "Create a new VS Code Copilot skill file with the correct category prefix. Use when: creating a skill, adding a skill, make skill, new skill, scaffold skill. Applies a category prefix (python-, git-, web-, data-, test-, infra-, ai-, util-, cli-, db-, profile-) based on the skill's purpose."
argument-hint: "Describe the skill you want to create"
---

# make-skill

Create a new Copilot skill (`SKILL.md`) with the correct category prefix applied to the folder and `name` field.

## Category Prefixes

See [./references/categories.md](./references/categories.md) for the full prefix table.

| Prefix | Domain |
|--------|--------|
| `python-` | Python tooling, packages, analysis |
| `git-` | Git workflows, branching, history |
| `web-` | Frontend, HTML/CSS/JS, browsers |
| `data-` | Data pipelines, ETL, analytics |
| `test-` | Testing, coverage, mocking |
| `infra-` | CI/CD, Docker, cloud, IaC |
| `ai-` | LLMs, embeddings, ML models |
| `util-` | General utilities, file ops |
| `cli-` | Command-line tools and scripts |
| `db-` | Databases, queries, migrations |
| `profile-` | GitHub profile content, tutorials, README publishing |

## Procedure

1. **Get the skill description** from the user's message or `argument-hint`.

2. **Determine the category prefix** by matching the skill's domain to the table above.
   - If ambiguous, ask the user to pick a prefix from [./references/categories.md](./references/categories.md).

3. **Compose the full skill name**: `<prefix><short-name>` (e.g., `git-conventional-commits`, `python-docstring`).

4. **Determine the scope**:
   - Project-specific → `.github/skills/<skill-name>/SKILL.md` in the workspace root.
   - Personal/cross-workspace → `~/.copilot/skills/<skill-name>/SKILL.md`.
   - Default to project scope unless the user says otherwise.

5. **Create the skill folder and `SKILL.md`** with this template:

```markdown
---
name: <full-skill-name>
description: "<One sentence. Start with action verbs. Include 3-5 trigger phrases. Max 1024 chars.>"
argument-hint: "<Short hint for slash-command invocation>"
---

# <Title>

## When to Use
- <trigger scenario 1>
- <trigger scenario 2>

## Procedure
1. <Step 1>
2. <Step 2>
3. <Step 3>

## Notes
- <Any caveats or tips>
```

6. **Validate**:
   - `name` field matches the folder name exactly.
   - `description` contains trigger keywords so the agent can discover it.
   - Frontmatter uses spaces (not tabs), and string values with colons are quoted.

7. **Confirm** the file path and name to the user.
