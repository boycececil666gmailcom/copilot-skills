---
name: git-init
description: "Initialize a local folder as a git repo and push it to GitHub using the gh CLI. Use when: new repo, init and push, push to github, create github repo, first commit, git init push, bootstrap repo. Handles Windows PATH refresh, safe.directory, git identity, and gh credential auth automatically."
argument-hint: "Path to the folder to push (defaults to current workspace)"
---

# git-init-push

Initialize a local directory as a git repository, create a GitHub remote via `gh`, and push — handling all common Windows friction points automatically.

---

## Repo Naming & Folder Placement Convention

Every repo **must** follow the tiered naming convention before being initialized:

### Format

```
t{tier}-{tech-slug}-{project-name}
```

- **tier** — numeric tier matching the owner's profile expertise level (1, 2, or 3)
- **tech-slug** — lowercase hyphenated identifier for the primary technology (see table below)
- **project-name** — short kebab-case description of the project

### Examples

```
t1-python-daemon-git
t1-unity-hrsim
t2-js-hell-love-tcb
t2-godot-project-wildcard
```

### Tier → Tech Slug Reference

| Tier | Tech slugs |
|------|-----------|
| **t1** — Primary Focus | `python`, `csharp`, `unity`, `blender`, `godot`, `mcp`, `stable-diffusion` |
| **t2** — Building Experience | `js`, `unreal`, `ros`, `html` |
| **t3** — Exploring | `ethereum`, `raspberry-pi`, `stm32` |

### Folder Location

All repos live directly under `D:\boycececil666gmailcom\`:

```
D:\boycececil666gmailcom\t1-python-daemon-git\
D:\boycececil666gmailcom\t2-js-hell-love-tcb\
```

### Pre-Init Checklist

Before running `git init`, **verify**:

1. The folder name matches `t{tier}-{tech-slug}-{project-name}` exactly.
2. The folder is located at `D:\boycececil666gmailcom\{repo-name}\`.
3. The GitHub repo name will be identical to the folder name (no rename after push).

If the user provides a project name that doesn't follow this pattern, **prompt them to confirm the correct tier and tech slug** before proceeding.

---

## Prerequisites

- `git` installed (`winget install --id Git.Git -e`)
- `gh` installed (`winget install --id GitHub.cli -e`)
- `gh` authenticated (`gh auth login`)

If any prerequisite is missing, install it before continuing.

## Procedure

### 1. Refresh PATH (Windows)

Newly installed tools are not on PATH until the terminal is restarted. Refresh in-session:

```powershell
$env:PATH = [System.Environment]::GetEnvironmentVariable("PATH","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("PATH","User")
```

Verify: `git --version` and `gh --version` both print versions.

### 2. Authenticate gh (once per machine)

```powershell
gh auth login
```

- Host: **GitHub.com**
- Protocol: **HTTPS**
- Authenticate Git: **Yes**
- Method: **Login with a web browser** → copy the code → paste in browser → authorize

Verify: `gh auth status` shows `✓ Logged in`.

### 3. Configure git identity (once per machine)

```powershell
git config --global user.email "you@example.com"
git config --global user.name "YourName"
```

### 4. Mark directory as safe (if owner mismatch on Windows)

If git reports "dubious ownership":

```powershell
git config --global --add safe.directory D:/path/to/repo
```

### 5. Initialize, stage, and commit

```powershell
cd <repo-path>
git init
git add .
git commit -m "Initial commit"
```

### 6. Create GitHub remote and push

**If the repo does NOT exist on GitHub yet:**

```powershell
gh repo create <repo-name> --public --source=. --remote=origin --push
```

Use `--private` instead of `--public` if desired.

**If the repo already exists on GitHub:**

```powershell
git remote set-url origin https://github.com/<username>/<repo-name>.git
git -c "credential.helper=!gh auth git-credential" push -u origin master
```

## Decision Points

| Situation | Action |
|-----------|--------|
| `git`/`gh` not found | Install via winget, then refresh PATH |
| "dubious ownership" error | Run `git config --global --add safe.directory <path>` |
| "Author identity unknown" | Run `git config --global user.email/name` |
| Repo already exists on GitHub | Use `git remote set-url` + push with gh credential helper |
| Repo does not exist yet | Use `gh repo create --source=. --push` |
| Auth fails on push | Use `-c "credential.helper=!gh auth git-credential"` |

## Completion Check

- [ ] `git log --oneline` shows the initial commit
- [ ] `git remote -v` shows the correct GitHub URL
- [ ] `gh repo view` opens the repo page successfully
