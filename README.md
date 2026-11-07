# Software Factory 🏭

> **Stop babysitting AI agents in terminal tabs.**  
> Turn your GitHub issues into tested, draft pull requests while you sleep.

[![Install with skills.sh](https://img.shields.io/badge/install-skills.sh-blueviolet?style=for-the-badge)](https://skills.sh)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)](LICENSE)
[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)](https://github.com/features/actions)

```bash
npx skills add atpaawej/software-factory
```

---

## Why Software Factory?

Right now, using AI coding agents feels like a full-time babysitting job:

1. You open a terminal tab.
2. You craft a prompt, run an agent, and stare at the screen for 8 minutes.
3. You review tiny diffs, nudge context, manually run tests, and hand-craft git commits.
4. You repeat this for every single issue in your backlog.

**You became the bottleneck between your issues and your code.**

The **Software Factory** flips this model entirely. It turns your GitHub issue tracker into an asynchronous, autonomous development pipeline:

```text
  You label an issue: `agent:implement`
                     │
                     ▼
  (Close your laptop. Go live your life.)
                     │
                     ▼
  • GitHub Actions spins up an isolated, tokenless cleanroom.
  • The agent writes code, passes local tests, typechecks, and linters.
  • Orchestrator force-pushes a deterministic branch.
  • A Draft PR arrives in your inbox with full rationale and green CI checks.
                     │
                     ▼
  You review the diff and click Merge.
```

---

## ⚡ Install in 60 Seconds

### Step 1: Install the Skill via `skills.sh`

Add the skill to your agent globally or in your current project:

```bash
npx skills add atpaawej/software-factory
```

*(Works with Claude Code, Cursor, Codex, Windsurf, OpenCode, and any agentskills.io-compatible agent).*

---

### Step 2: Initialize Your Repository

Run the automated bootstrapper in any target repo:

```bash
./scripts/init-factory.sh
```

This single command:
- ✅ Configures the 5 state-machine labels (`ready-for-agent`, `agent:implement`, `agent:queued`, `agent:in-progress`, `agent:blocked`).
- ✅ Grants GitHub Actions permissions to create pull requests.
- ✅ Installs production workflows in `.github/workflows/`.
- ✅ Installs the PR reflection generator in `.github/scripts/`.

---

### Step 3: Add Your Free Runner Key

Set your LLM API key once in your GitHub repository secrets:

```bash
gh secret set OPENCODE_API_KEY
```

> **$0 Operating Cost:** Configured out-of-the-box to run with account-authenticated free-tier models (`opencode/deepseek-v4.1-flash-free`) on standard GitHub Actions runners. Zero credit card balance required.

---

## 🎯 How It Works in Practice

### 1. Trigger an Issue Run
When an issue has clear acceptance criteria:

```bash
gh issue edit <ISSUE_NUMBER> --add-label agent:implement
```

The factory locks the issue (`agent:in-progress`), executes in GitHub Actions, and opens a draft PR linking back with `Closes #<N>`.

### 2. Queue Dependent Features (Auto-Chaining)
Have a feature that depends on another issue being completed first?

1. Add native GitHub dependency:
   ```bash
   BLOCKER_ID=$(gh api repos/:owner/:repo/issues/<BLOCKER_N> --jq .id)
   gh api --method POST repos/:owner/:repo/issues/<N>/dependencies/blocked_by -F issue_id="$BLOCKER_ID"
   ```
2. Label the issue:
   ```bash
   gh issue edit <N> --add-label agent:queued
   ```

**The moment the blocking issue merges and closes, the factory automatically promotes `#N` to `agent:implement` and launches the run.** No human babysitting required.

---

## 🛡️ The Zero-Risk Trust Boundary

You shouldn't trust an AI agent with root access to your repository. Neither do we.

The Software Factory enforces military-grade isolation between the untrusted agent and your code:

| What Runaway Agents Could Do | How Software Factory Guarantees Safety |
| :--- | :--- |
| **Exfiltrate tokens / push rogue tags** | **Token-Scrubbed Sandbox (`env -i`):** Neither `GITHUB_TOKEN` nor any repo secret is passed to the agent subshell. It physically cannot touch GitHub. |
| **Accidentally push to `main`** | **Deterministic Branch Isolation:** Work is strictly committed to `agent/issue-<N>-<slug>`. |
| **Merge broken code** | **Mandatory Human Merge:** Only human maintainers can merge PRs. The factory opens **Draft PRs only**. |
| **Pass broken tests** | **CI is the Leash:** The agent must pass local test, typecheck, and build suites before exiting; GitHub Actions CI displays red/green status. |
| **Accidentally auto-close other issues** | **Keyword Sanitization:** Closing statements in agent prose (`fixes #12` $\to$ `fixes ~~#12~~`) are stripped so merged PRs never close unintended issues. |

---

## 📋 The State Machine at a Glance

| Label | Who Applies It | What It Means |
| :--- | :--- | :--- |
| `ready-for-agent` | Human (Triage) | Issue is fully specified. Triage signal only. |
| `agent:implement` | Human or Auto-Promote | **The Trigger.** Spawns an immediate factory run. |
| `agent:queued` | Human | Backlog queue. Auto-triggers when all blockers close. |
| `agent:in-progress` | Factory Orchestrator | **Lock.** Runner is active; prevents duplicate runs. |
| `agent:blocked` | Factory Orchestrator | Run failed or was refused. Needs human eyes. |

---

## 🔍 PR Descriptions Written for Humans

A reviewer shouldn't have to decipher an agent's diff from scratch.

Before stopping, the agent writes its rationale to `/tmp/factory-pr-body.md`. Our compiler packages it into 6 structured sections on every draft PR:

1. **Summary**: Two to four sentences on what this change makes possible.
2. **What changed**: Grouped by module and component with file paths.
3. **Decisions and deviations**: Architectural choices and departures from original spec.
4. **Verification**: Exact test and build commands verified.
5. **Gaps and follow-ups**: Intentional omissions and things to watch.
6. **Review guide**: Where to start reading in the diff.

---

## 🗂️ Skill Structure

```text
.
├── SKILL.md                          # agentskills.io manifest
├── README.md                         # You are here
├── scripts/
│   └── init-factory.sh               # 1-command repository bootstrapper
├── assets/
│   ├── workflows/
│   │   ├── agent-implement.yml       # Single-issue implementation workflow
│   │   └── agent-promote-queued.yml  # Automated blocker queue promoter
│   └── scripts/
│       └── build-pr-body.sh          # PR description compiler & keyword neutralizer
└── references/
    ├── state-machine.md              # Lifecycle rules and transition mechanics
    ├── trust-boundaries.md           # Token scrubbing and sandbox security
    └── troubleshooting.md            # Failure playbook and recovery commands
```

---

## 🚀 Get Started Now

Install with one command and build your first autonomous issue pipeline today:

```bash
npx skills add atpaawej/software-factory
```

---

## License

MIT © [atpaawej](https://github.com/atpaawej)
