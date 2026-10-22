# ecoskills

Claude Code 用の個人スキル集です。このリポジトリはそのまま Claude Code のプラグインマーケットプレイスとして利用できます。

## スキル

| スキル | 概要 |
| --- | --- |
| [orchestrated-planning](skills/orchestrated-planning/SKILL.md) | 大きめのタスクを多角的に分析し、サブエージェントへの設計・調査の委任と敵対的レビューを経て `docs/plans/` に計画書をまとめる |
| [orchestrated-execution](skills/orchestrated-execution/SKILL.md) | 計画書を専用ブランチ・worktree でタスクごとに実装し、各タスクのレビューと最終レビューを経てマージまたは PR 作成まで行う |

## エージェント

スキルが起動するサブエージェントのうち、役割が決まっているものを専用エージェントとして同梱しています。必要なツールだけを持たせることで、`general-purpose` よりコンテキストを小さく保ちます。プラグインなしでスキルだけを使う場合、スキルは `general-purpose` で同じプロンプトを使います。

| エージェント | 用途 |
| --- | --- |
| [ecoskills:researcher](agents/researcher.md) | `orchestrated-planning` の情報収集。評価や推奨を含まない、出典つきの事実をまとめる |
| [ecoskills:reviewer](agents/reviewer.md) | 両スキルの敵対的レビュー。読み取り専用で検証し、指摘をファイルに書く |
| [ecoskills:implementer](agents/implementer.md) | `orchestrated-execution` の実装。計画書の 1 ユニットを 1 コミットで実装し、完了条件の実行結果を報告する |
| [ecoskills:advisor](agents/advisor.md) | `orchestrated-execution` で実装が行き詰まったときの相談役。何も編集せずに原因を診断し、進め方を勧める |

ブラウザや MCP サーバーなど、これらのエージェントにないツールが必要なユニットでは、スキルは `general-purpose` を使います。

## インストール

```
/plugin marketplace add ecofighter/ecoskills
/plugin install ecoskills@ecoskills
```

インストール後は、計画づくりを頼むと `orchestrated-planning` が、`docs/plans/` の計画書の実行を頼むと `orchestrated-execution` が呼び出されます。`/ecoskills:orchestrated-planning` のように名前を指定して呼ぶこともできます。

## ライセンス

[MIT](LICENSE)
