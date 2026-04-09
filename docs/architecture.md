# Architecture

## Loop Flow

```mermaid
flowchart LR
    A[Issue] --> B[Script: Branch + PR] --> C[Planner] --> D[Generator]
    D --> E[Evaluator]
    E -->|fail| D
    E -->|pass| F{"More files?"}
    F -->|yes| D
    F -->|done| G[Script: Build Check]
    G --> H[Next Issue]
    H --> A

    style A fill:#7c6af7,color:#fff
    style B fill:#2d6a4f,color:#fff
    style G fill:#2d6a4f,color:#fff
    style H fill:#7c6af7,color:#fff
```

Green = guaranteed by shell scripts. Purple = triggers.

Each Issue gets its own branch and PR (1:1:1).

## What scripts guarantee vs what Claude handles

| Step | Who | Guaranteed? |
|------|-----|-------------|
| Branch + PR creation | Shell script | Yes |
| `@claude` tag injection | Shell script | Yes |
| Planning | Planner sub-agent | Best effort |
| Implementation | Generator sub-agent | Best effort |
| Code review | Evaluator sub-agent | Best effort |
| Build verification | Shell script | Yes |
| Next Issue creation | Claude | Best effort |

## Trigger Rules

```mermaid
flowchart LR
    A{"Who created?"}
    A -->|Human| B{"'@claude' in body?"}
    B -->|Yes| C[Fires]
    B -->|No| D[Skip]
    A -->|"claude[bot]"| E["Script prepends @claude"]
    E --> C

    style C fill:#2d6a4f,color:#fff
    style D fill:#d62828,color:#fff
```

Bot-created Issues without `@claude` are patched automatically by the setup script.

## Branch Strategy

```mermaid
gitGraph
    commit id: "main"
    branch claude/issue-1
    commit id: "Issue #1"
    checkout main
    branch claude/issue-2
    commit id: "Issue #2"
    checkout main
    branch claude/issue-3
    commit id: "Issue #3"
```

Each Issue = one branch = one PR. Review and merge individually.
