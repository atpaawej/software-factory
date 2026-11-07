# Software Factory

An autonomous, issue-driven software factory skill following the [agentskills.io](https://agentskills.io) specification.

The software factory transforms GitHub issues into tested, draft pull requests via unattended agent runs inside GitHub Actions. Humans triage, specify, and merge. The factory executes, tests, and documents. Nothing in this system merges to the default branch on its own.

---

## Architecture & Trust Boundaries

```text
       [ GitHub Issue ]
              │
       (triage / queue)
              ▼
    [ agent:implement ] ──► [ GitHub Actions Orchestrator ]
                                     │
                     ┌───────────────┴───────────────┐
                     ▼                               ▼
            (Preflight Check)                 (Acquire Lock)
                     │                      `agent:in-progress`
                     ▼
          [ Sandboxed Subshell ]
          • Token-scrubbed (`env -i`)
          • NO GITHUB_TOKEN passed
          • Edits code, runs local tests
          • Commits: "Part of #N: ..."
          • Writes: /tmp/factory-pr-body.md
                     │
                     ▼
        [ Orchestrator Delivery ]
        • Rebase onto origin/main
        • Force push: `agent/issue-<N>-<slug>`
        • Neutralize closing keywords
        • REST API: Open Draft PR (`Closes #N`)
        • Release lock: remove `agent:in-progress`
                     │
                     ▼
              [ Draft PR Open ]
              • Red / Green CI status
              • Agent reflection body
              • Human review & merge
```

### Core Security Principles

1. **Tokenless Cleanroom Execution**: The coding agent executes inside an `env -i` scrubbed environment. It receives no `GITHUB_TOKEN` or repository write permissions.
2. **CI is the Leash**: The agent must satisfy local test, typecheck, and build commands before exiting. Remote CI provides independent validation on the draft PR.
3. **Closing Keyword Neutralization**: Closing verbs in agent prose (`fixes #12` $\to$ `fixes ~~#12~~`) are sanitized so an agent diff cannot inadvertently close other issues.
4. **Mandatory Human Merge**: Only humans review and merge pull requests to `main`.

---

## State Machine

| Label | Meaning |
| :--- | :--- |
| `ready-for-agent` | Specification is triage-approved and ready for work. (Triage signal) |
| `agent:implement` | **The Trigger.** Starts an autonomous run immediately. |
| `agent:queued` | Queued in backlog. Auto-promoted when all native blockers close. |
| `agent:in-progress` | **Execution Lock.** Active runner holds this issue; prevents duplicate runs. |
| `agent:blocked` | Execution failed or was refused. Requires human review. |

---

## Quickstart

### 1. Initialize a Repository

Run the initializer script against any target git repository:

```bash
./scripts/init-factory.sh --install-to /path/to/my-repo
```

This will:
- Copy workflows to `.github/workflows/` (`agent-implement.yml`, `agent-promote-queued.yml`).
- Copy the PR description builder to `.github/scripts/build-pr-body.sh`.
- Create the 5 state machine labels on the remote GitHub repository.
- Configure GitHub Actions permissions to permit pull request creation.

### 2. Set Up Model Credentials

Configure the runner API key secret in your GitHub repository:

```bash
gh secret set OPENCODE_API_KEY
```

### 3. Run It on an Issue

Label an issue with `agent:implement`:

```bash
gh issue edit <ISSUE_NUMBER> --add-label "agent:implement"
```

Or trigger manually:

```bash
gh workflow run agent-implement.yml -f issue_number=<ISSUE_NUMBER>
```

---

## Directory Structure

```text
.
├── SKILL.md                          # agentskills.io entry point
├── README.md                         # Architecture and documentation
├── scripts/
│   └── init-factory.sh               # CLI setup and repository bootstrapper
├── assets/
│   ├── workflows/
│   │   ├── agent-implement.yml       # Single-issue implementer workflow
│   │   └── agent-promote-queued.yml  # Blocker-aware queue promoter workflow
│   └── scripts/
│       └── build-pr-body.sh          # PR description compiler & keyword neutralizer
└── references/
    ├── state-machine.md              # State transitions and lifecycle rules
    ├── trust-boundaries.md           # Token scrubbing and sandbox architecture
    └── troubleshooting.md            # Diagnosing and recovering from failures
```

---

## License

MIT
