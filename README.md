# Claude Continuous Improvement Loop

An infinite self-improving loop powered by [Claude Code Action](https://github.com/anthropics/claude-code-action).
Create one Issue -- Claude takes it from there.

<p align="left">
  <a href="https://github.com/anthropics/claude-code-action"><img src="https://img.shields.io/badge/Claude_Code_Action-191919?style=flat&logo=anthropic&logoColor=white" /></a>
  <a href="https://docs.github.com/en/actions"><img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=githubactions&logoColor=white" /></a>
  <a href="https://www.anthropic.com/claude"><img src="https://img.shields.io/badge/Claude-D4A574?style=flat&logo=anthropic&logoColor=white" /></a>
  <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-green?style=flat" /></a>
</p>
<img width="1365" height="608" alt="スクリーンショット 2026-04-08 18 00 43" src="https://github.com/user-attachments/assets/2d176058-ebc3-44d9-b557-57b1101611b9" />

## How it works

```
Issue -> [Script] Branch + PR -> Planner -> Generator -> Evaluator -> [Script] Build + Auto-merge -> Next Issue -> ...
```

Critical steps run as **shell scripts** (branch/PR creation, build verification, auto-merge).
Planning, implementation, and code review run as **sub-agents**.

Each Issue gets its own branch and PR. Build passes: auto-merged. Build fails: left for manual review.

See [docs/architecture.md](docs/architecture.md) for diagrams.

## Quick start

### 1. Copy files

```bash
cp -r .github <your-repo>/
cp -r .claude <your-repo>/
```

### 2. Install Claude GitHub App

Install [Claude for GitHub](https://github.com/apps/claude) on your repository.

### 3. Add secret

**Settings > Secrets and variables > Actions:**

| Secret | Description |
|--------|-------------|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code OAuth token |

### 4. Enable PR creation

**Settings > Actions > General > Workflow permissions:**
Enable "Allow GitHub Actions to create and approve pull requests".

### 5. Create CLAUDE.md

Use [`CLAUDE.md.example`](CLAUDE.md.example) as a template.
Claude reads this to understand your project.

### 6. Start the loop

Create an Issue with `@claude` at the start of the body.

## File structure

```
.github/workflows/claude.yml       # Workflow (scripts + Claude + sub-agents)
.claude/agents/dev-planner-agent.md # Plans implementation from Issue
.claude/agents/dev-generator-agent.md # Implements one file at a time
.claude/agents/dev-evaluator-agent.md # Verifies implementation quality
CLAUDE.md.example                   # Template for project config
```

## What each step does

| # | Step | Type | Description |
|---|------|------|-------------|
| 1 | Branch + PR | Script | Creates branch and PR. Injects `@claude` if missing. |
| 2 | Planner | Agent | Analyzes issue, outputs implementation plan. |
| 3 | Generator | Agent | Implements one file at a time. |
| 4 | Evaluator | Agent | Reviews implementation. Retries on fail (max 3). |
| 5 | Build + Merge | Script | Runs build. Pass: auto-merge. Fail: comment on PR. |
| 6 | Next Issue | Claude | Creates next improvement issue. Loop continues. |

## Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `model` | `sonnet` | Claude model |
| `timeout_minutes` | `60` | Max runtime per issue |
| `allowed_tools` | See workflow | Tools Claude can use |

## Stopping and restarting

| Action | How |
|--------|-----|
| Stop | Close the latest open Issue |
| Restart | Create a new Issue with `@claude` |

## License

MIT
