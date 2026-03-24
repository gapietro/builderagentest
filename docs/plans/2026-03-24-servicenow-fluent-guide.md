# ServiceNow Fluent Development Guide — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a complete step-by-step training guide for setting up a ServiceNow scoped app using snc CLI, developing locally in Claude Code, and pushing code to both the instance and GitHub.

**Architecture:** Static documentation project — a primary `docs/guide.md` runbook converted to `docs/guide.html`, plus scaffolded repo structure that mirrors exactly what a developer would have after completing the guide. The repo itself IS the example project from the guide.

**Tech Stack:** ServiceNow CLI (snc), GitHub CLI (gh), Git, Claude Code (cc), ServiceNow MCP server, Node.js, Markdown, HTML

---

## Task 1: Initialize Repo & Git

**Files:**
- Create: `.gitignore`
- Create: `README.md`

**Step 1: Initialize git**

```bash
cd /Users/greg.pietro/projects/builderagentest
git init
git branch -M main
```

Expected: `Initialized empty Git repository`

**Step 2: Create `.gitignore`**

```gitignore
# ServiceNow CLI
.snc/
snc-config.json
*.snc-credentials

# Node
node_modules/
npm-debug.log

# OS
.DS_Store
Thumbs.db

# Secrets / credentials — NEVER commit these
.env
*.env.local
instance-credentials.json
```

**Step 3: Create `README.md`**

```markdown
# ServiceNow Fluent Development Guide

Training material for setting up a ServiceNow scoped app using the
ServiceNow CLI (snc), developing locally in Claude Code, and syncing
code to your PDI and GitHub.

## Quick Start

See [docs/guide.md](docs/guide.md) for the full step-by-step runbook.

## What This Covers

1. Prerequisites & Installation (snc, node, git, gh)
2. PDI Authentication
3. Create a Scoped App
4. Connect to Claude Code
5. Development Loop (pull → code → push)
6. Git Workflow
7. MCP Setup (Claude ↔ ServiceNow instance)
```

**Step 4: Initial commit**

```bash
git add .gitignore README.md
git commit -m "chore: initialize repo with gitignore and README"
```

---

## Task 2: Create GitHub Repo & Push

**Step 1: Create remote repo**

```bash
gh repo create builderagentest \
  --public \
  --description "ServiceNow Fluent Development Guide — training material" \
  --source=. \
  --remote=origin \
  --push
```

Expected: GitHub URL printed, branch pushed.

**Step 2: Verify**

```bash
gh repo view --web
```

Expected: Repo opens in browser.

---

## Task 3: Write Guide — Phase 1 & 2 (Prerequisites + Auth)

**Files:**
- Create: `docs/guide.md`

**Step 1: Create docs/ and begin guide**

Write `docs/guide.md` with the following content for Phases 1 and 2:

```markdown
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
```

**Step 2: Commit**

```bash
git add docs/guide.md
git commit -m "docs: add Phase 1 (installation) and Phase 2 (PDI auth)"
git push
```

---

## Task 4: Write Guide — Phase 3 (Create Scoped App)

**Files:**
- Modify: `docs/guide.md`

**Step 1: Append Phase 3 to guide**

```markdown
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
```

**Step 2: Commit**

```bash
git add docs/guide.md
git commit -m "docs: add Phase 3 (create scoped app)"
git push
```

---

## Task 5: Write Guide — Phase 4 (Connect to Claude Code)

**Files:**
- Modify: `docs/guide.md`
- Create: `.claude/settings.json` (placeholder/template)

**Step 1: Append Phase 4 to guide**

```markdown
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

Create `.claude/settings.json`:

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

> **Important:** `.claude/settings.json` containing credentials should be
> in your `.gitignore`. The template above is safe to commit because it
> uses placeholder values. Replace with real values locally — never commit them.

Update `.gitignore` to add:
```
.claude/settings.local.json
```
```

**Step 2: Create `.claude/settings.json` template**

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

**Step 3: Commit**

```bash
git add docs/guide.md .claude/settings.json
git commit -m "docs: add Phase 4 (Claude Code setup) and MCP settings template"
git push
```

---

## Task 6: Write Guide — Phase 5 & 6 (Dev Loop + Git Workflow)

**Files:**
- Modify: `docs/guide.md`

**Step 1: Append Phases 5 and 6**

```markdown
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
```

**Step 2: Commit**

```bash
git add docs/guide.md
git commit -m "docs: add Phase 5 (dev loop) and Phase 6 (git workflow)"
git push
```

---

## Task 7: Write Guide — Phase 7 (MCP Setup)

**Files:**
- Modify: `docs/guide.md`

**Step 1: Append Phase 7**

```markdown
---

## Phase 7: MCP Setup — Claude ↔ ServiceNow

> **What is MCP?** Model Context Protocol (MCP) is an open standard that
> lets Claude Code talk directly to external systems — including your
> ServiceNow instance. With MCP configured, you can ask Claude to query
> tables, create records, and inspect your instance without leaving your
> terminal.

### 7.1 Install the ServiceNow MCP Server

```bash
npx -y @servicenow/mcp-server --help
```

This fetches and verifies the MCP server package.

### 7.2 Configure Credentials (Local Only)

Edit `.claude/settings.json` (the one you created in Phase 4),
replacing placeholders with your real PDI values:

```json
{
  "mcpServers": {
    "servicenow": {
      "command": "npx",
      "args": ["-y", "@servicenow/mcp-server"],
      "env": {
        "SN_INSTANCE_URL": "https://dev12345.service-now.com",
        "SN_USERNAME": "admin",
        "SN_PASSWORD": "your-actual-password"
      }
    }
  }
}
```

> **CRITICAL:** This file with real credentials must NEVER be committed
> to git. Rename it to `.claude/settings.local.json` (which is gitignored)
> and keep the template version (with placeholders) as `.claude/settings.json`.

### 7.3 Restart Claude Code

```bash
# Exit Claude Code (Ctrl+C or /exit), then restart
claude
```

### 7.4 Verify MCP Connection

In Claude Code, type:

```
/mcp
```

You should see `servicenow` listed as a connected server.

### 7.5 Try It — Ask Claude About Your Instance

Example prompts to try:
- "List the tables in my ServiceNow scoped app"
- "Show me the Script Includes in scope x_snc_training"
- "Create a test incident record on my instance"

Claude will use the MCP connection to query your instance in real time.

---

## Summary: Full Workflow Reference

```
Day-to-day development loop:
┌─────────────────────────────────────────────────┐
│ 1. git checkout -b feature/my-change            │
│ 2. snc app pull          (sync from instance)   │
│ 3. Edit files in Claude Code / VS Code          │
│ 4. snc app push          (push to instance)     │
│ 5. Verify change in PDI browser                 │
│ 6. git add src/ && git commit -m "feat: ..."    │
│ 7. git push && gh pr create                     │
│ 8. Merge PR after review                        │
└─────────────────────────────────────────────────┘
```
```

**Step 2: Commit**

```bash
git add docs/guide.md
git commit -m "docs: add Phase 7 (MCP setup) and workflow summary"
git push
```

---

## Task 8: Generate HTML Version

**Files:**
- Create: `docs/guide.html`

**Step 1: Check for pandoc or marked**

```bash
which pandoc || which marked || which node
```

**Step 2: Convert Markdown to HTML**

Option A — pandoc (preferred):
```bash
pandoc docs/guide.md \
  --standalone \
  --toc \
  --metadata title="ServiceNow Fluent Development Guide" \
  -o docs/guide.html
```

Option B — if pandoc not available, use marked via npx:
```bash
npx marked docs/guide.md > docs/guide.html
```

**Step 3: Verify HTML opens correctly**

```bash
open docs/guide.html
```

Review in browser — check headings, code blocks, and table of contents.

**Step 4: Commit**

```bash
git add docs/guide.html
git commit -m "docs: generate HTML version of guide"
git push
```

---

## Task 9: Final Review & PR

**Step 1: Run final checks**

```bash
# Confirm all files committed
git status

# Confirm remote is up to date
git log --oneline origin/main..HEAD
```

Expected: clean status, no unpushed commits.

**Step 2: Open PR if working on a branch**

```bash
gh pr create \
  --title "feat: ServiceNow Fluent Development Guide" \
  --body "Full step-by-step training guide for snc CLI setup, PDI auth, scoped app creation, Claude Code integration, dev loop, git workflow, and MCP setup."
```

**Step 3: Merge**

```bash
gh pr merge --squash
```

---

## Verification Checklist

Before declaring complete:

- [ ] `docs/guide.md` exists and covers all 7 phases
- [ ] `docs/guide.html` is generated and renders correctly in browser
- [ ] `README.md` links to guide
- [ ] `.gitignore` excludes snc credentials and `.env` files
- [ ] `.claude/settings.json` template is committed (placeholders only, no real creds)
- [ ] All commits are on a branch, merged via PR
- [ ] `git status` is clean
