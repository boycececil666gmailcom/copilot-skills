---
name: godot-code
description: "Write or refactor Godot GDScript using world-server style conventions, region blocks, readable structure, and public API documentation. Use when: add godot code, refactor gdscript, organize script regions, document public api, world module docs."
argument-hint: "Describe the script to write or refactor"
---

# godot-code

## When to Use
- Adding a new GDScript file to the project
- Refactoring an existing script for clarity
- Documenting public API functions
- Creating or updating a module README

---

## Script Structure Convention

Every GDScript file must follow this region order. Omit regions that have no content.

```gdscript
extends <BaseClass>
# filename.gd — One-line summary of what this script does.
# Longer explanation if needed (who owns what, what is caller's responsibility).


#region Constants
# ...
#endregion


#region Signals
# ...
#endregion


#region Exported Fields
# ...
#endregion


#region Private Vars
# ...
#endregion


#region Lifecycle
func _ready() -> void:
    # ...
#endregion


#region Public API
# Doc comment for every public function.
func some_public_method() -> void:
    # ...
#endregion


#region Private API
func _some_private_method() -> void:
    # ...
#endregion


#region Private — <SubCategory>
# Use sub-category labels when a Private API section is large enough to split.
#endregion
```

---

## Rules

### Naming
- Private fields and methods start with `_` (e.g. `_map`, `_build_map()`).
- Exported fields are public and need no prefix.
- Constants are `UPPER_SNAKE_CASE` when truly constant; `_UPPER_SNAKE_CASE` when private.

### Doc Comments
Every **public** function (no `_` prefix) must have a `#` comment on the line(s) immediately above it:
```gdscript
# Returns the generated map as a 2-D Array[Array[Dictionary]].
# Call generate() first; returns empty dict if coordinates are out of bounds.
func get_cell(x: int, y: int) -> Dictionary:
```

Private helpers that are non-obvious should also have a short comment.

### Readability
- Align assignment operators when initializing a group of related variables.
- Keep functions short — if a function exceeds ~20 lines, split it.
- Prefer explicit types on variables where the type isn't obvious from the right-hand side.

### Region Labels for Large Sections
When a Private section covers more than one concern, add a dash-separated label:
```gdscript
#region Private — Noise Setup
#region Private — Map Construction
```

---

## Module README Convention

Every first-level subfolder that acts as a game module (e.g. `World/`, `UI/`, `Enemy/`) **must** contain a `README.md` with this structure:

```markdown
# <Module Name>

One paragraph describing what the module is responsible for and what it explicitly
does NOT do (boundary statement).

## Folder Structure

| Path | Role |
|------|------|
| `runtime/foo.gd` | Description |
| `test/bar.gd`    | Description |

## Quick-Start API

### `NodeName` (`path/to/script.gd`)

| Method / Property | Signature | Description |
|---|---|---|
| `method_name` | `(arg: Type) -> ReturnType` | What it does |

## Data Format

If the module reads external data (JSON, etc.), describe the schema here.

## Dependencies

List any autoloads, singletons, or other modules this module relies on.
```

### Procedure when writing/refactoring a module

1. Apply region structure to every `.gd` file in the module.
2. Add doc comments to all public functions.
3. Create or update the module `README.md`.
