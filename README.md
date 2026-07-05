# mr-docs

LLMコーディング時代の「調査・設計対話で得た知見が、明示的に保存しない限り消えてしまう」問題に対応するClaude Codeプラグイン。

## 設計思想

ドキュメント管理を「キャプチャ」「精査」「配布」の3層に分ける。

- **キャプチャ層**: 調査結果を無秩序に受け皿(`docs/notes/inbox/`)へ投入する。分類コストをここで払わない。
- **精査層**: inboxを`doc-gardener`サブエージェントが精査し、`docs/knowledge/`への統合・昇格や`docs/archive/`への退避、陳腐化検知を行う。破壊的操作(マージ・退避)を含むため、手動実行かつコミットは人間に委ねる設計。
- **配布層**: このプラグイン自体。

### スキルとエージェントの役割分担

各スキル(`SKILL.md`)は「何を・なぜすべきか」を判断するオーケストレーションだけに徹し、具体的な手続き(ファイル命名・frontmatter・冪等性チェックなど)は専用のサブエージェントに委任する。

- `init` → `docs-scaffolder`: 会話文脈に依存しない機械的なセットアップなので `context: fork` でエージェントに丸ごと委任する。
- `capture` → `note-capturer`: 「何を保存すべきか」の判断には会話文脈が必要なため fork せず、スキル自身が要約を作ってからエージェントに渡す(エージェントは会話履歴を見られないため)。
- `garden` → `doc-gardener`: 大量に読んで判断する処理なので `context: fork` でメインの会話コンテキストを消費しないようにする。

## 提供スキル

| スキル | 役割 | 実処理を担うエージェント |
| :--- | :--- | :--- |
| `/mr-docs:init` | プロジェクトに `docs/notes/inbox`, `docs/knowledge`, `docs/archive`, `docs/index.md` の構造とAGENTS.md/CLAUDE.mdへのキャプチャ規則を導入する。プロジェクトごとに最初の1回だけ実行する | `agents/docs-scaffolder.md` |
| `/mr-docs:capture` | 調査・設計検討の結果を要約し、`docs/notes/inbox/` への保存を依頼する。手動でも呼べるが、まとまった調査を行った回答の最後にClaudeが自発的に呼ぶことを想定している | `agents/note-capturer.md` |
| `/mr-docs:garden` | inboxの精査(マージ/昇格/退避)・knowledgeの陳腐化検知・`docs/index.md`の再生成を行う。破壊的操作を含むため手動実行のみで、`git commit`はしない | `agents/doc-gardener.md` |

## 使い方

1. 対象プロジェクトでこのプラグインを有効化する。
2. プロジェクトルートで `/mr-docs:init` を実行し、`docs/` 構造とAGENTS.mdのキャプチャ規則を導入する。
3. 以降、Claudeが調査・比較・設計検討を行うと、`/mr-docs:capture` により自動的に `docs/notes/inbox/` へ知見が蓄積される。
4. inboxが溜まってきたら `/mr-docs:garden` を実行して精査する。変更は`git diff`で確認してからコミットする。

## 今後

- 精査タイミングの半自動化(SessionStartフックでinbox滞留件数を通知)
