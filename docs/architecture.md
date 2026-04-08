# Architecture

## Loop Flow

```mermaid
flowchart LR
    A[Issue] --> B[Plan] --> C[Implement] --> D[Verify]
    D -->|fail| C
    D -->|pass| E[Commit to PR]
    E --> F[Next Issue]
    F --> A

    style A fill:#7c6af7,color:#fff
    style E fill:#2d6a4f,color:#fff
    style F fill:#7c6af7,color:#fff
```

All commits go to one branch (`claude/continuous-improvement`) and one PR.  
The loop stops when you merge the PR to main.

## Trigger Rules

```mermaid
flowchart LR
    A{Who created?}
    A -->|Human| B{@claude in body?}
    B -->|Yes| C[Fires]
    B -->|No| D[Skip]
    A -->|claude bot| C

    style C fill:#2d6a4f,color:#fff
    style D fill:#d62828,color:#fff
```

## Branch Strategy

```mermaid
gitGraph
    commit id: "main"
    branch claude/continuous-improvement
    commit id: "Issue #1"
    commit id: "Issue #2"
    commit id: "Issue #3"
    commit id: "..."
```

One branch. One PR. You merge when ready.
