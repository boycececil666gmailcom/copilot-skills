---
name: profile-update-projects
description: >
  Scan the currently opened repository, verify it is pushed to a GitHub remote,
  and add or update its entry in the Projects section of the GitHub profile README.
  Use when: update profile projects, add project to profile, sync repo to profile,
  add repo to readme, register project in profile, profile projects sync.
argument-hint: "Optionally provide a custom description or override the tier (t1/t2/t3)"
user-invocable: true
---

# profile-update-projects

Scan the opened workspace repository, check that it has a GitHub remote, gather
its metadata, and upsert a table row in the `## Projects` section of the profile
README at `d:\boycececil666gmailcom\boycececil666gmailcom\README.md`.

---

## When to Use

- User says "update profile projects", "add this repo to my profile", "sync project to profile"
- User has just pushed a new repo and wants it listed on the profile
- User wants to refresh a project's description / tech stack badges in the profile

---

## Profile README Location

```
d:\boycececil666gmailcom\boycececil666gmailcom\README.md
```

Always make changes in that file, then commit and push from that directory.

---

## Step 1 — Identify the Opened Repository

Get the workspace root from VS Code context (the folder currently open in the editor).
Run:

```powershell
cd <workspace-root>
git remote get-url origin
```

- If the command **fails** (no remote configured) → abort and tell the user to push the repo first (use the `git-init` skill if needed).
- If it **succeeds** → extract the GitHub URL.

Parse `owner` and `repo-name` from the URL.  
Supported URL formats:
- `https://github.com/owner/repo-name.git`
- `git@github.com:owner/repo-name.git`

---

## Step 2 — Fetch Repo Metadata via gh CLI

```powershell
gh repo view owner/repo-name --json name,description,languages,url
```

Extract:
| Field | Usage |
|-------|-------|
| `name` | Repo slug for the table link |
| `description` | Row description (truncate to ~80 chars if long) |
| `languages` | Build tech-stack badge string |
| `url` | Used as the link target |

If `description` is empty, generate a short one from the repo name (un-slug it).

---

## Step 3 — Determine the Tier

Read the **repo name prefix**:

| Prefix | Tier Section |
|--------|-------------|
| `t1-`  | `### Tier 1 — Primary Focus` |
| `t2-`  | `### Tier 2 — Building Experience` |
| `t3-`  | `### Tier 3 — Exploring` |
| none   | Ask the user which tier, or default to Tier 2 |

---

## Step 4 — Build Tech-Stack Badges

Use this reference to convert language names returned by `gh repo view` into shield badges:

```
Python      → ![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
C#          → ![C#](https://img.shields.io/badge/C%23-239120?style=flat-square&logo=csharp&logoColor=white)
JavaScript  → ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black)
TypeScript  → ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
Rust        → ![Rust](https://img.shields.io/badge/Rust-CE422B?style=flat-square&logo=rust&logoColor=white)
Go          → ![Go](https://img.shields.io/badge/Go-00ADD8?style=flat-square&logo=go&logoColor=white)
C++         → ![C++](https://img.shields.io/badge/C%2B%2B-00599C?style=flat-square&logo=cplusplus&logoColor=white)
C           → ![C](https://img.shields.io/badge/C-A8B9CC?style=flat-square&logo=c&logoColor=black)
Java        → ![Java](https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white)
Kotlin      → ![Kotlin](https://img.shields.io/badge/Kotlin-7F52FF?style=flat-square&logo=kotlin&logoColor=white)
Swift       → ![Swift](https://img.shields.io/badge/Swift-F05138?style=flat-square&logo=swift&logoColor=white)
Shell       → ![Shell](https://img.shields.io/badge/Shell-4EAA25?style=flat-square&logo=gnubash&logoColor=white)
PowerShell  → ![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?style=flat-square&logo=powershell&logoColor=white)
HTML        → ![HTML](https://img.shields.io/badge/HTML-E34F26?style=flat-square&logo=html5&logoColor=white)
CSS         → ![CSS](https://img.shields.io/badge/CSS-1572B6?style=flat-square&logo=css3&logoColor=white)
GDScript    → ![Godot](https://img.shields.io/badge/Godot-478CBF?style=flat-square&logo=godotengine&logoColor=white)
```

For **well-known frameworks / engines** inferred from the repo name or description:

```
unity   in name → ![Unity](https://img.shields.io/badge/Unity-FFFFFF?style=flat-square&logo=unity&logoColor=black)
blender in name → ![Blender](https://img.shields.io/badge/Blender-E87D0D?style=flat-square&logo=blender&logoColor=white)
unreal  in name → ![Unreal Engine](https://img.shields.io/badge/Unreal-0E1128?style=flat-square&logo=unrealengine&logoColor=white)
godot   in name → ![Godot](https://img.shields.io/badge/Godot-478CBF?style=flat-square&logo=godotengine&logoColor=white)
```

Include up to **3 badges** (most prominent languages/engines first).  
If no language info is available, omit the Tech Stack cell value (leave blank).

---

## Step 5 — Choose an Emoji

Pick one emoji that fits the repo's primary domain:

| Domain | Emoji |
|--------|-------|
| Python script / tool | 🌾 |
| Game (Unity / Godot / Unreal) | 🎮 |
| Web / JS | 🏙 |
| C# / .NET desktop | 🏔 |
| Blender / 3D | 🎨 |
| CLI / daemon / automation | 🔧 |
| Security / encryption | 🔐 |
| Data / ML / AI | 🤖 |
| Embedded / hardware | 🔌 |
| Other / misc | 📦 |

---

## Step 6 — Build the Table Row

Format:

```markdown
| <emoji> [<repo-name>](<github-url>) | <description> | <badge1> <badge2> | [▶ Demo](<github-url>) |
```

If there is a live demo URL other than the repo itself, use it in the Demo cell.  
Otherwise use: `▶ *coming soon*` (no link).

Example output row:
```markdown
| 🔧 [t1-python-daemon-git](https://github.com/boycececil666gmailcom/t1-python-daemon-git) | CLI daemon that auto-clones all GitHub repos locally and syncs them on a schedule | ![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white) | ▶ *coming soon* |
```

---

## Step 7 — Check for Existing Entry

Search the profile README for the repo name:

```powershell
Select-String -Path "d:\boycececil666gmailcom\boycececil666gmailcom\README.md" -Pattern "<repo-name>"
```

- **Found** → Replace the entire matching `|` row with the newly built row (update in place).
- **Not found** → Append the new row at the **end** of the correct tier table, before the next `###` or `---` boundary.

---

## Step 8 — Write, Commit, and Push

1. Apply the edit to the README (use file editing tools — do not use `echo` redirects).
2. From the profile repo directory:

```powershell
cd d:\boycececil666gmailcom\boycececil666gmailcom
git add README.md
git commit -m "profile: add/update project <repo-name>"
git push
```

---

## Decision Table

| Situation | Action |
|-----------|--------|
| No git remote found | Abort — ask user to push first (suggest `git-init` skill) |
| Repo is private | Proceed normally; note it will show on profile but the repo may not be publicly visible |
| Repo has no description on GitHub | Infer description from repo name |
| Repo name has no tier prefix | Ask user for tier, default to Tier 2 if no response |
| Row already exists in README | Overwrite the row in place |
| Row is new | Append to the bottom of the matching tier table |
| Push fails | Check `gh auth status`; retry with `git -c "credential.helper=!gh auth git-credential" push` |

---

## Completion Check

- [ ] `Select-String -Path README.md -Pattern "<repo-name>"` returns exactly one match
- [ ] The row shows the correct description, badges, and link
- [ ] `git log --oneline -1` in the profile repo shows the new commit
- [ ] `git status` is clean (nothing left to commit)
