---
inclusion: always
---

# Design-First Spec ワークフロー

ユーザーが「design-first」と指定した場合、通常の spec ワークフロー（requirements → design → tasks）の前に design-draft ステップを挟む。

## 背景

LLM は先に書いたドキュメントに技術的詳細を集中させ、後に書いたドキュメントでは既出情報を繰り返さない傾向がある。この特性を利用して、design を先に書くことで requirements の抽象度を保つ。

## フロー

1. spec フォルダを作成する（通常の命名規則に従う）
2. `design-draft.md` を作成する
   - コードベースを探索し、現状のアーキテクチャを把握する
   - 技術的な設計判断（型定義、データフロー、モジュール構成、インターフェース）を書く
   - Correctness Properties を含める
3. ユーザーにレビューを依頼する
4. ユーザーが承認したら、通常の spec ワークフローを起動する
   - requirements.md 作成時: design-draft.md の内容を参照し、WHAT（何を満たすべきか）に集中する。HOW（どう実装するか）は書かない
   - design.md 作成時: design-draft.md をベースに洗練する
   - tasks.md 作成時: 通常通り

## requirements.md の書き方ガイド

design-draft.md が存在する場合、requirements.md では以下を守る:

- 受け入れ基準に具体的な型名（`HashMap<String, String>` 等）を書かない
- 受け入れ基準にフィールド定義を書かない
- EARS 構文（WHEN / SHALL / THEN）で振る舞いを記述する
- 技術的な実現方法は design.md に任せる

## design-draft.md の配置

spec フォルダ内に配置する:

```
.kiro/specs/{feature-name}/
├── design-draft.md   ← 最初に作成
├── requirements.md   ← spec ワークフローで作成
├── design.md         ← spec ワークフローで作成（draft をベースに洗練）
└── tasks.md          ← spec ワークフローで作成
```
