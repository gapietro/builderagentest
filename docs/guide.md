# ServiceNow Fluent Development Guide

> A step-by-step runbook for setting up a ServiceNow scoped app,
> developing locally in Claude Code, and syncing code to your PDI and GitHub.
>
> **Audience:** ServiceNow internal developers — all skill levels.
> Experienced devs: headings are scannable, commands are copy-paste ready.

---

## Prerequisites

Before starting, ensure you have:
- A ServiceNow PDI (Personal Developer Instance)
- A GitHub account with SSH or HTTPS access configured
- macOS or Linux (Windows WSL2 also works)

---

## Phase 1: Installation & Verification

### 1.1 Check Existing Tools

Run each command to verify what's already installed:

```bash
snc --version        # ServiceNow CLI
node --version       # Node.js (required by snc) — need v18+
git --version        # Git
gh --version         # GitHub CLI
```

If any command returns "command not found", follow the install steps below.

### 1.2 Install / Upgrade ServiceNow CLI (snc)

> **Why snc?** The ServiceNow CLI is the official tool for creating,
> pulling, and pushing scoped apps between your local machine and a
> ServiceNow instance. Think of it like `git` but for ServiceNow app files.

**Download snc:**

1. Go to: [developer.servicenow.com](https://developer.servicenow.com)
2. Sign in with your ServiceNow credentials
3. Navigate to: **Tools → ServiceNow CLI**
4. Download the installer for your OS
5. Run the installer

**Verify installation:**

```bash
snc --version
```

Expected output (version may differ):
```
snc/x.x.x <os>-<arch> node-vxx.x.x
```

**If you have an old version**, re-run the installer from the link above —
it will upgrade in place.

### 1.3 Install Node.js (if needed)

snc requires Node.js v18 or higher.

```bash
node --version
```

If below v18 or not installed:
- **macOS (recommended):** `brew install node`
- **All platforms:** Download from [nodejs.org](https://nodejs.org)

### 1.4 Install GitHub CLI (if needed)

```bash
# macOS
brew install gh

# Verify
gh --version
```

**Authenticate gh with GitHub:**

```bash
gh auth login
```

Follow the prompts — choose GitHub.com → HTTPS → authenticate via browser.

### 1.5 Configure Git Identity

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@company.com"
```

Verify:

```bash
git config --global --list | grep user
```

---

## Phase 2: PDI Authentication

> You already have a PDI through the internal provisioning process.
> This phase connects your local snc CLI to that instance.

### 2.1 Configure snc

```bash
snc configure
```

You will be prompted for:
- **Instance URL:** `https://your-instance.service-now.com`
- **Username:** Your ServiceNow login
- **Password:** Your ServiceNow password (or OAuth token if configured)

> **Security note:** snc stores credentials locally in `~/.snc/`.
> Never copy these files into your project repo or commit them.
> The `.gitignore` in this project already excludes them.

### 2.2 Verify Connection

```bash
snc instance info
```

Expected output shows your instance name, version, and status.

If you see an authentication error:
- Double-check your instance URL (no trailing slash)
- Verify your credentials work by logging into the instance in a browser
- Re-run `snc configure`

---

## Phase 3: Create a Scoped App

> **What is a scoped app?** In ServiceNow, all custom development lives
> inside "scoped applications" — isolated namespaces that prevent conflicts
> with other apps and the platform itself. A scope looks like `x_acme_myapp`.
> If you already know this, skip ahead to 3.1.

### 3.1 Create the App

```bash
snc app create
```

You will be prompted for:
- **App name:** e.g., `My Training App`
- **Scope:** e.g., `x_snc_training` (auto-suggested, you can customize)
- **Version:** `1.0.0`

snc creates the app on your PDI and returns an app sys_id.

### 3.2 Pull the App to Local

```bash
snc app pull
```

This syncs the initial app structure from your PDI into a local `src/` directory.
The structure will look like:

```
src/
├── app_scope_x_snc_training/
│   ├── sys_app.xml
│   ├── sys_scope.xml
│   └── ...
```

> **Tip:** Always run `snc app pull` before starting work to make sure
> your local files are in sync with the instance.

### 3.3 Verify in the Instance

Log into your PDI and navigate to:
**App Engine Studio → My Apps** (or **System Applications → Studio**)

Your new app should appear in the list.

---

## Phase 4: Connect to Claude Code

> **What is Claude Code?** Claude Code (CC) is Anthropic's AI-powered CLI
> development environment. It understands your codebase and can help you
> write, review, and push ServiceNow app code — including talking directly
> to your instance via MCP (covered in Phase 7).

### 4.1 Open the Project in Claude Code

From your project root:

```bash
claude
```

Claude Code will start and index your project files.

### 4.2 Create a CLAUDE.md

A `CLAUDE.md` file gives Claude context about your project.
Create one at the project root:

```markdown
# My Training App — ServiceNow Scoped App

## Project Overview
ServiceNow scoped app (scope: x_snc_training) on PDI instance.
Primary development via ServiceNow CLI (snc) + Claude Code.

## Key Commands
- Pull from instance: `snc app pull`
- Push to instance: `snc app push`
- App source: `src/`

## Conventions
- All changes pushed to instance AND committed to git
- Never commit snc credentials or .snc/ directory
- Branch naming: feature/, fix/, chore/
```

### 4.3 MCP Settings (Placeholder — See Phase 7)

Create `.claude/settings.json` at your project root with this template:

```json
{
  "mcpServers": {
    "servicenow": {
      "command": "npx",
      "args": ["-y", "@servicenow/mcp-server"],
      "env": {
        "SN_INSTANCE_URL": "https://your-instance.service-now.com",
        "SN_USERNAME": "your-username",
        "SN_PASSWORD": "your-password"
      }
    }
  }
}
```

> **Important:** This template uses placeholder values and is safe to commit.
> When you fill in real credentials, save to `.claude/settings.local.json`
> instead (which is gitignored). Never commit real credentials.

---

## Phase 5: Development Loop

The core workflow: pull from instance → edit locally → push back.

### 5.1 Pull Before You Start

Always sync the latest from your instance before editing:

```bash
snc app pull
```

### 5.2 Edit Files Locally

Open files in `src/` using VS Code or Claude Code.
Common files you'll edit:
- `src/<scope>/sys_script_include_*.xml` — Script Includes
- `src/<scope>/sys_ui_page_*.xml` — UI Pages
- `src/<scope>/sys_rest_message_*.xml` — REST Messages

> **Tip:** Ask Claude Code to help — e.g., "Add a utility function to
> the IncidentUtils Script Include that formats incident numbers."

### 5.3 Push to Instance

```bash
snc app push
```

This uploads your local changes to the PDI.

Verify in the instance: open the record you changed in Studio or the
application navigator and confirm your edits are there.

### 5.4 Handling Conflicts

If snc reports a conflict (instance has changes you don't have locally):

```bash
snc app pull          # Get latest from instance first
# Resolve any file conflicts manually
snc app push          # Then push your changes
```

> **Rule of thumb:** Pull first, always. Especially if others share the instance.

---

## Phase 6: Git Workflow

### 6.1 Create a Feature Branch

Never commit directly to main:

```bash
git checkout -b feature/my-first-change
```

Branch naming:
| Prefix | When to use |
|--------|-------------|
| `feature/` | New functionality |
| `fix/` | Bug fixes |
| `chore/` | Config, cleanup |
| `docs/` | Documentation only |

### 6.2 Stage and Commit

After pushing to your instance and verifying it works:

```bash
git add src/
git commit -m "feat: add incident utility functions"
```

Keep commits small and focused. One logical change per commit.

### 6.3 Push Branch and Open PR

```bash
git push -u origin feature/my-first-change
gh pr create --title "Add incident utility functions" --body "Adds formatIncidentNumber utility to IncidentUtils Script Include."
```

### 6.4 Merge and Clean Up

After PR is approved:

```bash
gh pr merge --squash
git checkout main
git pull
git branch -d feature/my-first-change
```
