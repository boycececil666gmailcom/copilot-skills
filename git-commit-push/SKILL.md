---
name: git-commit-push
description: "Stage, commit, and push the currently opened folder to GitHub. Use when: push changes, commit and push, git push, save to github, sync to remote. Commit message is auto-generated as a timestamp plus an AI-written summary of what changed."
argument-hint: "Optional extra context for the commit message"
---

# git-push-folder

Stage all changes in the current workspace folder, generate a commit message from the timestamp and a concise AI summary of the diff, then push to the remote.

## Procedure

### 1. Refresh PATH (Windows)

```powershell
$env:PATH = [System.Environment]::GetEnvironmentVariable("PATH","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("PATH","User")
```

### 2. Verify the repo state

```powershell
git -C <workspace-root> status
git -C <workspace-root> remote -v
```

- If no remote exists, follow the `git-init-push` skill first.
- If there are no changes (`nothing to commit`), stop and report "nothing to push".

### 3. Inspect the diff to write the commit message

Run:

```powershell
git -C <workspace-root> diff --staged
git -C <workspace-root> diff        # unstaged
git -C <workspace-root> status --short
```

From the diff output, write a short commit body covering:
- Which files changed and why (inferred from content)
- Any new features, fixes, or deletions worth noting

### 4. Compose the commit message

Format:

```
[YYYY-MM-DD HH:MM UTC] <one-line summary>

<optional 2-3 line body if changes are non-trivial>
```

Example:
```
[2026-03-10 11:42 UTC] Add gh-based auth, remove token credentials

- Replaced Basic auth header with gh credential helper
- Removed credentials section from config template
- _ensure_cloned now uses gh repo clone
```

### 5. Stage, commit, and push

```powershell
git -C <workspace-root> add .
git -C <workspace-root> commit -m "<composed message>"
git -c "credential.helper=!gh auth git-credential" -C <workspace-root> push
```

### 6. Confirm

Report:
- The commit hash and message
- The remote and branch pushed to
- Any warnings (e.g. rejected push, diverged branch)

## Decision Points

| Situation | Action |
|-----------|--------|
| Nothing staged/unstaged | Report "nothing to push", stop |
| No remote configured | Run `git-init-push` skill first |
| Push rejected (non-fast-forward) | Run `git pull --rebase` then retry push |
| Untracked secrets/tokens visible in diff | Warn the user before committing |
| `gh` not authenticated | Run `gh auth login` first |

## Notes

- Always run `git diff` before committing — never commit blindly.
- If the diff contains credentials or secrets, stop and warn the user.
- The timestamp is always UTC.
