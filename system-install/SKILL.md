---
name: system-install
description: "Install any program or package using the best available method. Use when: install app, install package, install tool, system install, download and install, get program. Tries winget → choco/scoop → npm/pip/cargo → GitHub search → official site HTTP download automatically."
argument-hint: "Name of the app or package to install (e.g. 'ffmpeg', 'node', 'docker')"
---

# cli-system-install

Install any program on Windows using the best available method, falling back through a chain of sources until successful. If all package managers fail, search GitHub and the official site, then HTTP-fetch the installer to `~/Downloads/new-app/`.

---

## Install Chain (in order)

| Step | Method | Example |
|------|--------|---------|
| 1 | **winget** | `winget install --id <id> --source winget` |
| 2 | **Chocolatey** (`choco`) | `choco install <name> -y` |
| 3 | **Scoop** | `scoop install <name>` |
| 4 | **npm** (for JS tools) | `npm install -g <name>` |
| 5 | **pip** (for Python tools) | `pip install <name>` |
| 6 | **cargo** (for Rust tools) | `cargo install <name>` |
| 7 | **GitHub search** | Search `github.com/<name>` releases |
| 8 | **Official site HTTP fetch** | Download installer to `~/Downloads/new-app/` |

---

## Procedure

### 0. Refresh PATH

```powershell
$env:PATH = [System.Environment]::GetEnvironmentVariable("PATH","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("PATH","User")
```

### 1. Try winget

```powershell
winget search <name>
```

- If a match is found, install it:
  ```powershell
  winget install --id <matched-id> --source winget --accept-package-agreements --accept-source-agreements
  ```
- If exit code is 0 → **done**.
- If no match or non-zero exit → continue to step 2.

### 2. Try Chocolatey

Check if `choco` is available:
```powershell
Get-Command choco -ErrorAction SilentlyContinue
```

If available:
```powershell
choco search <name>
choco install <name> -y
```
- If exit code is 0 → **done**.
- If `choco` is not installed and user confirms, install it first:
  ```powershell
  Set-ExecutionPolicy Bypass -Scope Process -Force
  [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
  Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
  ```
- If still no match → continue to step 3.

### 3. Try Scoop

Check if `scoop` is available:
```powershell
Get-Command scoop -ErrorAction SilentlyContinue
```

If available:
```powershell
scoop search <name>
scoop install <name>
```
- If exit code is 0 → **done**.
- If `scoop` is not installed and user confirms, install it:
  ```powershell
  Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
  Invoke-RestMethod get.scoop.sh | Invoke-Expression
  ```
- If no match → continue to step 4.

### 4. Try npm (JavaScript / Node tools)

Only if the package looks like a Node.js tool or user requests it:
```powershell
Get-Command npm -ErrorAction SilentlyContinue
npm install -g <name>
```
- If exit code is 0 → **done**.
- Otherwise → continue to step 5.

### 5. Try pip (Python tools)

Only if the package looks like a Python tool or user requests it:
```powershell
Get-Command pip -ErrorAction SilentlyContinue
pip install <name>
```
- If exit code is 0 → **done**.
- Otherwise → continue to step 6.

### 6. Try cargo (Rust tools)

Only if the package looks like a Rust crate or user requests it:
```powershell
Get-Command cargo -ErrorAction SilentlyContinue
cargo install <name>
```
- If exit code is 0 → **done**.
- Otherwise → continue to step 7.

### 7. Search GitHub for a Release

Use the GitHub search API to find a matching repo with releases:

```powershell
# Via gh CLI
gh search repos <name> --limit 5 --json fullName,description,url

# Then list latest release assets
gh release list -R <owner>/<repo> --limit 3
gh release view --repo <owner>/<repo> --json assets
```

- Identify the correct Windows asset (`.exe`, `.msi`, `.zip` with `.exe` inside).
- Download URL found → proceed to **step 8**.
- No suitable release found → continue to the official site search.

**Official site fallback** — before step 8, report to the user:
> "No package manager found `<name>`. Searching for the official download page..."

Construct the most likely official URL patterns:
- `https://<name>.io/`
- `https://<name>.dev/`
- `https://www.<name>.com/download`
- `https://github.com/<name>/<name>/releases`

If a URL can be confidently identified, proceed to step 8.  
If ambiguous, show the top candidates and ask the user to confirm the URL.

### 8. HTTP Fetch — Download to `~/Downloads/new-app/`

Ensure the download directory exists:
```powershell
$dest = "$env:USERPROFILE\Downloads\new-app"
New-Item -ItemType Directory -Force -Path $dest | Out-Null
```

Download the installer:
```powershell
$url = "<confirmed-download-url>"
$filename = Split-Path $url -Leaf
$outPath = Join-Path $dest $filename

Invoke-WebRequest -Uri $url -OutFile $outPath -UseBasicParsing
Write-Host "Downloaded to: $outPath"
```

After download:
- If `.msi`: `Start-Process msiexec.exe -ArgumentList "/i `"$outPath`" /qn" -Wait`
- If `.exe`: `Start-Process $outPath -Wait`
- If `.zip`: unzip and locate the `.exe`:
  ```powershell
  Expand-Archive -Path $outPath -DestinationPath "$dest\<name>" -Force
  ```
  Then report the extracted path to the user.

---

## Decision Table

| Situation | Action |
|-----------|--------|
| winget finds exact match | Install via winget and stop |
| winget finds multiple matches | Show list, ask user to confirm ID |
| choco/scoop not installed | Ask user before installing the manager |
| npm/pip/cargo — unclear fit | Skip unless tool looks language-specific |
| GitHub has a release asset | Download via `gh release download` |
| Official URL ambiguous | Show 2–3 candidates, ask user to pick |
| HTTP download succeeds | Open `~/Downloads/new-app/` and report path |
| All methods fail | Report failure with links found and ask user for direct URL |

---

## Notes

- Always run PATH refresh (step 0) before any command.
- Prefer `winget` and `scoop` for Windows-native apps; `choco` for legacy packages.
- Never silently install a package manager without user confirmation.
- Validate downloaded files are from official or trusted sources before executing them.
- `~/Downloads/new-app/` accumulates downloads — do not auto-delete previous ones.
