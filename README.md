# Claude Continuous Improvement Loop

An infinite self-improving loop powered by [Claude Code Action](https://github.com/anthropics/claude-code-action).  
Create one Issue -- Claude takes it from there.

<p align="left">
  <a href="https://github.com/anthropics/claude-code-action"><img src="https://img.shields.io/badge/Claude_Code_Action-191919?style=flat&logo=anthropic&logoColor=white" /></a>
  <a href="https://docs.github.com/en/actions"><img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=githubactions&logoColor=white" /></a>
  <a href="https://www.anthropic.com/claude"><img src="https://img.shields.io/badge/Claude-D4A574?style=flat&logo=anthropic&logoColor=white" /></a>
  <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-green?style=flat" /></a>
</p>

## How It Works

```
Issue -> Plan -> Implement -> Verify -> Commit -> Next Issue -> ...
```

All improvements accumulate on one branch (`claude/continuous-improvement`) in a single PR.  
The loop runs until you merge the PR to main.

**Key points:**

- **Planner / Generator / Evaluator** agents ensure quality at every step
- **Single PR** -- all changes stack on one branch, easy to review
- **Never stops** -- each Issue triggers the next automatically
- **Bot-proof** -- Issues created by `claude[bot]` fire without needing `@claude` in the body
- **You stay in control** -- merge the PR whenever you're ready

See [docs/architecture.md](docs/architecture.md) for diagrams.

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

### 4. Create a CLAUDE.md

Use [`CLAUDE.md.example`](CLAUDE.md.example) as a starting point.  
This file tells Claude about your project's structure, conventions, and rules.

### 5. Start the loop

Create an Issue with `@claude` in the body:

```
Title: Add dark mode toggle
Body:  Add a dark mode toggle button to the header. @claude
```

## File Structure

```
.github/workflows/
  claude.yml                  # Workflow definition (triggers, Steps 1-6)

.claude/agents/
  dev-planner-agent.md        # Analyze issue, create implementation plan
  dev-generator-agent.md      # Implement one file at a time
  dev-evaluator-agent.md      # Verify implementation quality

docs/
  architecture.md             # Architecture diagrams (Mermaid)

CLAUDE.md.example             # Template for your project config
```

## Configuration

Edit `.github/workflows/claude.yml` to customize:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `model` | `sonnet` | Claude model (`sonnet`, `opus`) |
| `timeout_minutes` | `60` | Max runtime per Issue |
| `allowed_tools` | See workflow | Tools Claude can use |
| `allowed_bots` | `claude[bot]` | Bots allowed to trigger the workflow |

## Stopping and Restarting

| Action | How |
|--------|-----|
| **Stop** | Merge the PR to main |
| **Restart** | Create a new Issue with `@claude` in the body |
| **Pause** | Close the latest open Issue created by claude[bot] |

## License

MIT
