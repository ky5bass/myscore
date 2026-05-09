# 設計: 音符の長さ（NoteValue）変更

## コンポーネント

### NoteValueService

CurrentNoteValueの管理と、既存セルのNoteValue変更（吸収・分割）を処理する。

```typescript
interface NoteValueService {
  // CurrentNoteValueの取得・設定
  getCurrentNoteValue(trackId: string): NoteValue;
  setCurrentNoteValue(trackId: string, noteValue: NoteValue): void;

  // 既存セルのNoteValue変更（吸収・分割のロジック）
  changeCellNoteValue(trackId: string, cellId: string, newNoteValue: NoteValue): ChangeResult;
}

interface ChangeResult {
  // 吸収されたセルのID一覧（長い音価への変更時）
  absorbedCellIds: string[];
  // 新規追加されたセルのID一覧（短い音価への変更時）
  addedCellIds: string[];
}
```

### 吸収ロジック（長い音価への変更）

```typescript
function absorbCells(
  track: MelodyTrack,
  cellIndex: number,
  oldNoteValue: NoteValue,
  newNoteValue: NoteValue
): MelodyCell[] {
  const oldUnits = NOTE_VALUE_UNITS[oldNoteValue];
  const newUnits = NOTE_VALUE_UNITS[newNoteValue];
  const diffUnits = newUnits - oldUnits;

  // 直後のセルから差分単位分を吸収
  let absorbed = 0;
  const result: MelodyCell[] = [];
  let i = cellIndex + 1;
  while (absorbed < diffUnits && i < track.cells.length) {
    absorbed += NOTE_VALUE_UNITS[track.cells[i].noteValue];
    result.push(track.cells[i]);
    i++;
  }
  return result;
}
```

### 分割ロジック（短い音価への変更）

```typescript
function splitCell(
  cell: MelodyCell,
  newNoteValue: NoteValue
): MelodyCell[] {
  const oldUnits = NOTE_VALUE_UNITS[cell.noteValue];
  const newUnits = NOTE_VALUE_UNITS[newNoteValue];
  const count = Math.round(oldUnits / newUnits);

  // 元のセルを新しいNoteValueに変更し、残りは未記入セルを追加
  const result: MelodyCell[] = [
    { ...cell, noteValue: newNoteValue },
  ];
  for (let i = 1; i < count; i++) {
    result.push({
      id: generateId(),
      noteValue: newNoteValue,
      noteName: null,
      octave: null,
    });
  }
  return result;
}
```

## コマンド

```typescript
interface ChangeNoteValueCommand extends Command {
  trackId: string;
  cellId: string;
  newNoteValue: NoteValue;
}
```

## 正確性プロパティ

### プロパティ10: NoteValue変更後のセル総量保存

任意のトラックとNoteValue変更操作に対して、変更前後でトラック全体の「音価単位の合計」は変化しない。変更対象セルのNoteNameは変更前後で同一。

### プロパティ11: CurrentNoteValueの新規セルへの適用

任意のCurrentNoteValue設定後に追加される新規セルは、そのCurrentNoteValueを持つ。

## テスト方針

| 種別 | 対象 |
|---|---|
| ユニットテスト | 吸収・分割の具体例、入力モード中の変更拒否 |
| プロパティテスト | プロパティ10, 11 |
