# 設計: 保存/読み込み・Undo/Redo・コピー/ペースト

## コンポーネント

### FileIOService（Main Process）

```typescript
interface FileIOService {
  save(filePath: string, data: SerializedProject): Promise<void>;
  load(filePath: string): Promise<SerializedProject>;
  saveBackup(data: SerializedProject): Promise<void>;
  pruneBackups(): Promise<void>;
}

interface SerializedProject {
  version: number;  // スキーマバージョン（現在: 1）
  project: Project;
}
```

### AutoBackupService（Main Process）

```typescript
interface AutoBackupService {
  start(intervalMs: number): void;  // 5分 = 300000ms
  stop(): void;
  readonly maxGenerations: number;  // 36
}
```

### CommandBus / UndoStack

```typescript
interface Command {
  execute(state: ProjectState): ProjectState;
  undo(state: ProjectState): ProjectState;
}

interface CommandBus {
  dispatch(command: Command): void;
  undo(): void;
  redo(): void;
  readonly canUndo: boolean;
  readonly canRedo: boolean;
}
```

UndoStackの動作:
- `dispatch()` → コマンドを実行し、undoStackにpush。redoStackをクリア
- `undo()` → undoStackからpopして`undo()`実行、redoStackにpush
- `redo()` → redoStackからpopして`execute()`実行、undoStackにpush

### ClipboardService

```typescript
interface ClipboardService {
  copy(selection: SelectionState, project: Project): void;
  paste(targetTrackId: string, targetCellId: string): PasteCommand | null;
  readonly clipboardData: ClipboardData | null;
}

interface ClipboardData {
  type: 'cells' | 'bar';
  tracks: Array<{
    trackType: 'melody' | 'text';
    cells: Cell[];
  }>;
}
```

#### ペースト可否判定

```typescript
function canPaste(clipboard: ClipboardData, targetTracks: Track[]): boolean {
  if (clipboard.tracks.length !== targetTracks.length) return false;
  return clipboard.tracks.every((src, i) => src.trackType === targetTracks[i].type);
}
```

## コマンド

```typescript
interface PasteCommand extends Command {
  clipboard: ClipboardData;
  targetTrackId: string;
  targetCellId: string;
}

interface ChangeDefaultSystemBreakCommand extends Command {
  newValue: number;
}

interface ChangeLocalSystemBreakCommand extends Command {
  systemIndex: number;
  newValue: number | null;  // null = LocalSystemBreakを解除
}
```

## エラーハンドリング

| エラー種別 | 対応 |
|---|---|
| 保存先への書き込み権限なし | エラーダイアログを表示。データは失わない |
| 読み込みファイルが存在しない | エラーダイアログを表示。グリッドを変更しない |
| 不正なJSONフォーマット | エラーダイアログを表示。グリッドを変更しない |
| スキーマバージョン不一致 | バージョン不一致の旨を表示。マイグレーション可能な場合は変換を試みる |
| バックアップ保存失敗 | サイレントに失敗。次回バックアップ時に再試行 |
| Undoスタックが空 | 操作を無視 |
| Redoスタックが空 | 操作を無視 |
| TrackType不一致のペースト | 操作を無視 |

## 正確性プロパティ

### プロパティ12: シリアライズのラウンドトリップ

任意の有効なProjectオブジェクトに対して、シリアライズ→デシリアライズの結果は元のProjectと等価。

### プロパティ13: 不正ファイルのデシリアライズエラー

任意の不正なJSON文字列に対して、デシリアライザはエラーを返し、グリッドの状態を変更しない。

### プロパティ14: バックアップ世代数の上限

任意の保存操作の繰り返しに対して、バックアップファイル数は常に36以下。

### プロパティ15: Undo/Redoのラウンドトリップ

任意の操作シーケンスに対して、全操作をUndoした後にRedoすると元の状態に戻る。新たな操作実行後はRedo履歴が空。

### プロパティ16: ペーストのTrackType整合性

ペースト先のTrackタイプの並び順がコピー元と完全に一致する場合のみペーストが実行される。一致しない場合はグリッドの状態は変化しない。

## テスト方針

| 種別 | 対象 |
|---|---|
| ユニットテスト | 不正JSONのエラー例、空スタックへのUndo/Redo、TrackType一致/不一致の具体例、バックアップタイマー動作（モック使用） |
| プロパティテスト | プロパティ12, 13, 14, 15, 16 |
