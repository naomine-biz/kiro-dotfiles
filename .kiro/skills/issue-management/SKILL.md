---
name: issue-management
description: >-
  GitLab Issue の管理ワークフロー。Issue の作成・更新、MR との連携、
  MR タイトル・description の書き方を扱う。「Issue を作って」「MR を作成」「ステータス更新」と言われたときに使う。
---

# Issue 管理ワークフロー

## GitLab リポジトリ

- **リポジトリ**: `{OWNER}/{PROJECT}`
- **URL**: https://gitlab.com/{OWNER}/{PROJECT}

> ⚠️ プロジェクトに合わせて上記を書き換えてください。

## Label 体系

### Priority
| Label | 説明 |
|-------|------|
| `priority::high` | 高優先度 |
| `priority::medium` | 中優先度 |
| `priority::low` | 低優先度 |

### Type
| Label | 用途 |
|-------|------|
| `type::task` | スコープ明確、そのまま実装に入れるもの |
| `type::story` | 調査・設計が必要、複数タスクに分解される可能性があるもの |

### ステータス
| Label | 説明 |
|-------|------|
| (なし) | Backlog / Open |
| `status::wip` | 長期的に進行中（調査、設計、依存待ち等） |
| `status::active` | ブランチ切ってコーディング中（短期、今やってる） |

Board「Workflow」で Open → WIP → Active → Closed のカンバン管理が可能。
`status::active` は常に 1〜2 個が目安。

### カテゴリ
| Label | 説明 |
|-------|------|
| `feature-add` | 新機能追加 |
| `feature-remove` | 機能削除 |
| `security-gap` | セキュリティ設計上のギャップ |

### モード
| Label | 説明 |
|-------|------|
| `mode::vibe` | Vibe フロー（一気に実装） |
| `mode::spec` | Spec フロー（対話しながら段階的に実装） |

## Assignee（担当者）

- 一人プロジェクトの場合は指定不要

## Issue のリンク

### Linked Items（関連付け）

```bash
# Issue #20 に #16 を関連付け（デフォルト: relates_to）
glab api --method POST "projects/{OWNER}%2F{PROJECT}/issues/20/links" \
  -f target_project_id={OWNER}/{PROJECT} -f target_issue_iid=16

# ブロック関係を設定
glab api --method POST "projects/{OWNER}%2F{PROJECT}/issues/20/links" \
  -f target_project_id={OWNER}/{PROJECT} -f target_issue_iid=16 -f link_type=blocks
```

## Issue テンプレート

Issue 作成時は `.gitlab/issue_templates/` を先に確認し、該当しそうなテンプレートがあればその構造に従って description を書く。

## Issue ワークフロー

### Issue 作成
```bash
glab issue create -t "タイトル" -l "priority::high,feature-add" -d "description" --no-editor
```

### Issue 着手時（「このIssueに着手して」と言われたら）

1. `glab issue view <number>` で Issue の内容を確認し、ユーザーに概要を伝える
2. `glab issue list -l "status::active"` で現在 Active な Issue を確認
   - Active が既にある場合: ユーザーに「#X が Active ですが、切り替えますか？」と確認
   - 切り替える場合: 既存の Active Issue から `status::active` を外す
3. ブランチを作成: `git checkout -b feature/<number>-<short-name>`
4. Issue に `status::active` label を付与:
   ```bash
   glab api --method PUT "projects/{OWNER}%2F{PROJECT}/issues/<number>" \
     -f add_labels=status::active -f remove_labels=status::wip
   ```
5. 他の status label（`status::wip` 等）があれば外す
6. Issue の label で実装フローを分岐:

**Vibe フロー**（`mode::vibe` label あり）:
- Issue の description をもとに一気に実装
- コミット → プッシュ → MR 作成まで自動で進める
- MR description は統一テンプレートで作成

**Spec フロー**（`mode::spec` label あり）:
- ユーザーと対話しながら進める
- まず Spec（design-draft → requirements → design → tasks）を作成
- tasks.md のタスクを1つずつ実行、各タスク完了後にユーザー確認
- 全タスク完了後に MR 作成

**どちらもなし**:
- ユーザーに「Vibe フローと Spec フローどちらで進めますか？」と確認する

### MR 作成時

```bash
glab mr create \
  --title "feat(scope): Short description" \
  --description "Closes #<number>
..." \
  --source-branch feature/<number>-<short-name> \
  --target-branch main \
  --remove-source-branch \
  --no-editor
```

MR 作成後は URL を提示して終了。**マージは Kiro からは行わない。**

⚠️ **MR マージの禁止事項:**
- `glab mr merge` コマンドは **絶対に実行しない**
- `glab api` で merge エンドポイントを叩くことも禁止
- マージはユーザーが GitLab Web UI から手動で行う
- ユーザーに「マージして」と言われたら、MR の URL を提示して Web UI からのマージを案内する

### MR タイトルの書き方

- **言語**: 英語
- **形式**: `feat(scope): Short description`
- **重要**: 直近のコミットメッセージではなく、Spec 全体を表す簡潔なタイトルにする
- Spec の `requirements.md` や `design.md` を参照してタイトルを決める

### MR description の書き方

```markdown
Closes #<issue-number>

## Why (背景)
なぜこの変更が必要か

## What (変更内容)
何を変更したか

## Impact (影響)
影響範囲、破壊的変更の有無
```

- Spec がある場合は `## Spec` セクションに `.kiro/specs/{spec-folder}/` を追記

### MR マージ後
- `Closes #番号` が description にあれば、マージ時に Issue が自動 Close される

## CLI リファレンス

Issue 操作には `glab` CLI を使用：
- `glab issue list` - Issue 一覧
- `glab issue view <number>` - Issue 詳細
- `glab issue create` - 新規 Issue 作成
- `glab issue close <number>` - Issue Close
- `glab issue update <number>` - Issue 更新
- `glab mr create` - MR 作成
- `glab mr list` - MR 一覧
- `glab mr merge` - **使用禁止**（Web UI からマージすること）
