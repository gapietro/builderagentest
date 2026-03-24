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
