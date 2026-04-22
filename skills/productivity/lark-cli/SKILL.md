---
name: lark-cli
description: Install and configure the official Lark/Feishu CLI (@larksuite/cli) — 200+ commands covering Docs, Calendar, Messenger, Drive, Sheets, Mail, Tasks, etc. Designed for humans and AI Agents.
tags: [feishu, lark, cli, api, integration]
---

# Lark CLI — Official Feishu/Lark CLI Tool

GitHub: https://github.com/larksuite/cli
NPM: `@larksuite/cli` (NOT `@lark/cli`)

## Quick Install

> **Package name:** `@larksuite/cli` (NOT `@lark/cli` — that's a different, unrelated package)

### Method 1: npm (may fail in restricted environments)
```bash
npm install -g @larksuite/cli
```

**Common npm failures:**
- `EACCES` permission denied → use `--prefix ~/.local` or binary method
- `spawnSync curl ENOENT` → the post-install script needs `curl`; use binary method
- `404 Not Found` → you typed `@lark/cli` instead of `@larksuite/cli`

### Method 2: Binary download (preferred when npm fails)
```bash
# Detect architecture
ARCH=$(uname -m)  # aarch64 → arm64, x86_64 → amd64
OS=$(uname -s | tr '[:upper:]' '[:lower:]')

# Download from GitHub releases (check latest version)
VERSION="1.0.12"  # update as needed
wget -O /tmp/lark-cli.tar.gz "https://github.com/larksuite/cli/releases/download/v${VERSION}/lark-cli-${VERSION}-${OS}-${ARCH}.tar.gz"

# Extract and install
tar xzf /tmp/lark-cli.tar.gz -C /tmp/
mkdir -p ~/.local/bin
cp /tmp/lark-cli ~/.local/bin/lark-cli
chmod +x ~/.local/bin/lark-cli

# Verify
~/.local/bin/lark-cli --version
```

**Pitfall:** If `~/.local/bin/lark-cli` already exists as a dangling symlink, remove it first with `rm -f` before copying.

## Configuration

```bash
# Non-interactive setup with app credentials
echo "YOUR_APP_SECRET" | lark-cli config init --app-id "YOUR_APP_ID" --app-secret-stdin --brand feishu

# Check status
lark-cli auth status
```

## User Login (OAuth Device Flow)

For full access (read docs, calendar, etc. on behalf of user):

```bash
# Initiate login — returns a verification URL for the user
lark-cli auth login --recommend --no-wait

# The output contains verification_url and device_code
# User opens the URL and authorizes in browser

# Complete login (after user authorizes)
lark-cli auth login --device-code <DEVICE_CODE>
```

**Tip for AI Agents:** The `--no-wait` flag returns immediately. Send the `verification_url` to the user. After they authorize, run the `--device-code` command to complete.

## Key Commands

```bash
lark-cli auth status          # Check auth state
lark-cli auth scopes          # List available API scopes
lark-cli auth login --recommend --no-wait  # Start OAuth login

# Example commands (Shortcuts use + prefix)
lark-cli calendar +agenda     # View calendar
lark-cli docs +list           # List docs
```

## Available Domains
approval, attendance, base, calendar, contact, docs, drive, event, im, mail, minutes, sheets, slides, task, vc, wiki

## AI Agent Skills (23 skills)
```bash
npx skills add larksuite/cli -y -g  # Install 23 AI Agent Skills
```

**Skills timeout pitfall:** If `npx skills add` times out (common on slow connections), the skills are cloned to `/tmp/skills-XXXX/skills/` but not installed. Fix manually:
```bash
mkdir -p ~/.agents/skills
cp -r /tmp/skills-*/skills/* ~/.agents/skills/
```

Installed skills: `lark-approval`, `lark-attendance`, `lark-base`, `lark-calendar`, `lark-contact`, `lark-doc`, `lark-drive`, `lark-event`, `lark-im`, `lark-mail`, `lark-minutes`, `lark-openapi-explorer`, `lark-shared`, `lark-sheets`, `lark-skill-maker`, `lark-slides`, `lark-task`, `lark-vc`, `lark-whiteboard`, `lark-whiteboard-cli`, `lark-wiki`, `lark-workflow-meeting-summary`, `lark-workflow-standup-report`

## MCP Server (alternative integration)
There's also an official MCP server: `@larksuiteoapi/lark-mcp` — can be used with Hermes native MCP client for direct tool integration.

### Creating Feishu Documents

```bash
# STDIN method (preferred — avoids absolute-path validation errors)
cat /tmp/my-doc.md | lark-cli docs +create --title "Title" --markdown - --as user

# Or cd to the file's directory and use a relative path
cd /some/dir && lark-cli docs +create --title "Title" --markdown ./file.md --as user
```

**⚠️ Pitfall:** `--markdown @/absolute/path` is rejected. The flag only accepts `-` (stdin) or a path relative to `$CWD`.

Response: `{"ok": true, "data": {"doc_id": "BWU...", "doc_url": "https://www.feishu.cn/docx/BWU..."}}`

Other operations:
```bash
lark-cli docs +fetch <doc_id>                                         # Read
cat /tmp/updated.md | lark-cli docs +update <doc_id> --markdown - --as user  # Update
lark-cli docs +search "keyword"                                       # Search
```

### Sending Messages

```bash
lark-cli im +messages-send --chat-id oc_xxx --text "Hello" --as bot
lark-cli im +chat-search --json  # Find chat IDs
```

## Troubleshooting
- **npm install fails with EACCES**: Use `--prefix ~/.local` or download binary directly
- **npm install fails (no curl)**: The post-install script needs curl; use binary download method instead
- **Package not found**: Package is `@larksuite/cli`, not `@lark/cli`
- **Dangling symlink error**: `rm -f ~/.local/bin/lark-cli` before copying
- **Skills install wrong repo**: Use `larksuite/cli`, not `lark/cli`
- **lark-cli not found after install**: Add to PATH: `export PATH="$HOME/.local/bin:$PATH"` and persist in `~/.bashrc`
- **Python shutil.copy2 FileNotFoundError**: In sandboxed environments, `os.path.exists()` may return True but `shutil.copy2` still fails. Use shell commands (`cp`, `chmod`) instead of Python for file operations.
- **GitHub API releases**: If GitHub web times out, use the API directly: `https://api.github.com/repos/larksuite/cli/releases/latest` to get download URLs
