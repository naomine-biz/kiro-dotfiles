---
inclusion: always
---

# Skill 自動ロード

ユーザーのメッセージに以下のキーワードが含まれる場合、対応する Skill を `discloseContext` で即座にロードすること。
応答を生成する前にロードを完了させる。

| キーワード（いずれか一致） | Skill 名 |
|---|---|
| Issue, チケット, タスク, MR, マージリクエスト, ステータス更新, バックログ | issue-management |
| Spec, 要件, 設計ドキュメント, tasks.md | spec-management |
