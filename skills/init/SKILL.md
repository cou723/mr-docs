---
name: init
description: docs/ ディレクトリがまだ整備されていないプロジェクトに、知見キャプチャ用の構造(notes/inbox, knowledge, archive, index.md)とAGENTS.md/CLAUDE.mdへのキャプチャ規則を導入する。「このプロジェクトにmr-docsを導入して」「ドキュメント管理を始めたい」等の依頼で使う。
context: fork
agent: docs-scaffolder
disable-model-invocation: true
---

このプロジェクトに mr-docs の `docs/` 構造(`notes/inbox`, `knowledge`, `archive`, `index.md`)と、AGENTS.md/CLAUDE.mdへのキャプチャ規則を導入してください。既存の構成やファイルは壊さず、不足分だけ冪等に補ってください。
