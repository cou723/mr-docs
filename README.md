# mr-docs

LLMコーディング時代の「調査・設計対話で得た知見が、明示的に保存しない限り消えてしまう」問題に対応するClaude Codeプラグイン。

## 設計思想

ドキュメント管理を「キャプチャ」「精査」「配布」の3層に分ける。

- **キャプチャ層**: 調査結果を無秩序に受け皿(`docs/notes/inbox/`)へ投入する。分類コストをここで払わない。
- **精査層**: inboxを`doc-gardener`サブエージェントが精査し、繰り返し参照される判断だけを`docs/principles/`へ原則として昇格・統合する。原則は「抽象度・汎用度」を軸にした木構造で書き、各ノードに`must`/`should`/`may`の優先度を付ける。この書き方自体を規定する「原則とは何か」のメタ原則は`docs/principles/index.md`に置く。`docs/`配下にmr-docs管理外の既存ドキュメントがあれば、判断ルールに蒸留できるものは取り込み、手順書・リファレンス類は据え置いて目次にだけ載せる。破壊的操作(統合・移動・原則本文の書き換え)を含むため、手動実行かつコミットは人間に委ねる設計。
- **配布層**: 上記の構造とキャプチャ規則を各プロジェクトへ導入・運用する仕組み。このプラグインが担う。

### 何を書くべきか

残す価値があるのは、コードやgit履歴から読み取れない「調査・比較・意思決定の過程とその根拠」である。

- **対象**: Web調査・技術比較・アーキテクチャ検討・設計上の意思決定。試行錯誤の末に「何もしない」と決めた経緯も、却下した選択肢と理由を含めて残す価値が高い(採用案はコードに残るが、失敗した案は記録しない限り消え、同じ検討が繰り返されるため)。逆に、コードやgit履歴から復元できる事実の時系列記録は書かない。
- **形式**: 出来事の時系列ではなく、「何を検討し、なぜその結論に至ったか」という判断の記録に蒸留する。構成は背景・内容(検討した選択肢と理由)・結論と未解決点の3点。
- **基準**: 迷ったら「将来の自分や別のセッションが、同じ調査・試行錯誤を繰り返しそうか」で判断する。繰り返しそうなら書く。

### 原則の書き方(精査層の出力形式)

inboxの生の記録と違い、`docs/principles/`に置く原則は「今後の判断で従うべきルール」に蒸留したものである。すべてinboxから機械的に生まれるわけではなく、`doc-gardener`が繰り返し参照される判断だけを選んで昇格させる。

- **粒度**: 1原則は「`## 原則: <一文で言える主張>`」の見出しと、`**[must]**` / `**[should]**` / `**[may]**` で始まる根拠込みの本文からなる。さらに具体的な手段は「下位の原則」として箇条書きでネストできる。
- **木構造**: 上位ほど抽象的・汎用的(目的)、下位ほど具体的・限定的(現時点の最善の手段)。階層の深さは固定しない。下位は上位の優先度を自動継承しない。
- **優先度**: `must`(逸脱すると事故や目的の毀損に直結する)/ `should`(既定の判断として従うが例外を認める)/ `may`(参考程度)の3段階。
- **バッティングの解決**: 同じ上位原則にぶら下がる原則同士が対立する場合はその上位原則自身のタイブレーカーに従い、異なる上位原則同士が対立する場合は優先度レベル(mustが勝つ)で比較する。両方mustなら機械的には決めず、推測せず調査してから個別に判断する。
- **この形式自体を規定するメタ原則**は `/mr-docs:init` が生成する `docs/principles/index.md` に書かれており、プロジェクトごとに編集して育てていく。

### スキルとエージェントの役割分担

各スキル(`SKILL.md`)は「何を・なぜすべきか」を判断するオーケストレーションだけに徹し、具体的な手続き(ファイル命名・frontmatter・冪等性チェックなど)は専用のサブエージェントに委任する。

- `init` → `docs-scaffolder`: 会話文脈に依存しない機械的なセットアップなので `context: fork` でエージェントに丸ごと委任する。
- `capture` → `note-capturer`: 「何を保存すべきか」の判断には会話文脈が必要なため fork せず、スキル自身が要約を作ってからエージェントに渡す(エージェントは会話履歴を見られないため)。
- `garden` → `doc-gardener`: 大量に読んで判断する処理なので `context: fork` でメインの会話コンテキストを消費しないようにする。

## 提供スキル

| スキル | 役割 | 実処理を担うエージェント |
| :--- | :--- | :--- |
| `/mr-docs:init` | プロジェクトに `docs/notes/inbox`, `docs/principles`, `docs/index.md` の構造とAGENTS.md/CLAUDE.mdへのキャプチャ規則を導入する。プロジェクトごとに最初の1回だけ実行する | `agents/docs-scaffolder.md` |
| `/mr-docs:capture` | 調査・設計検討の結果を要約し、`docs/notes/inbox/` への保存を依頼する。手動でも呼べるが、まとまった調査を行った回答の最後にClaudeが自発的に呼ぶことを想定している | `agents/note-capturer.md` |
| `/mr-docs:garden` | inboxの精査(原則への昇格・統合)・管理外ドキュメントの取り込み・既存原則の陳腐化検知・`docs/index.md`と`docs/principles/index.md`の再生成を行う。破壊的操作を含むため手動実行のみで、`git commit`はしない | `agents/doc-gardener.md` |

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
4. inboxが溜まってきたら `/mr-docs:garden` を実行して精査する。繰り返し参照される判断が `docs/principles/` へ原則として昇格する。変更は`git diff`で確認してからコミットする。

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
