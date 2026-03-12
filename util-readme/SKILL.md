---
name: util-readme
description: "Write a concise README for the opened folder or a named module. Use when: write readme, generate readme, document module, add readme, create readme, document project."
argument-hint: "Optional: path or module name to document (defaults to the open workspace folder)"
---

# util-readme

Write a minimal, elegant README for a module or folder.

## When to Use
- User says "write readme", "generate readme", "document this module", "add a readme"
- Targeting either the open workspace folder or a specific sub-module path

## Procedure

1. **Identify the target.**
   - If the user named a path or module, use that.
   - Otherwise use the current workspace folder.

2. **Explore the target** using file search and code reading tools to collect:
   - What the module does (its single responsibility)
   - Public API: exported functions, classes, signals, or endpoints
   - Dependencies: imports, `@export` nodes, external libs, config files
   - A realistic minimal call / usage pattern

3. **Write the README** following the template below. Keep it short — ruthlessly cut anything that isn't essential.

4. **Place the file** at `<target>/README.md`.
   - If one already exists, read it first and update in-place rather than overwriting blindly.

## Output Template

```markdown
# <module-name>

<One or two sentences. What it does and why it exists. No fluff.>

## API

| Name | Description |
|------|-------------|
| `symbol()` | What it returns / does |

## Dependencies

- `DependencyName` — why it's needed

## Usage

\`\`\`gdscript
# Minimal working example
\`\`\`
```

## Format Rules
- No "Overview", "Introduction", or filler sections
- API table only lists **public** symbols (no `_` prefixed internals)
- Dependencies lists only things the module **directly** requires
- Usage example must be runnable with no extra setup
- Prefer a single code block over prose explanation
- Keep the whole file under 60 lines

## Notes
- For GDScript modules: public API = functions without `_` prefix + `@export` vars + signals
- For JSON config files: describe the schema shape, not the values
- If the module is a thin wrapper, say so explicitly in the summary line
