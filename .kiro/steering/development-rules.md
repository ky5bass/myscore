# 開発ルール

## specファイルの行数制限

- specファイルは各ファイル単体で基本100行程度、最大150行以内に収めること（requirements.md と design.md はそれぞれ独立してカウントする）
- いずれかのファイルが150行を超えそうになった場合は、機能単位での分割を提案すること

## specの構成

- 全体のアーキテクチャ・技術スタック・データモデルは `./docs/requirements-spec.md` に記載する
- 各specの design.md には、そのspec内で必要なインターフェース・コマンド・データ構造のみを自己完結する形で記載する
- 各specの requirements.md には、ユーザーストーリーと受け入れ基準を記載する

## spec一覧

| spec | 概要 |
|---|---|
| grid-layout | グリッド表示・SystemBreak・BarLine・セル幅 |
| melody-input | NoteName入力・IMEエンジン・オクターブ自動決定・VerticalOffset |
| octave-drag | ドラッグによるOctaveShift |
| note-value | NoteValue変更・セル吸収/分割 |
| track-management | 複数トラック管理・TextTrack入力 |
| persistence | 保存/読み込み・Undo/Redo・コピー/ペースト |
