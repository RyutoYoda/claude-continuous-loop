# Architecture

## Flow

```mermaid
flowchart LR
    A[Issue] --> B["Script:<br/>Branch + PR"]
    B --> C[Planner]
    C --> D[Generator]
    D --> E[Evaluator]
    E -->|fail| D
    E -->|pass| F{"More files?"}
    F -->|yes| D
    F -->|done| G["Claude:<br/>Next Issue"]
    G --> H["Script:<br/>Build"]
    H -->|pass| I["Script:<br/>Merge"]
    H -->|fail| J["Script:<br/>@claude comment"]
    J --> K[Claude fixes]
    K --> H
    I -.->|"next Issue triggers"| A

    style B fill:#2d6a4f,color:#fff
    style H fill:#2d6a4f,color:#fff
    style I fill:#2d6a4f,color:#fff
    style J fill:#c9a84c,color:#000
    style G fill:#7c6af7,color:#fff
```

Green = shell script (guaranteed). Purple = Claude. Yellow = build fix loop.

## Two jobs

| Job | Trigger | Purpose |
|-----|---------|---------|
| `implement` | Issue created | Branch, PR, implement, build, merge |
| `fix` | PR comment with `@claude` | Fix build failures, re-verify, merge |

## Next Issue creation

Claude creates the next Issue (actor = `claude[bot]`), which triggers the `implement` job.

`GITHUB_TOKEN` cannot trigger workflows on the same repo (GitHub recursion prevention).
Only `claude[bot]` as actor can trigger the next cycle.

## Build fix loop

```mermaid
flowchart LR
    A["Script:<br/>npm run build"] -->|pass| B[Merge]
    A -->|fail| C["Script:<br/>@claude + error"]
    C --> D[Claude fixes]
    D --> E["Script:<br/>Re-build"]
    E -->|pass| B
    E -->|"fail x3"| F[Close PR]

    style A fill:#2d6a4f,color:#fff
    style B fill:#2d6a4f,color:#fff
    style C fill:#c9a84c,color:#000
    style E fill:#2d6a4f,color:#fff
    style F fill:#d62828,color:#fff
```

3 build failures = PR closed (prevents infinite retry).

## Script vs Agent

| Step | Owner | Guaranteed? |
|------|-------|-------------|
| Branch + PR creation | Script | Yes |
| @claude injection | Script | Yes |
| Planning | Planner agent | Best effort |
| Implementation | Generator agent | Best effort |
| Code review | Evaluator agent | Best effort |
| Next Issue creation | Claude (claude[bot]) | Best effort |
| Build verification | Script | Yes |
| Build fix request | Script | Yes |
| Auto-merge | Script | Yes |
| Retry limit (3x) | Script | Yes |

## Branch strategy

Each Issue = one branch = one PR.

```mermaid
gitGraph
    commit id: "main"
    branch claude/issue-1
    commit id: "Issue #1"
    checkout main
    merge claude/issue-1
    branch claude/issue-2
    commit id: "Issue #2"
    checkout main
    merge claude/issue-2
    branch claude/issue-3
    commit id: "Issue #3"
    checkout main
    merge claude/issue-3
```
