# 開発ルール

## specファイルの行数制限

- specファイルは各ファイル単体で基本100行程度、最大150行以内に収めること（requirements.md と design.md はそれぞれ独立してカウントする）
- いずれかのファイルが150行を超えそうになった場合は、機能単位での分割を提案すること

## specの構成

- 全体のアーキテクチャ・技術スタック・データモデルは `./docs/requirements-spec.md` に記載する
- 各specの design.md には、そのspec内で必要なインターフェース・コマンド・データ構造のみを自己完結する形で記載する
- 各specの requirements.md には、ユーザーストーリーと受け入れ基準を記載する

## requirements.mdの運用ルール

- requirements.md のメタ情報セクションに、対応するGitHub IssueのURLを必ず記載すること

## design.mdの運用ルール

- design.md には設計判断の理由を明記すること
- 「〜という考えから〜とした」「〜という要望を受けたため〜とした」のように、なぜその設計にしたかが読み取れる書き方にすること

## task.mdの運用ルール

- 完了したtask.mdは削除すること

## spec一覧

| spec | 概要 | Issue |
|---|---|---|
| grid-layout | グリッド表示・SystemBreak・BarLine・セル幅 | https://github.com/ky5bass/myscore/issues/1 |
| melody-input | NoteName入力・IMEエンジン・オクターブ自動決定・VerticalOffset | https://github.com/ky5bass/myscore/issues/2 |
| octave-drag | ドラッグによるOctaveShift | https://github.com/ky5bass/myscore/issues/3 |
| note-value | NoteValue変更・セル吸収/分割 | https://github.com/ky5bass/myscore/issues/4 |
| track-management | 複数トラック管理・TextTrack入力 | https://github.com/ky5bass/myscore/issues/5 |
| persistence | 保存/読み込み・Undo/Redo・コピー/ペースト | https://github.com/ky5bass/myscore/issues/6 |
