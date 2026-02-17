---
name: issue-management
description: >-
  GitLab / GitHub 両対応の Issue 管理ワークフロー。Issue の作成・着手・ラベル操作・MR/PR 作成を扱う。
  「Issue」「チケット」「タスク」「MR」「マージリクエスト」「PR」「プルリクエスト」「ステータス更新」「バックログ」と言われたときに使う。
---

# Issue 管理ワークフロー

## リポジトリ情報

- **プラットフォーム**: GitLab / GitHub（自動検出）
- **リポジトリ**: `{OWNER}/{PROJECT}`

> ⚠️ プロジェクトに合わせて上記を書き換えてください。

## プラットフォーム検出

Issue 操作の前に、リポジトリのリモート URL からプラットフォームを判定する。

```bash
git remote get-url origin
```

| URL に含まれる文字列 | プラットフォーム | CLI |
|---|---|---|
| `gitlab.com` または `gitlab.` | GitLab | `glab` |
| `github.com` | GitHub | `gh` |

判定できない場合はユーザーに確認する。

---

## Label 体系（共通）

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

---

## Issue テンプレート

Issue 作成時はテンプレートを先に確認し、該当しそうなものがあればその構造に従って description を書く。

| プラットフォーム | テンプレート配置場所 |
|---|---|
| GitLab | `.gitlab/issue_templates/` |
| GitHub | `.github/ISSUE_TEMPLATE/` |

---

## Issue ワークフロー

### Issue 作成

**GitLab:**
```bash
glab issue create -t "タイトル" -l "priority::high,feature-add" -d "description" --no-editor
```

**GitHub:**
```bash
gh issue create -t "タイトル" -l "priority::high,feature-add" -b "description"
```

### Issue 着手時（「この Issue に着手して」と言われたら）

1. Issue の内容を確認し、ユーザーに概要を伝える
   - GitLab: `glab issue view <number>`
   - GitHub: `gh issue view <number>`
2. 現在 Active な Issue を確認
   - GitLab: `glab issue list -l "status::active"`
   - GitHub: `gh issue list -l "status::active"`
   - Active が既にある場合: ユーザーに「#X が Active ですが、切り替えますか？」と確認
   - 切り替える場合: 既存の Active Issue から `status::active` を外す
3. ブランチを作成: `git checkout -b feature/<number>-<short-name>`
4. Issue に `status::active` label を付与し、他の status label を外す

**GitLab:**
```bash
glab api --method PUT "projects/{OWNER}%2F{PROJECT}/issues/<number>" \
  -f add_labels=status::active -f remove_labels=status::wip
```

**GitHub:**
```bash
gh issue edit <number> --add-label "status::active" --remove-label "status::wip"
```

5. Issue の label で実装フローを分岐:

**Vibe フロー**（`mode::vibe` label あり）:
- Issue の description をもとに一気に実装
- コミット → プッシュ → MR/PR 作成まで自動で進める

**Spec フロー**（`mode::spec` label あり）:
- ユーザーと対話しながら進める
- まず Spec（design-draft → requirements → design → tasks）を作成
- tasks.md のタスクを1つずつ実行、各タスク完了後にユーザー確認
- 全タスク完了後に MR/PR 作成

**どちらもなし**:
- ユーザーに「Vibe フローと Spec フローどちらで進めますか？」と確認する

---

## Issue のリンク

### GitLab: Linked Items

```bash
# Issue #20 に #16 を関連付け（デフォルト: relates_to）
glab api --method POST "projects/{OWNER}%2F{PROJECT}/issues/20/links" \
  -f target_project_id={OWNER}/{PROJECT} -f target_issue_iid=16

# ブロック関係を設定
glab api --method POST "projects/{OWNER}%2F{PROJECT}/issues/20/links" \
  -f target_project_id={OWNER}/{PROJECT} -f target_issue_iid=16 -f link_type=blocks
```

### GitHub: Issue 参照

GitHub には API レベルのリンク機能がないため、Issue body 内で `#番号` による参照を使う。
タスクリスト形式で子 Issue を管理する場合:

```markdown
## Related Issues
- #16
- Blocked by #20
```

---

## MR / PR 作成

### GitLab: Merge Request

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

### GitHub: Pull Request

```bash
gh pr create \
  --title "feat(scope): Short description" \
  --body "Closes #<number>
..." \
  --base main \
  --head feature/<number>-<short-name>
```

### MR/PR タイトルの書き方

- **言語**: 英語
- **形式**: `feat(scope): Short description`
- **重要**: 直近のコミットメッセージではなく、Spec 全体を表す簡潔なタイトルにする
- Spec の `requirements.md` や `design.md` を参照してタイトルを決める

### MR/PR description の書き方

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

### MR/PR 作成後

URL を提示して終了。

### マージ後
- `Closes #番号` が description にあれば、マージ時に Issue が自動 Close される（GitLab / GitHub 共通）

---

## ⚠️ マージの禁止事項

- GitLab: `glab mr merge` コマンドは **絶対に実行しない**
- GitHub: `gh pr merge` コマンドは **絶対に実行しない**
- API 経由でのマージも禁止
- マージはユーザーが Web UI から手動で行う
- ユーザーに「マージして」と言われたら、MR/PR の URL を提示して Web UI からのマージを案内する

---

## CLI リファレンス

### GitLab (`glab`)

| コマンド | 用途 |
|---|---|
| `glab issue list` | Issue 一覧 |
| `glab issue view <number>` | Issue 詳細 |
| `glab issue create` | 新規 Issue 作成 |
| `glab issue close <number>` | Issue Close |
| `glab issue update <number>` | Issue 更新 |
| `glab mr create` | MR 作成 |
| `glab mr list` | MR 一覧 |
| `glab mr merge` | **使用禁止** |

### GitHub (`gh`)

| コマンド | 用途 |
|---|---|
| `gh issue list` | Issue 一覧 |
| `gh issue view <number>` | Issue 詳細 |
| `gh issue create` | 新規 Issue 作成 |
| `gh issue close <number>` | Issue Close |
| `gh issue edit <number>` | Issue 更新 |
| `gh pr create` | PR 作成 |
| `gh pr list` | PR 一覧 |
| `gh pr merge` | **使用禁止** |
