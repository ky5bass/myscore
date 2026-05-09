# 設計: 複数トラック管理・TextTrack入力

## コンポーネント

### TrackManager

トラックの追加・削除・並び替えを管理する。

```typescript
interface TrackManager {
  addTrack(type: 'melody' | 'text', insertIndex: number): Track;
  removeTrack(trackId: string): void;
  duplicateTrack(trackId: string): Track;
  getTrack(trackId: string): Track;
  getAllTracks(): Track[];
}
```

### TextInputHandler

TextTrackの編集モードを管理する。MelodyTrackのInputHandlerとは独立したインタラクションモデル。

```typescript
interface TextInputHandler {
  // 編集モード開始
  startEdit(trackId: string, cellId: string, preserveContent: boolean): void;
  // キー入力処理
  handleKeyDown(event: KeyboardEvent): void;
  // 編集モード終了（確定 or 破棄）
  endEdit(commit: boolean): void;

  readonly isEditing: boolean;
  readonly editingCell: CellRef | null;
  readonly editBuffer: string;
}
```

### TextTrackのセルデータ

```typescript
interface TextCell {
  id: string;
  noteValue: NoteValue;
  text: string;  // 空文字 = 未記入
}

interface TextTrack {
  id: string;
  type: 'text';
  name: string;
  cells: TextCell[];
}
```

## コマンド

```typescript
interface AddTrackCommand extends Command {
  trackType: 'melody' | 'text';
  insertIndex: number;
}

interface RemoveTrackCommand extends Command {
  trackId: string;
}

interface DuplicateTrackCommand extends Command {
  trackId: string;
}

interface SetTextCommand extends Command {
  trackId: string;
  cellId: string;
  newText: string;
}
```

## テスト方針

| 種別 | 対象 |
|---|---|
| ユニットテスト | トラック追加/削除、TextTrack編集モードの動作確認、Esc破棄の検証 |
| プロパティテスト | なし（このspecのロジックはユニットテストで十分カバー可能） |
