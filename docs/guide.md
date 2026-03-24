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
- Node.js **v20.18.0 or later** ([nodejs.org](https://nodejs.org) — choose LTS)

---

## Phase 1: Installation & Verification

### 1.1 Check Existing Tools

Run each command to verify what's already installed:

```bash
now-sdk --version    # ServiceNow SDK (Fluent dev engine)
node --version       # Node.js — need v20.18.0+
npm --version        # npm — need v8.19.3+
git --version        # Git
gh --version         # GitHub CLI
```

If any command returns "command not found", follow the install steps below.

### 1.2 Install Node.js (if needed)

The ServiceNow SDK requires **Node.js v20.18.0 or later**.

```bash
node --version
```

If below v20.18.0 or not installed:
- **macOS (recommended):** `brew install node@20`
- **All platforms:** Download **LTS** from [nodejs.org](https://nodejs.org)

Verify after install:

```bash
node --version   # Should show v20.18.0 or higher
npm --version    # Should show v8.19.3 or higher
```

### 1.3 Install ServiceNow SDK (now-sdk)

> **What is now-sdk?** The ServiceNow SDK is the Fluent pro-code development
> engine. It compiles Fluent DSL (TypeScript-like metadata) to ServiceNow
> records and manages the full app lifecycle: `init`, `download`, `install`.

```bash
npm install -g @servicenow/sdk
```

**Verify:**

```bash
now-sdk --version
```

Expected output (version may differ):
```
4.4.x
```

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

## Phase 2: Instance Authentication

> This phase connects your local `now-sdk` to your PDI.
> The SDK has its own built-in authentication — no separate CLI required.

### 2.1 Add Your Instance as an Alias

```bash
now-sdk auth --add <your-alias>
```

Replace `<your-alias>` with a short name for your instance, e.g., `dev`:

```bash
now-sdk auth --add dev
```

You will be prompted for:
- **Instance URL:** `https://your-instance.service-now.com`
- **Username:** Your ServiceNow login
- **Password:** Your ServiceNow password

> **Security note:** Credentials are stored locally by the SDK.
> Never hardcode them in project files or commit them to git.
> The `.gitignore` in this project already excludes credential files.

### 2.2 Verify Authentication

```bash
now-sdk auth --list
```

Expected output shows your alias and instance URL with a connected status.

If you see an authentication error:
- Double-check your instance URL (no trailing slash)
- Verify your credentials work by logging into the instance in a browser
- Re-run `now-sdk auth --add dev`

### 2.3 CI/CD Authentication (Optional)

For non-interactive environments (pipelines, automation), use environment variables instead of stored credentials:

```bash
export SN_SDK_INSTANCE_URL="https://your-instance.service-now.com"
export SN_SDK_USER="your-username"
export SN_SDK_USER_PWD="your-password"
export SN_SDK_NODE_ENV="SN_SDK_CI_INSTALL"
```

---

## Phase 3: Create a Fluent Scoped App

> **What is a Fluent scoped app?** In the modern SDK workflow, you write
> app metadata as Fluent DSL — a TypeScript-like language. The SDK compiles
> it to ServiceNow records and installs them to your instance.
> All development is local-first. If you know this already, skip to 3.1.

### 3.1 Initialize the App

```bash
now-sdk init
```

You will be prompted to choose a template (e.g., `fullstack React` for UI, or a basic app template) and provide:
- **App name:** e.g., `My Training App`
- **Scope:** e.g., `x_snc_training` (auto-suggested, you can customize)
- **Version:** `1.0.0`
- **Instance alias:** the alias you set in Phase 2 (e.g., `dev`)

The SDK scaffolds a local Fluent project structure in a new directory.

### 3.2 Install Dependencies

After `now-sdk init`, install the project's npm dependencies before doing anything else:

```bash
npm install
```

This pulls in React, TypeScript, the ServiceNow SDK libraries, and all other packages defined in `package.json`. You only need to do this once after init (and again if `package.json` changes).

### 3.3 Explore the Project Structure (for reference)

After `now-sdk init` completes, your project will look like this:

```
my-training-app/
├── src/                   # Your Fluent DSL source files
│   └── ...
├── now.config.json        # SDK project config (scope, instance alias, etc.)
├── now.dev.mjs            # Dev server entry point
├── now.prebuild.mjs       # Prebuild script
├── package.json           # npm config and dependencies
├── .eslintrc              # Linting rules
└── .vscode/               # VS Code workspace settings
```

> **Note:** The config file is `now.config.json` — not `now-sdk.json` as shown
> in some older documentation.

### 3.4 Download Existing App Metadata (if applicable)

> **Only applies if** the app already exists on your instance AND you have already
> run `now-sdk init` locally. The `download` command requires a `now-sdk.json`
> project config (created by `init`) to know which scope to pull.

If those conditions are met, sync from your instance:

```bash
now-sdk download src/
```

This pulls the existing app metadata from your PDI into the local `src/` directory.

If you see `No valid scope found` — the app isn't on the instance yet. Run
`now-sdk install` first (Phase 5.4) to push your local app up, then download
will work for subsequent syncs.

### 3.5 Verify in the Instance

Log into your PDI and navigate to:
**App Engine Studio → My Apps** (or **System Applications → Studio**)

Your app will appear after you run `now-sdk install` in Phase 5.

---

## Phase 4: Connect to Claude Code

> **What is Claude Code?** Claude Code (CC) is Anthropic's AI-powered CLI
> development environment. It understands your Fluent codebase and can help
> you write, review, and install ServiceNow app code — including talking
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
- Build (compile only):   `now-sdk build`
- App source: `src/`

## Conventions
- All changes installed to instance AND committed to git
- Never commit SDK credentials or instance auth files
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

The core Fluent workflow: edit locally → build → install to instance.
Download is only used to sync changes made directly on the instance back to local.

---

### When to use each command

| Command | When to use |
|---------|-------------|
| `now-sdk build` | After editing — compiles Fluent DSL to `dist/`. **Required before install.** |
| `now-sdk install` | After building — deploys `dist/` to your instance. |
| `now-sdk download src/` | Only when changes were made directly on the instance (e.g. in Studio). Syncs them back to local. |

---

### 5.1 Edit Files Locally

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

### 5.2 Build — Compile Fluent DSL

**This step is required before every install.** It compiles your source into
`dist/`, which is what gets deployed to the instance.

```bash
now-sdk build
```

Fix any errors reported before proceeding. The `dist/` directory is created
(or updated) by this command — `now-sdk install` will fail if you skip it.

### 5.3 Install to Instance

```bash
now-sdk install
```

Deploys the compiled `dist/` to your PDI. Verify in the instance: open the
record you changed in Studio and confirm your edits are there.

### 5.4 Download (Only When Needed)

Use `now-sdk download` **only** if changes were made directly on the instance
(e.g. someone edited a record in Studio) and you need to pull them back to local.

```bash
now-sdk download src/
```

> **Scope error?** If you see `No valid scope found`, the scope in your
> `now.config.json` doesn't exist on the target instance. This happens when
> the project was initialized against a different instance. Re-run
> `now-sdk init` or update `now.config.json` to point to the correct instance
> and scope.

> **Rule of thumb:** Local is the source of truth. Edit locally → build → install.
> Only download when you have no choice (instance-side edits you need back).

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
│ 2. Edit Fluent DSL in Claude Code / VS Code         │
│ 3. now-sdk build         (compile DSL → dist/)      │
│ 4. now-sdk install       (deploy dist/ to PDI)      │
│ 5. Verify change in PDI browser                     │
│ 6. git add src/ && git commit -m "feat: ..."        │
│ 7. git push && gh pr create                         │
│ 8. Merge PR after review                            │
└─────────────────────────────────────────────────────┘

Note: Use `now-sdk download src/` only when changes were made directly
on the instance and you need to pull them back to local.
```
