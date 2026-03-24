# Design: ServiceNow Fluent Development Guide

**Date:** 2026-03-24
**Status:** Approved

## Purpose

Create a step-by-step training guide for setting up a ServiceNow scoped app using the ServiceNow CLI (snc), connecting it to a PDI, developing locally in Claude Code, and pushing code to both the instance and GitHub. Serves as onboarding material for the team.

## Audience

Mixed: new ServiceNow developers and experienced devs unfamiliar with CLI-based development.

## Output Format

- `docs/guide.md` — Primary narrative runbook (Markdown)
- `docs/guide.html` — Same content in HTML format
- `README.md` — Short intro + links to guide

## Project Structure

```
builderagentest/
├── docs/
│   ├── guide.md
│   ├── guide.html
│   └── plans/
│       └── 2026-03-24-servicenow-fluent-guide-design.md
├── src/                      # ServiceNow scoped app (synced via snc)
├── .gitignore
├── README.md
└── .claude/
    └── settings.json         # MCP config for ServiceNow instance
```

## Phases

### Phase 1: Prerequisites & Installation
- Check for existing `snc` install + version (`snc --version`)
- Install/upgrade path from ServiceNow Developer site
- Verify Node.js version requirement
- Install GitHub CLI (`gh`) and configure git identity
- Verify all tools: snc, git, gh, node

### Phase 2: PDI Setup & Authentication
- Assumes everyone already has a PDI (internal process, not documented here)
- `snc configure` — connecting snc to PDI instance URL + credentials
- Verify connection: `snc instance info`

### Phase 3: Create Scoped App
- `snc app create` — walk through prompts (scope, name, version)
- Brief explanation of scoped apps (beginner-friendly, skippable for experienced devs)
- First sync: pull initial app structure to `src/`

### Phase 4: Connect to Claude Code
- Open project in Claude Code (`claude` from project root)
- CLAUDE.md setup for ServiceNow context
- MCP config: add ServiceNow MCP server to `.claude/settings.json`
- Verify Claude can see the instance

### Phase 5: Development Loop
- `snc app pull` — sync instance changes down to local `src/`
- Edit files locally in VS Code / Claude Code
- `snc app push` — push local changes back to instance
- When to pull vs push; handling conflicts

### Phase 6: Git Workflow
- `git init` + connect to GitHub remote
- Branch naming: `feature/`, `fix/`, `chore/`
- What to commit (src/ changes) vs ignore (snc auth config, secrets)
- PR workflow via `gh pr create`
- `.gitignore` template for ServiceNow projects

### Phase 7: MCP Setup
- What MCP is and why it matters
- Install ServiceNow MCP server
- Configure `.claude/settings.json` with instance URL
- Demo: Claude querying a table / interacting with instance directly
- Security: never commit instance credentials to git

## Constraints & Assumptions

- Audience is ServiceNow internal — PDI provisioning process is known, not documented
- Generic example scoped app used for the walkthrough (adaptable to real apps)
- Source control: GitHub
- Instance type: PDI
