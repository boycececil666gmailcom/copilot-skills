---
name: godot-push
description: "Export Godot project to dist folder, copy StreamingAsset, and commit/push changes. Use when: godot export push, deploy godot project, build and commit godot."
argument-hint: "Path to Godot project (optional, defaults to current workspace)"
---

# godot-push

## When to Use
- Exporting and deploying a Godot project
- Building Godot game and pushing to git
- Automating Godot release process

## Procedure
1. Run headless Godot export to dist folder using the "Windows Desktop" preset.
2. Copy the StreamingAsset folder to the dist folder.
3. Invoke the git-commit-push skill to commit and push the changes.

## Notes
- Assumes Godot is installed and available in PATH.
- Uses the export preset "Windows Desktop".
- StreamingAsset is excluded from export, so copied manually.
