# ecoskills

Claude Code 用の個人スキル集です。このリポジトリはそのまま Claude Code のプラグインマーケットプレイスとして利用できます。

## スキル

| スキル | 概要 |
| --- | --- |
| [orchestrated-planning](skills/orchestrated-planning/SKILL.md) | 大きめのタスクを多角的に分析し、サブエージェントへの設計・調査の委任と敵対的レビューを経て `docs/plans/` に計画書をまとめる |
| [orchestrated-execution](skills/orchestrated-execution/SKILL.md) | 計画書を専用ブランチ・worktree でタスクごとに実装し、各タスクのレビューと最終レビューを経てマージまたは PR 作成まで行う |

## インストール

```
/plugin marketplace add ecofighter/ecoskills
/plugin install ecoskills@ecoskills
```

インストール後は、計画づくりを頼むと `orchestrated-planning` が、`docs/plans/` の計画書の実行を頼むと `orchestrated-execution` が呼び出されます。`/ecoskills:orchestrated-planning` のように名前を指定して呼ぶこともできます。

## ライセンス

[MIT](LICENSE)
