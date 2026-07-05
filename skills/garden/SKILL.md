---
name: garden
description: docs/notes/inbox に溜まった調査メモを精査し、docs/knowledge への統合・昇格やdocs/archiveへの退避、陳腐化した知見の検知、docs/index.mdの再生成を行う。ユーザーが「精査して」「docsを整理して」「gardenして」などと明示的に依頼したときに使う棚卸しコマンド。自動では実行しない。
context: fork
agent: doc-gardener
disable-model-invocation: true
---

docs/ の棚卸しを実行してください。inboxの精査(マージ/昇格/退避)、knowledgeの陳腐化チェック、`docs/index.md`の再生成を行い、実施内容を簡潔に報告してください。`git commit`はしないでください。
