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

> **Deprecation warnings are normal.** `npm install` will print warnings about
> deprecated packages (inflight, glob, eslint, etc.). These come from the SDK's
> own dependencies — ignore them and proceed.

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

### 4.3 MCP Setup (See Phase 7)

MCP is configured once via the `claude mcp add` command — not via a
settings file. See Phase 7 for the full setup (takes under a minute).

---

## Phase 5: Development Loop

The core Fluent workflow: edit locally → build → install to instance.
Download is only used to sync changes made directly on the instance back to local.

---

### When to use each command

| Command | When to use |
|---------|-------------|
| `now-sdk build` | After editing — compiles Fluent DSL to `dist/`. **Required before install.** |
| `now-sdk install --alias <name>` | After building — deploys `dist/` to your instance. |
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
now-sdk install --alias <your-alias>
```

Replace `<your-alias>` with the alias you set in Phase 2 (e.g., `dev`):

```bash
now-sdk install --alias dev
```

Deploys the compiled `dist/` to your PDI. On success, the SDK prints a direct
URL to your app on the instance:

```
App installed successfully.
https://your-instance.service-now.com/sys_app.do?sys_id=...
```

Open that URL to verify the install — you can confirm your changes are live
directly in the instance browser.

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

## Phase 5b: ServiceNow IDE Setup

> **What is the ServiceNow IDE?** A standalone desktop IDE (VS Code-based)
> built by ServiceNow. It includes the Build Agent (Now Assist), native
> `now-sdk` integration via the **Build and Install** button, and a Source
> Control panel for git workflows — all in one place.
>
> Do this after your app is successfully installed to the instance (Phase 5.3).

### 5b.1 Open Your Project

Launch the ServiceNow IDE, then open your project folder:

**File → Open Folder** → navigate to your local `now-sdk` project directory
(the folder containing `now.config.json`) → **Open**

The IDE loads your project. The Source Control panel (branch icon in the left
sidebar) will show **"No connected repositories"** if git hasn't been
initialized yet in this folder.

### 5b.2 Load Your App in Build Agent

In the right-hand **Build Agent** panel:

1. Click the **"Create or import application"** dropdown
2. Select your app from the list (it will appear because you installed it in Phase 5.3)

This connects the IDE to your app on the instance — enabling Build Agent to
query your app, create files, and interact with your ServiceNow records
directly from the IDE.

### 5b.3 Initialize the Local Git Repository

In the **Source Control** panel:

1. Click **"Initialize Repository"**
2. When prompted for the default branch name, type `main` and press **Enter**

The IDE initializes a local git repo in your project folder with `main` as
the default branch. You'll see your project files appear as untracked changes
in the Source Control panel.

> **Already have a repo?** If you initialized git outside the IDE (e.g. via
> terminal), the Source Control panel will detect it automatically — skip
> the Initialize step.

---

## Phase 6: Git Workflow

### 6.1 Create a GitHub Remote Repository

Before pushing, create a remote repo on GitHub and link it to your local project.

**Option A — One command (create + link + push):**

```bash
gh repo create <repo-name> --private --source=. --remote=origin --push
```

Example:
```bash
gh repo create build-agent-troubleshooter --private --source=. --remote=origin --push
```

This creates the GitHub repo, sets `origin` as the remote, and pushes `main`
in a single step. Use `--public` instead of `--private` if appropriate.

**Option B — Create repo first, then link manually:**

```bash
# Create the repo on GitHub
gh repo create <repo-name> --private

# Add it as the remote
git remote add origin https://github.com/<your-username>/<repo-name>.git

# Push main and set tracking
git push -u origin main
```

### 6.2 Link the ServiceNow IDE to the Remote

After setting the remote (step 6.1), the ServiceNow IDE Source Control panel
detects `origin` automatically. To verify and configure credentials:

1. In the IDE, open Source Control (branch icon in sidebar)
2. You should see your commits and the remote listed — if prompted, authenticate with GitHub
3. Use the **"..."** menu → **"Push"** to push any pending commits to `origin/main`

To confirm the remote is wired correctly from the terminal:

```bash
git remote -v
```

Expected output:
```
origin  https://github.com/<your-username>/<repo-name>.git (fetch)
origin  https://github.com/<your-username>/<repo-name>.git (push)
```

### 6.3 Create a Feature Branch

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

### 6.4 Stage and Commit

After installing to your instance and verifying it works:

```bash
git add src/
git commit -m "feat: add incident utility functions"
```

Keep commits small and focused. One logical change per commit.

### 6.5 Push Branch and Open PR

```bash
git push -u origin feature/my-first-change
gh pr create --title "Add incident utility functions" --body "Adds formatIncidentNumber utility to IncidentUtils Script Include."
```

### 6.6 Merge and Clean Up

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
> ServiceNow instance. With MCP configured, you can ask Claude to interact
> with your instance, manage tools, and run agentic workflows without
> leaving your terminal.

### 7.1 Register the MCP Server

Run this once. Using `--scope user` makes the server available in every
project on your machine — you won't need to repeat this per project.

```bash
# User scope (recommended — available in every project)
claude mcp add --scope user --transport stdio foundry -- \
  node ~/projects/tool-foundry-mcp/dist/index.js
```

### 7.2 Verify Registration

```bash
claude mcp list
```

You should see `foundry` listed in the output.

### 7.3 Try It — Ask Claude About Your Instance

Example prompts to try in Claude Code:
- "List the tools available in foundry"
- "Show me the ServiceNow app metadata for my current project"
- "What Script Includes are in my scoped app?"

Claude will use the MCP connection to interact with your instance in real time.

---

## Summary: Full Workflow Reference

```
One-time setup (per app):
┌─────────────────────────────────────────────────────┐
│ 1. now-sdk init && npm install                      │
│ 2. now-sdk build && now-sdk install --alias <name>  │
│ 3. Open project in ServiceNow IDE                   │
│ 4. Source Control → Initialize Repository → "main"  │
│ 5. gh repo create --private --source=. --remote=origin --push │
└─────────────────────────────────────────────────────┘

Day-to-day Fluent development loop:
┌─────────────────────────────────────────────────────┐
│ 1. git checkout -b feature/my-change                │
│ 2. Edit Fluent DSL in Claude Code / VS Code         │
│ 3. now-sdk build         (compile DSL → dist/)      │
│ 4. now-sdk install --alias <alias>  (deploy to PDI) │
│ 5. Verify change in PDI browser                     │
│ 6. git add src/ && git commit -m "feat: ..."        │
│ 7. git push && gh pr create                         │
│ 8. Merge PR after review                            │
└─────────────────────────────────────────────────────┘

Note: Use `now-sdk download src/` only when changes were made directly
on the instance and you need to pull them back to local.
```
