# Claude 無限改善ループ アーキテクチャ

## 全体フロー

```mermaid
flowchart TD
    A["Issue作成"] -->|"GitHub Events:<br/>issues.opened"| B{"誰が作成?"}
    B -->|"人間 → @claude が本文に必要"| C["claude.yml ワークフロー起動"]
    B -->|"claude[bot] → 無条件で発火"| C

    C --> D["Step 1: ブランチ準備"]
    D -->|"既にあれば checkout & pull"| E["claude/continuous-improvement"]
    D -->|"なければ新規作成"| E

    E --> F["Step 2: Planner<br/>dev-planner-agent"]
    F -->|"実装計画 JSON"| G["Step 3: Generator → Evaluator ループ"]

    G --> H["dev-generator-agent<br/>1ファイル実装"]
    H --> I["dev-evaluator-agent<br/>検証"]
    I -->|"pass"| J{次のファイル?}
    I -->|"fail (最大3回)"| H
    J -->|"あり"| H
    J -->|"全完了"| K["Step 4: ビルド検証<br/>npx tsc --noEmit"]

    K --> L["Step 5: commit & push<br/>claude/continuous-improvement"]
    L --> M{"PRは既にある?"}
    M -->|"No"| N["gh pr create<br/>'Claude 継続的改善'"]
    M -->|"Yes"| O["gh pr comment<br/>変更概要を追記"]
    N --> P["Step 6: 次の改善Issue作成<br/>必ず作成・スキップしない"]
    O --> P

    P --> Q["gh issue create"]
    Q -->|"claude[bot]作成 → 自動トリガー"| A

    style A fill:#7c6af7,color:#fff
    style C fill:#1a1a2e,color:#fff
    style E fill:#2d6a4f,color:#fff
    style N fill:#c9a84c,color:#000
    style O fill:#c9a84c,color:#000
    style Q fill:#7c6af7,color:#fff
```

## トリガー条件

```mermaid
flowchart LR
    subgraph "Issue作成時のトリガー判定"
        A{"作成者は?"}
        A -->|"人間"| B{"本文に @claude ある?"}
        B -->|"Yes"| C["ワークフロー発火"]
        B -->|"No"| D["何もしない"]
        A -->|"claude[bot]"| C
    end

    style C fill:#2d6a4f,color:#fff
    style D fill:#d62828,color:#fff
```

- **人間が作ったIssue** → 本文に `@claude` が必要
- **claude[bot]が作ったIssue** → 無条件で自動トリガー（`@claude` 書き忘れでループが止まらない）

## ブランチ戦略 — 1ブランチ・1PR・複数コミット

```mermaid
gitGraph
    commit id: "main (現状)"
    branch claude/continuous-improvement
    commit id: "feat: Issue #12 バッジ追加"
    commit id: "feat: Issue #13 検索機能"
    commit id: "fix: Issue #14 バグ修正"
    commit id: "feat: Issue #15 ..."
    commit id: "feat: Issue #16 ..."
```

```mermaid
flowchart LR
    subgraph "1つのPR (Claude 継続的改善)"
        C1["commit: Issue #12"] --> C2["commit: Issue #13"] --> C3["commit: Issue #14"] --> C4["..."]
    end
    C4 -->|"開発者がレビュー"| M["main にマージ"]

    style M fill:#2d6a4f,color:#fff
```

- 全Issueの実装が **`claude/continuous-improvement` ブランチの1つのPR** にコミットされ続ける
- 各Issue = 1コミット。PRにどんどん積まれていく
- **開発者が確認してマージするまで止まらない**
- マージ後、再開したければ新しいIssueを作成

## 停止条件

```mermaid
flowchart LR
    A["ループ継続"] --> B["常に次のIssueを作成"]
    B --> C["ループ継続"]
    D["オーナーがPRをmainにマージ"] --> E["ループ停止"]
    E --> F["再開したければ新Issueを手動作成"]

    style C fill:#2d6a4f,color:#fff
    style E fill:#d62828,color:#fff
```

- **止まらない**: 改善点がある限り永遠にループし続ける
- **唯一の停止**: オーナーがPRをmainにマージした時点でブランチが消えてループ終了
- **再開**: 新しいIssueを `@claude` 付きで作成すれば再スタート

## 必要なファイル（配布セット）

| ファイル | 役割 | 配布 |
|---------|------|------|
| `.github/workflows/claude.yml` | ワークフロー定義 (トリガー・Step 1-6) | 必須 |
| `.claude/agents/dev-planner-agent.md` | 実装計画を作成 | 必須 |
| `.claude/agents/dev-generator-agent.md` | 1ファイルずつコード実装 | 必須 |
| `.claude/agents/dev-evaluator-agent.md` | 実装結果を検証 | 必須 |
| `CLAUDE.md` | プロジェクト固有の規約 | 各自作成 |

## セットアップ（他のリポジトリで使う場合）

1. 上記4ファイルをリポジトリにコピー
2. [Claude GitHub App](https://github.com/apps/claude) をリポジトリにインストール
3. GitHub Secrets に `CLAUDE_CODE_OAUTH_TOKEN` を設定
4. `CLAUDE.md` をプロジェクトに合わせて作成
5. Issue を `@claude` 付きで作成 → ループ開始
