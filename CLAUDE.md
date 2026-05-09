# Claude Code 設定

## 開発ルールの参照先

`.kiro/steering/development-rules.md` を必ず確認してから作業を開始すること。
spec の行数制限・requirements.md への Issue URL 記載・design.md の理由記述形式など、プロジェクト固有のルールが定義されている。

---

## spec の読み方

タスクを受け取ったら、以下の順序で spec を読む:

1. `.kiro/specs/<spec名>/requirements.md` — 何を作るか（ユーザーストーリー・受け入れ基準）
2. `.kiro/specs/<spec名>/design.md` — どう作るか（インターフェース・ロジック・設計の理由）
3. `.kiro/specs/<spec名>/task.md` — 何から実装するか（タスクリスト）

全体のアーキテクチャ・データモデルは `docs/requirements-spec.md` を参照。

---

## タスクの進め方

1. `task.md` のタスクを上から順に実装する
2. 各タスク完了後、`task.md` のチェックボックスを更新する
3. 全タスク完了後、PR を作成する

PR 作成の手順:
- `git checkout -b <spec名>/<作業内容の簡単な説明>`
- コミットメッセージ: 変更の理由を1行で添える（例: `IMEEngineを実装: OSのIMEを介さずローマ字→カタカナ変換`）
- PR 本文に対応 spec と Issue へのリンクを記載する

---

## Git ルール

**ブランチ命名**: `<spec名>/<作業内容>`
例: `melody-input/ime-engine`、`grid-layout/system-break`

**コミット粒度**: タスク単位でコミットする。1タスク = 1コミットを目安にする。

**PR 作成**: 実装完了後に必ず PR を作成する。マージは人間が行う。

---

## やってはいけないこと

- `task.md` を勝手に削除しない（spec 完了処理は Kiro が行う）
- `.kiro/steering/development-rules.md` のルールを無視しない
- spec に記載のないインターフェースや抽象化を勝手に追加しない
- `docs/requirements-spec.md` を直接編集しない（Kiro の spec 完了処理で更新される）
- 仕様の疑問を勝手に解釈して実装しない。不明な点は PR コメントまたは会話で確認を求める
- main ブランチに直接コミットしない
