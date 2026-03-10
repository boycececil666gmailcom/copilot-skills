---
name: profile-publish-tutorial
description: >
  Publish a new tutorial or passage article into the tiered tutorials system and
  auto-update the GitHub profile README.  Use when the user wants to add a tutorial,
  publish a passage, write a new article, or create a new markdown entry under any
  expertise tier.  Supports two modes: exact content (written as-is) and reference
  content (agent rewrites and restructures it first).
argument-hint: 'Title, tier/topic, and content or reference material for the tutorial'
user-invocable: true
---

# Publish Tutorial Skill

## When to Use
- User says "add a tutorial", "publish a passage", "create a new article", "write a new tutorial"
- User provides content and wants it filed under a tier/topic
- User provides rough notes or reference material and wants a polished article created

---

## Required Inputs

Always collect these before proceeding:

| Input | Example |
|-------|---------|
| **Title** | `"Rigging Basics in Blender"` |
| **Tier** | `tier1` / `tier2` / `tier3` |
| **Topic** | `blender`, `python`, `unity`, etc. (must match an existing topic folder) |
| **Mode** | *Exact* — use content verbatim · *Reference* — rewrite first |
| **Content** | The actual text or reference material |

---

## Mode A — Exact Content

User provides the finished text to store as-is.

```mermaid
flowchart TD
    A([User provides title + exact content]) --> B[Run publish_tutorial.py]
    B --> C[File created in tutorials/tierN/topic/]
    C --> D[update_readme.py runs automatically]
    D --> E[git add README.md tutorials/]
    E --> F["git commit -m 'docs: add [topic] tutorial'"]
    F --> G[git push]
    G --> H([Live on GitHub profile])
```

**Run:**
```powershell
python "d:\boycececil666gmailcom\boycececil666gmailcom\.copilot\skills\publish-tutorial\scripts\publish_tutorial.py" `
  --title "Article Title" `
  --tier tier1 `
  --topic blender `
  --content "# Article Title\n\nYour content here."
```

Or pass a file:
```powershell
python "d:\boycececil666gmailcom\boycececil666gmailcom\.copilot\skills\publish-tutorial\scripts\publish_tutorial.py" `
  --title "Article Title" `
  --tier tier1 `
  --topic blender `
  --content-file path\to\draft.md
```

---

## Mode B — Reference Content (Agent Rewrite)

User provides rough notes, a passage from elsewhere, or unstructured reference material.
The agent rewrites it into a clean, well-structured tutorial before saving.

```mermaid
flowchart TD
    A([User provides title + reference material]) --> B[Agent rewrites content]
    B --> C{Quality check}
    C -- Good --> D[Run publish_tutorial.py with rewritten content]
    C -- Needs work --> B
    D --> E[File created in tutorials/tierN/topic/]
    E --> F[update_readme.py runs automatically]
    F --> G[git add README.md tutorials/]
    G --> H["git commit -m 'docs: add [topic] tutorial'"]
    H --> I[git push]
    I --> J([Live on GitHub profile])
```

### Rewrite Rules

When rewriting reference content, follow these principles:

1. **Keep all meaning** — do not add, remove, or contradict facts from the source
2. **Improve structure** — use clear `##` / `###` headings to organise sections logically
3. **Improve flow** — write in clear, direct sentences; remove filler words
4. **Use code blocks** where appropriate (` ```language ... ``` `)
5. **Add a brief intro paragraph** summarising what the article covers
6. **Start the file with** `# Article Title` (the exact title the user provided)
7. Do NOT invent examples or facts not present in the reference

### After rewriting

Save the rewritten content to a temporary string, call the script with `--content`, then push:

```powershell
python "d:\boycececil666gmailcom\boycececil666gmailcom\.copilot\skills\publish-tutorial\scripts\publish_tutorial.py" `
  --title "Article Title" `
  --tier tier1 `
  --topic blender `
  --content "<rewritten markdown>"

git add README.md tutorials/
git commit -m "docs: add [topic] tutorial — [title]"
git push
```

---

## Topic → Tier Reference

| Topic folder | Tier |
|---|---|
| `python`, `csharp`, `unity`, `blender`, `photoshop`, `git`, `mcp`, `stable-diffusion` | `tier1` |
| `unreal-engine`, `javascript`, `ros2`, `html`, `godot` | `tier2` |
| `ethereum`, `raspberry-pi`, `stm32` | `tier3` |

---

## Script Location

`d:\boycececil666gmailcom\boycececil666gmailcom\.copilot\skills\publish-tutorial\scripts\publish_tutorial.py`
