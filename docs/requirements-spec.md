# myscore 全体仕様書

## 概要

myscore は、MacBook向けの軽量デスクトップ採譜アプリである。格子状のグリッドに音名を入力することで、音楽的アイデアを素早く記録することを目的とする。音声再生機能は持たず、完全無音で動作する。1曲ごとに1プロジェクトを作成して管理する。

## 設計方針

- **パフォーマンス最優先**: 100,000セルを超える場合でも全操作を16ms以内に完了させる
- **IME非依存**: ローマ字→カタカナ変換をアプリ内で独自処理し、OSのIMEを介さない
- **Canvasベースレンダリング**: DOM操作では100,000セルのパフォーマンスを維持できないため、HTML5 Canvas（または OffscreenCanvas）を採用する
- **Electronによる実装**: macOSネイティブAPIへのアクセスとWeb技術の両立のため Electron を採用する

## 技術スタック

| 層 | 技術 |
|---|---|
| シェル | Electron (macOS) |
| レンダリング | HTML5 Canvas / OffscreenCanvas |
| ロジック | TypeScript |
| 状態管理 | イミュータブルなコマンドパターン（Undo/Redo対応） |
| 永続化 | JSON ファイル（ローカルファイルシステム） |
| テスト | Vitest + fast-check（プロパティベーステスト） |

## アーキテクチャ

```
Electron Main Process
├── FileIO Service
├── AutoBackup Service
└── IPC Bridge

Electron Renderer Process
├── UI Layer
│   ├── Canvas Renderer
│   ├── Input Handler
│   └── IME Engine（ローマ字→カタカナ）
├── Application Layer
│   ├── App Controller
│   └── Command Bus
└── Domain Layer
    ├── Project Store
    ├── Grid Model
    ├── Undo/Redo Stack
    └── Clipboard
```

### レイヤー責務

| レイヤー | 責務 |
|---|---|
| UI Layer | Canvas描画・キーボード/マウスイベント受信・IME変換 |
| Application Layer | コマンドのディスパッチ・Undo/Redo管理 |
| Domain Layer | グリッドデータモデル・ビジネスロジック |
| Main Process | ファイルI/O・自動バックアップ・OS連携 |

### コマンドパターンによるUndo/Redo

全ての状態変更はコマンドオブジェクトとして表現する。各コマンドは `execute()` と `undo()` を持ち、UndoStackで管理する。

## 用語集

- **App**: myscoreアプリケーション全体
- **Project**: 1曲分の採譜データの単位
- **Grid**: 格子状に並んだセルの集合体（プロジェクト全体のキャンバス）
- **Cell**: 音符1つ分を表す最小単位のマス
- **NoteValue**: 音符の種類（全音符〜32分音符、各種3連符）。セルの横幅に対応する
- **CurrentNoteValue**: 現在選択中の音価モード
- **Pitch**: NoteNameとオクターブ番号の組み合わせで決まる絶対的な音の高さ
- **NoteName**: 音名。12音（ド・ノ・レ・ネ・ミ・ハ・バ・ソ・ゾ・ラ・ジ・シ）+ 特殊音3種（x・ー・ッ）の計15種類
- **SpecialNoteName**: 音階のない特殊音（x・ー・ッ）
- **PrecedingNoteName**: あるCellより前にあるCellのうち、最後に登場するNoteName
- **VerticalOffset**: Pitchに応じたセルの縦方向の表示位置
- **OctaveShift**: ドラッグ操作によるオクターブの上下変更
- **Track**: グリッド内の1行に対応する入力レーン
- **MelodyTrack**: 1セルに1つのNoteNameを入力でき、Pitchに応じて縦方向に表示位置がずれるトラック
- **TextTrack**: 1セルに複数文字のテキストを入力でき、縦方向の表示位置がずれないトラック
- **Bar**: 小節。拍子に基づいた時間軸上の区切り単位
- **BarLine**: 小節の境界を示す太線
- **SystemBreak**: 複数小節をまとめて折り返す単位
- **DefaultSystemBreak**: プロジェクト全体のデフォルトの折り返し小節数（初期値: 4小節）
- **LocalSystemBreak**: 特定の段に局所的に設定された折り返し小節数

## データモデル

```typescript
// 音名（15種類）
type NoteName =
  | 'ド' | 'ノ' | 'レ' | 'ネ' | 'ミ' | 'ハ' | 'バ'
  | 'ソ' | 'ゾ' | 'ラ' | 'ジ' | 'シ'
  | 'x' | 'ー' | 'ッ';

type SpecialNoteName = 'x' | 'ー' | 'ッ';

// 音価
type NoteValue =
  | 'whole' | 'half' | 'quarter' | 'eighth'
  | 'sixteenth' | 'thirty_second'
  | 'half_triplet' | 'quarter_triplet'
  | 'eighth_triplet' | 'sixteenth_triplet';

// 音価の基準単位（32分音符=1）に対する相対幅
const NOTE_VALUE_UNITS: Record<NoteValue, number> = {
  whole: 32, half: 16, quarter: 8, eighth: 4,
  sixteenth: 2, thirty_second: 1,
  half_triplet: 32 / 3, quarter_triplet: 16 / 3,
  eighth_triplet: 8 / 3, sixteenth_triplet: 4 / 3,
};

// セル
interface MelodyCell {
  id: string;
  noteValue: NoteValue;
  noteName: NoteName | null;
  octave: number | null;  // SpecialNoteNameの場合はnull
}

interface TextCell {
  id: string;
  noteValue: NoteValue;
  text: string;
}

type Cell = MelodyCell | TextCell;

// トラック
interface MelodyTrack {
  id: string;
  type: 'melody';
  name: string;
  currentNoteValue: NoteValue;  // 初期値: 'eighth'
  cells: MelodyCell[];
}

interface TextTrack {
  id: string;
  type: 'text';
  name: string;
  cells: TextCell[];
}

type Track = MelodyTrack | TextTrack;

// 拍子
interface TimeSignature {
  numerator: number;
  denominator: number;
}

// プロジェクト
interface Project {
  id: string;
  name: string;
  timeSignature: TimeSignature;
  defaultSystemBreak: number;  // 初期値: 4
  localSystemBreaks: Record<number, number>;
  tracks: Track[];
  createdAt: string;
  updatedAt: string;
}

// 選択状態
type SelectionState =
  | { type: 'none' }
  | { type: 'cell'; trackId: string; cellId: string }
  | { type: 'cell_range'; trackId: string; startCellId: string; endCellId: string }
  | { type: 'bar'; barIndex: number }
  | { type: 'bar_range'; startBarIndex: number; endBarIndex: number }
  | { type: 'multi_track_range'; tracks: Array<{ trackId: string; startCellId: string; endCellId: string }> };

// クリップボード
interface ClipboardData {
  type: 'cells' | 'bar';
  tracks: Array<{
    trackType: 'melody' | 'text';
    cells: Cell[];
  }>;
}

// シリアライズ形式
interface SerializedProject {
  version: number;
  project: Project;
}
```

## Pitch計算モデル

```typescript
// 対応音域: A1〜C6（計52音）
// 半音ステップ数（C1=0 を基準）
const NOTE_NAME_SEMITONE: Record<Exclude<NoteName, SpecialNoteName>, number> = {
  'ド': 0, 'ノ': 1, 'レ': 2, 'ネ': 3, 'ミ': 4, 'ハ': 5,
  'バ': 6, 'ソ': 7, 'ゾ': 8, 'ラ': 9, 'ジ': 10, 'シ': 11,
};

// VerticalOffset = (最高音の半音数 - 現在の半音数) * SEMITONE_HEIGHT_PX
const SEMITONE_HEIGHT_PX = 4;
```

## テスト戦略

- **ユニットテスト**: Vitest。特定の例・エッジケース・エラー条件を検証
- **プロパティベーステスト**: fast-check。全入力に対して成立すべき普遍的なプロパティを検証
- 各プロパティテストは最低100回のイテレーションを実行
- パフォーマンステストはVitest benchで100,000セルのデータセットを使用

## spec一覧

本プロジェクトは以下の機能specに分割して実装する:

| spec | 概要 |
|---|---|
| grid-layout | グリッド表示・SystemBreak・BarLine・セル幅 |
| melody-input | NoteName入力・IMEエンジン・オクターブ自動決定・VerticalOffset |
| octave-drag | ドラッグによるOctaveShift |
| note-value | NoteValue変更・セル吸収/分割 |
| track-management | 複数トラック管理・TextTrack入力 |
| persistence | 保存/読み込み・Undo/Redo・コピー/ペースト |
