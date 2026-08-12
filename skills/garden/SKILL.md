---
name: garden
description: docs/notes/inbox に溜まった調査メモを精査し、繰り返し参照される判断だけをdocs/principlesへ木構造・must/should/may優先度の原則として昇格させ、docs/配下の管理外ドキュメントの取り込み、既存原則の陳腐化検知、docs/index.mdの再生成を行う。ユーザーが「精査して」「docsを整理して」「gardenして」などと明示的に依頼したときに使う棚卸しコマンド。自動では実行しない。
context: fork
agent: doc-gardener
disable-model-invocation: true
---

docs/ の棚卸しを実行してください。inboxの精査(原則への昇格/統合)、docs/配下の管理外ドキュメントの取り込み、既存原則の陳腐化チェック、`docs/index.md`と`docs/principles/index.md`の再生成を行い、実施内容を簡潔に報告してください。`git commit`はしないでください。
