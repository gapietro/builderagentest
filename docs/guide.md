# ServiceNow Fluent Development Guide

> A step-by-step runbook for setting up a ServiceNow scoped app using the
> modern Fluent/SDK pro-code workflow, developing locally in Claude Code,
> and syncing code to your PDI and GitHub.
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

## Toolchain Overview

This guide uses **two complementary ServiceNow CLI tools**:

| Tool | Purpose | Install method |
|------|---------|----------------|
| `snc` | Instance authentication / profile management | Binary installer |
| `now-sdk` | Fluent pro-code app development (create, download, install) | npm |

Both are required. `snc` manages your instance connection; `now-sdk` is the Fluent development engine.

---

## Phase 1: Installation & Verification

### 1.1 Check Existing Tools

Run each command to verify what's already installed:

```bash
snc --version        # ServiceNow CLI (profile management)
now-sdk --version    # ServiceNow SDK (Fluent dev engine)
node --version       # Node.js — need v20.18.0+
git --version        # Git
gh --version         # GitHub CLI
```

If any command returns "command not found", follow the install steps below.

### 1.2 Install / Upgrade ServiceNow CLI (snc)

> **What is snc?** The ServiceNow CLI handles instance authentication and
> profile management. It is a binary installer — not available via npm or brew.

**Download snc:**

1. Go to: [developer.servicenow.com](https://developer.servicenow.com)
2. Sign in with your ServiceNow credentials
3. Navigate to: **Tools → ServiceNow CLI**
4. Download the installer for your OS and run it

**Alternative:** Download directly from [GitHub releases](https://github.com/ServiceNow/servicenow-cli/releases/latest) (no Store credentials required).

**Verify installation:**

```bash
snc --version
```

Expected output (version may differ):
```
snc/1.1.3 <os>-<arch> node-vxx.x.x
```

**If you have an old version**, re-run the installer — it upgrades in place.

### 1.3 Install ServiceNow SDK (now-sdk)

> **What is now-sdk?** The ServiceNow SDK is the Fluent pro-code development
> engine. It compiles Fluent DSL to ServiceNow metadata, and manages
> app creation, download from instance, and installation to instance.

```bash
npm install -g @servicenow/sdk
```

**Verify:**

```bash
now-sdk --version
```

Expected output (version may differ):
```
4.x.x
```

### 1.4 Install Node.js (if needed)

The ServiceNow SDK requires **Node.js v20.18.0 or later**.

```bash
node --version
```

If below v20.18.0 or not installed:
- **macOS (recommended):** `brew install node@20`
- **All platforms:** Download from [nodejs.org](https://nodejs.org) — choose the **LTS** release

### 1.5 Install GitHub CLI (if needed)

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

### 1.6 Configure Git Identity

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
> This phase connects your local `snc` CLI to that instance.

### 2.1 Create an snc Profile

```bash
snc configure profile set
```

You will be prompted for:
- **Host:** `https://your-instance.service-now.com`
- **Login method:** Basic (username/password) or OAuth
- **Username:** Your ServiceNow login
- **Password:** Your ServiceNow password

> **Security note:** snc stores credentials locally in `~/.snc/`.
> Never copy these files into your project repo or commit them.
> The `.gitignore` in this project already excludes them.

To use a named profile (recommended if you have multiple instances):

```bash
snc configure profile set --profile dev
```

### 2.2 List Configured Profiles

```bash
snc configure profile list
```

Expected output shows your configured profiles and their instance URLs.

### 2.3 Set the SDK Auth Alias

The `now-sdk` tool uses the snc profile you just created. Link them:

```bash
now-sdk auth --profile dev
```

Or if using the default profile, `now-sdk` will pick it up automatically.

Verify the SDK can reach your instance:

```bash
now-sdk auth --list
```

If you see an authentication error:
- Double-check your instance URL (no trailing slash)
- Verify your credentials work by logging into the instance in a browser
- Re-run `snc configure profile set`

---

## Phase 3: Create a Fluent Scoped App

> **What is a Fluent scoped app?** In the modern ServiceNow SDK workflow,
> you write app metadata as Fluent DSL (a TypeScript-like DSL). The SDK
> compiles it to ServiceNow XML and installs it to your instance.
> All development is local-first — your instance is the target, not the editor.
> If you already know this, skip ahead to 3.1.

### 3.1 Initialize the App

```bash
now-sdk init
```

You will be prompted for:
- **App name:** e.g., `My Training App`
- **Scope:** e.g., `x_snc_training` (auto-suggested, you can customize)
- **Version:** `1.0.0`

The SDK scaffolds a local Fluent project structure in a new directory.

### 3.2 Explore the Project Structure

The scaffolded structure will look like:

```
my-training-app/
├── src/
│   ├── index.ts         # App entry point (Fluent DSL)
│   └── ...
├── now-sdk.json         # SDK project config
├── package.json
└── tsconfig.json
```

### 3.3 Download Existing App Metadata (if applicable)

If you're starting from an app that already exists on your instance:

```bash
now-sdk download
```

This pulls the existing app metadata from your PDI into the local `src/` directory.

> **Tip:** Always run `now-sdk download` before starting work to make sure
> your local files are in sync with the instance.

### 3.4 Verify in the Instance

Log into your PDI and navigate to:
**App Engine Studio → My Apps** (or **System Applications → Studio**)

Your new app should appear in the list after you install it in Phase 5.

---

## Phase 4: Connect to Claude Code

> **What is Claude Code?** Claude Code (CC) is Anthropic's AI-powered CLI
> development environment. It understands your codebase and can help you
> write, review, and install ServiceNow Fluent app code — including talking
> directly to your instance via MCP (covered in Phase 7).

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
# My Training App — ServiceNow Fluent Scoped App

## Project Overview
ServiceNow Fluent scoped app (scope: x_snc_training) on PDI instance.
Primary development via ServiceNow SDK (now-sdk) + Claude Code.

## Key Commands
- Download from instance: `now-sdk download`
- Install to instance:    `now-sdk install`
- Initialize new app:     `now-sdk init`
- App source: `src/`

## Conventions
- All changes installed to instance AND committed to git
- Never commit snc credentials or .snc/ directory
- Branch naming: feature/, fix/, chore/
```

### 4.3 MCP Settings (Placeholder — See Phase 7)

Create `.claude/settings.json` at your project root with this template:

```json
{
  "mcpServers": {
    "fluent-mcp": {
      "command": "npx",
      "args": ["-y", "@modesty/fluent-mcp"],
      "env": {
        "SN_INSTANCE_URL": "https://your-instance.service-now.com",
        "SN_AUTH_TYPE": "basic",
        "SN_USER_NAME": "your-username",
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

The core Fluent workflow: download from instance → edit Fluent DSL locally → install back.

### 5.1 Download Before You Start

Always sync the latest from your instance before editing:

```bash
now-sdk download
```

### 5.2 Edit Files Locally

Open files in `src/` using VS Code or Claude Code.
You'll be writing Fluent DSL — TypeScript-like metadata declarations:

```typescript
// Example: defining a Script Include in Fluent
export const IncidentUtils = scriptInclude({
  name: 'IncidentUtils',
  script: () => {
    // your script logic here
  }
});
```

> **Tip:** Ask Claude Code to help — e.g., "Add a utility function to
> IncidentUtils that formats an incident number as INC0001234."

### 5.3 Install to Instance

```bash
now-sdk install
```

This compiles your Fluent DSL and installs the resulting metadata to your PDI.

Verify in the instance: open the record you changed in Studio or the
application navigator and confirm your edits are there.

### 5.4 Handling Conflicts

If your instance has changes that aren't in your local files:

```bash
now-sdk download      # Get latest from instance first
# Review and merge any differences manually
now-sdk install       # Then install your changes
```

> **Rule of thumb:** Download first, always. Especially if others share the instance.

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

After installing to your instance and verifying it works:

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

---

## Phase 7: MCP Setup — Claude ↔ ServiceNow

> **What is MCP?** Model Context Protocol (MCP) is an open standard that
> lets Claude Code talk directly to external systems — including your
> ServiceNow instance. With MCP configured, you can ask Claude to query
> tables, inspect your app metadata, and interact with your instance
> without leaving your terminal.
>
> **Note:** There is no official ServiceNow MCP server. This guide uses
> `fluent-mcp` — a community package that wraps the ServiceNow SDK.

### 7.1 Install fluent-mcp

```bash
npx -y @modesty/fluent-mcp --help
```

This fetches and verifies the fluent-mcp package.

### 7.2 Configure Credentials (Local Only)

Edit `.claude/settings.json` (the one you created in Phase 4),
replacing placeholders with your real PDI values:

```json
{
  "mcpServers": {
    "fluent-mcp": {
      "command": "npx",
      "args": ["-y", "@modesty/fluent-mcp"],
      "env": {
        "SN_INSTANCE_URL": "https://dev12345.service-now.com",
        "SN_AUTH_TYPE": "basic",
        "SN_USER_NAME": "admin",
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

You should see `fluent-mcp` listed as a connected server.

### 7.5 Try It — Ask Claude About Your Instance

Example prompts to try:
- "List the Script Includes in my ServiceNow scoped app"
- "Show me the tables defined in scope x_snc_training"
- "What app metadata is installed on my instance?"

Claude will use the MCP connection to query your instance in real time.

---

## Summary: Full Workflow Reference

```
Day-to-day Fluent development loop:
┌─────────────────────────────────────────────────────┐
│ 1. git checkout -b feature/my-change                │
│ 2. now-sdk download      (sync from instance)       │
│ 3. Edit Fluent DSL in Claude Code / VS Code         │
│ 4. now-sdk install       (compile + push to PDI)    │
│ 5. Verify change in PDI browser                     │
│ 6. git add src/ && git commit -m "feat: ..."        │
│ 7. git push && gh pr create                         │
│ 8. Merge PR after review                            │
└─────────────────────────────────────────────────────┘
```
