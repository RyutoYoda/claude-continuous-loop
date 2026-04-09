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

## How It Works

```
Issue -> [Script] Branch + PR -> [Planner] -> [Generator] -> [Evaluator] -> [Script] Build Check -> Next Issue -> ...
```

Critical steps (branch/PR creation, build verification) are handled by **shell scripts** -- not prompts.
Intellectual work (planning, implementation, code review) is handled by **sub-agents**.

Each Issue gets its own branch and PR (1:1:1). Review and merge individually.

See [docs/architecture.md](docs/architecture.md) for diagrams.

### Sub-Agents

| Agent | Role |
|-------|------|
| **Planner** | Reads the issue, investigates the codebase, outputs an implementation plan |
| **Generator** | Implements one file at a time based on the plan |
| **Evaluator** | Reviews the implementation, returns pass/fail with feedback |

Generator and Evaluator loop until each file passes (max 3 retries).

## Quick Start

### 1. Copy files to your repo

```bash
cp -r .github <your-repo>/
cp -r .claude <your-repo>/
```

### 2. Install the Claude GitHub App

Install [Claude for GitHub](https://github.com/apps/claude) on your repository.

### 3. Add your secret

Go to **Settings > Secrets and variables > Actions** and add:

| Secret | Description |
|--------|-------------|
| `CLAUDE_CODE_OAUTH_TOKEN` | Your Claude Code OAuth token |

### 4. Enable PR creation for Actions

Go to **Settings > Actions > General > Workflow permissions** and enable
"Allow GitHub Actions to create and approve pull requests".

### 5. Create a CLAUDE.md

Use [`CLAUDE.md.example`](CLAUDE.md.example) as a starting point.
This file tells Claude about your project's structure, conventions, and rules.

### 6. Start the loop

Create an Issue with `@claude` at the start of the body:

```
Title: Add dark mode toggle
Body:
@claude

Add a dark mode toggle button to the header.
```

## File Structure

```
.github/workflows/
  claude.yml                  # Workflow: scripts + Claude action + sub-agents

.claude/agents/
  dev-planner-agent.md        # Analyze issue, create implementation plan
  dev-generator-agent.md      # Implement one file at a time
  dev-evaluator-agent.md      # Verify implementation quality

docs/
  architecture.md             # Architecture diagrams (Mermaid)

CLAUDE.md.example             # Template for your project config
```

## What the workflow does

| Step | Type | What happens |
|------|------|-------------|
| 1. Checkout | Script | Clone the repo |
| 2. Create branch + PR | Script | Guaranteed. Also injects `@claude` if missing |
| 3. Planner | Sub-agent | Analyzes issue, outputs implementation plan |
| 4. Generator/Evaluator | Sub-agents | Implements and verifies each file |
| 5. Build check (Claude) | Claude | Runs build command, fixes errors |
| 6. Commit + push | Claude | Pushes to the PR branch |
| 7. Next Issue | Claude | Creates the next improvement issue |
| 8. Build verification | Script | Guaranteed. Posts result to PR |

## Configuration

Edit `.github/workflows/claude.yml` to customize:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `model` | `sonnet` | Claude model (`sonnet`, `opus`) |
| `timeout_minutes` | `60` | Max runtime per Issue |
| `allowed_tools` | See workflow | Tools Claude can use |

## Stopping and Restarting

| Action | How |
|--------|-----|
| **Stop** | Close the latest open Issue created by claude[bot] |
| **Restart** | Create a new Issue with `@claude` at the start of the body |

## License

MIT
