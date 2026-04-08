# Claude Continuous Improvement Loop

Claude Code Action を使った**無限改善ループ**のテンプレートです。
Issue を作成するだけで、Claude が自動的にコードを改善し続けます。

## How it works

```
Issue作成 → Planner(計画) → Generator(実装) → Evaluator(検証)
  → commit & push → 1つのPRに積み上げ → 次のIssue自動作成 → 繰り返し
```

- 全ての改善が **1ブランチ・1PR** にコミットされ続ける
- 開発者がPRをレビューしてマージするまで止まらない
- `claude[bot]` が作成したIssueは自動トリガー（`@claude` 書き忘れでも止まらない）

詳細なアーキテクチャ図は [docs/claude-loop-architecture.md](docs/claude-loop-architecture.md) を参照。

## Setup

### 1. ファイルをコピー

```bash
# このリポジトリの内容をあなたのプロジェクトにコピー
cp -r .github/workflows/claude.yml <your-repo>/.github/workflows/
cp -r .claude/ <your-repo>/.claude/
```

### 2. Claude GitHub App をインストール

[Claude GitHub App](https://github.com/apps/claude) をリポジトリにインストールしてください。

### 3. Secret を設定

GitHub リポジトリの Settings → Secrets and variables → Actions で以下を追加：

| Secret | 説明 |
|--------|------|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code の OAuth トークン |

### 4. CLAUDE.md を作成

`CLAUDE.md.example` を参考に、あなたのプロジェクト用の `CLAUDE.md` を作成してください。
Claude はこのファイルを読んでプロジェクトの規約を理解します。

### 5. ループ開始

Issue を作成して本文に `@claude` を含めるだけ！

```
タイトル: ボタンの色を変更する
本文: ヘッダーのボタンの色を青から緑に変更してください。 @claude
```

## File Structure

```
.github/workflows/
  claude.yml                  # ワークフロー定義（トリガー・Step 1-6）

.claude/agents/
  dev-planner-agent.md        # Step 2: 実装計画を作成
  dev-generator-agent.md      # Step 3: 1ファイルずつコード実装
  dev-evaluator-agent.md      # Step 3: 実装結果を検証

docs/
  claude-loop-architecture.md # アーキテクチャ図（Mermaid）

CLAUDE.md.example             # プロジェクト設定のテンプレート
```

## Customization

### allowed_tools

`claude.yml` の `allowed_tools` をプロジェクトに合わせて変更してください：

```yaml
allowed_tools: "Agent,Read,Write,Edit,MultiEdit,Glob,Grep,Bash(npm run build:*),Bash(npx tsc:*),Bash(git checkout:*),Bash(git add:*),Bash(git commit:*),Bash(git push:*),Bash(gh pr:*),Bash(gh issue:*)"
```

### timeout_minutes

デフォルトは60分。大きなタスクが多い場合は増やしてください。

### model

デフォルトは `sonnet`。`opus` に変更するとより高品質な実装が期待できます。

## Stopping the Loop

PRを main にマージすると、ブランチが削除されてループが停止します。
再開したい場合は、新しい Issue を `@claude` 付きで作成してください。

## License

MIT
