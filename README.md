# mr-docs

LLMコーディング時代の「調査・設計対話で得た知見が、明示的に保存しない限り消えてしまう」問題に対応するClaude Codeプラグイン。

## 設計思想

ドキュメント管理を「キャプチャ」「精査」「配布」の3層に分ける。

- **キャプチャ層**: 調査結果を無秩序に受け皿(`docs/notes/inbox/`)へ投入する。分類コストをここで払わない。
- **精査層**: inboxを`doc-gardener`サブエージェントが精査し、`docs/knowledge/`への統合・昇格や`docs/archive/`への退避、陳腐化検知を行う。`docs/`配下にmr-docs管理外の既存ドキュメントがあれば、知見の記録に該当するものは取り込み、手順書・リファレンス類は据え置いて目次にだけ載せる。破壊的操作(マージ・退避・移動)を含むため、手動実行かつコミットは人間に委ねる設計。
- **配布層**: 上記の構造とキャプチャ規則を各プロジェクトへ導入・運用する仕組み。このプラグインが担う。

### 何を書くべきか

残す価値があるのは、コードやgit履歴から読み取れない「調査・比較・意思決定の過程とその根拠」である。

- **対象**: Web調査・技術比較・アーキテクチャ検討・設計上の意思決定。試行錯誤の末に「何もしない」と決めた経緯も、却下した選択肢と理由を含めて残す価値が高い(採用案はコードに残るが、失敗した案は記録しない限り消え、同じ検討が繰り返されるため)。逆に、コードやgit履歴から復元できる事実の時系列記録は書かない。
- **形式**: 出来事の時系列ではなく、「何を検討し、なぜその結論に至ったか」という判断の記録に蒸留する。構成は背景・内容(検討した選択肢と理由)・結論と未解決点の3点。
- **基準**: 迷ったら「将来の自分や別のセッションが、同じ調査・試行錯誤を繰り返しそうか」で判断する。繰り返しそうなら書く。

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
| `/mr-docs:garden` | inboxの精査(マージ/昇格/退避)・管理外ドキュメントの取り込み・knowledgeの陳腐化検知・`docs/index.md`の再生成を行う。破壊的操作を含むため手動実行のみで、`git commit`はしない | `agents/doc-gardener.md` |

## インストール

Claude Codeのセッション内から:

```
/plugin marketplace add cou723/mr-docs
/plugin install mr-docs@mr-docs
```

ターミナルからでも同じことができる(新しいマシンへの導入はこちらが速い)。

```bash
claude plugin marketplace add cou723/mr-docs
claude plugin install mr-docs@mr-docs
```

### 更新

`plugin.json` の `version` を上げたコミットが更新として届く。

```
/plugin marketplace update mr-docs
/plugin update mr-docs
```

`~/.claude/settings.json` に以下を書いておくと、起動時に自動で追従する。

```json
{
  "extraKnownMarketplaces": {
    "mr-docs": {
      "source": { "source": "github", "repo": "cou723/mr-docs" },
      "autoUpdate": true
    }
  }
}
```

## 使い方

1. 対象プロジェクトでこのプラグインを有効化する。
2. プロジェクトルートで `/mr-docs:init` を実行し、`docs/` 構造とAGENTS.mdのキャプチャ規則を導入する。
3. 以降、Claudeが調査・比較・設計検討を行うと、`/mr-docs:capture` により自動的に `docs/notes/inbox/` へ知見が蓄積される。
4. inboxが溜まってきたら `/mr-docs:garden` を実行して精査する。変更は`git diff`で確認してからコミットする。

## 開発

ローカルのチェックアウトをそのまま入れて試せる。

```
/plugin marketplace add ./path/to/mr-docs
/plugin install mr-docs@mr-docs
/reload-plugins
```

検証はCIと同じコマンドをローカルでも実行できる。ルートがマーケットプレイス兼プラグインのため、ディレクトリを渡すと`marketplace.json`しか検証されない。`skills/`と`agents/`のfrontmatterまで検証するには`plugin.json`を直接指定する。

```bash
claude plugin validate . --strict
claude plugin validate .claude-plugin/plugin.json --strict
```

`skills/`・`agents/`・`plugin.json`を変更したら`plugin.json`の`version`を上げる。バージョンが同じままだとClaude Codeはキャッシュを使い続け、更新が届かない。CIでバンプ漏れを検出している。

## 今後

- 精査タイミングの半自動化(SessionStartフックでinbox滞留件数を通知)
