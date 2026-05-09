# 設計: オクターブのドラッグ変更

## コンポーネント

### DragHandler

ドラッグ操作を検出し、OctaveShiftの仮値を計算してプレビュー表示を制御する。

```typescript
interface DragHandler {
  onMouseDown(event: MouseEvent, hitResult: HitTestResult): void;
  onMouseMove(event: MouseEvent): void;
  onMouseUp(event: MouseEvent): void;
  readonly isDragging: boolean;
  readonly dragPreview: DragPreviewState | null;
}

interface DragPreviewState {
  cellRefs: CellRef[];
  previewOctaveDelta: number;
}
```

### ドラッグ量→オクターブ変換

```typescript
// ドラッグのY方向ピクセル量をオクターブ変化量に変換
const OCTAVE_DRAG_THRESHOLD_PX = SEMITONE_HEIGHT_PX * 12; // 1オクターブ = 12半音分のピクセル

function calculateOctaveDelta(dragDeltaY: number): number {
  // 上方向ドラッグ（負のdeltaY）= オクターブ上昇（正のdelta）
  return -Math.round(dragDeltaY / OCTAVE_DRAG_THRESHOLD_PX);
}
```

### 音域制約の適用

```typescript
function clampOctaveDelta(
  cells: MelodyCell[],
  delta: number
): number {
  // SpecialNoteNameのセルを除外
  const targetCells = cells.filter(c => c.noteName && !isSpecialNoteName(c.noteName));
  if (targetCells.length === 0) return 0;

  // 最も制約が厳しいセルで上限を決定
  let maxUp = Infinity;
  let maxDown = Infinity;
  for (const cell of targetCells) {
    maxUp = Math.min(maxUp, MAX_OCTAVE_FOR_NOTE[cell.noteName!] - cell.octave!);
    maxDown = Math.min(maxDown, cell.octave! - MIN_OCTAVE_FOR_NOTE[cell.noteName!]);
  }

  return Math.max(-maxDown, Math.min(maxUp, delta));
}
```

### プレビュー描画

ドラッグ中は `DragPreviewState` を `RenderState` に渡し、対象セルをグレー色で仮位置に描画する。

```typescript
// CanvasRenderer内のプレビュー描画ロジック
function renderDragPreview(ctx: CanvasRenderingContext2D, preview: DragPreviewState): void {
  for (const cellRef of preview.cellRefs) {
    const baseOffset = getVerticalOffset(cellRef);
    const previewOffset = baseOffset - preview.previewOctaveDelta * 12 * SEMITONE_HEIGHT_PX;
    // グレー色で仮位置に描画
    ctx.fillStyle = '#999999';
    drawCellText(ctx, cellRef, previewOffset);
  }
}
```

## コマンド

```typescript
interface OctaveShiftCommand extends Command {
  trackId: string;
  cellIds: string[];
  delta: number; // 正=上、負=下
}
```

## 正確性プロパティ

### プロパティ9: OctaveShiftの適用と音域制約

任意のNoteName（SpecialNoteNameを除く）のセル群とオクターブ変化量に対して:
1. SpecialNoteNameのセルはスキップされる
2. 複数セル選択時は全セルに同一の変化量が適用される
3. 変化量はCell群の中で音域（A1〜C6）の制約が最も厳しいCellによって上限が決まり、全セルのoctaveは音域内に収まる

## テスト方針

| 種別 | 対象 |
|---|---|
| ユニットテスト | SpecialNoteNameへのドラッグ無効、入力モード中のドラッグ拒否 |
| プロパティテスト | プロパティ9 |
