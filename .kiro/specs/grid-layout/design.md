# 設計: グリッド表示

## コンポーネント

### CanvasRenderer

グリッド全体を Canvas に描画する。仮想スクロールにより表示領域のみを描画する。

```typescript
interface CanvasRenderer {
  render(state: RenderState): void;
  setViewport(x: number, y: number): void;
  hitTest(canvasX: number, canvasY: number): HitTestResult;
}

interface RenderState {
  project: Project;
  selection: SelectionState;
  dragPreview: DragPreviewState | null;
  activeCell: CellRef | null;
}

interface CellRef {
  trackId: string;
  cellId: string;
}

type HitTestResult =
  | { type: 'cell'; trackId: string; cellId: string }
  | { type: 'bar'; barIndex: number }
  | { type: 'none' };
```

### GridLayoutEngine

セルの配置座標・SystemBreak折り返し・BarLine位置を計算する。

```typescript
interface GridLayoutEngine {
  // セルのピクセル幅を算出
  getCellWidth(noteValue: NoteValue): number;
  // 段（System）ごとの小節数を返す
  getSystemBarCounts(project: Project): number[];
  // 指定セルの描画座標を返す
  getCellPosition(trackId: string, cellId: string): { x: number; y: number; width: number; height: number };
  // BarLineの描画X座標一覧を返す
  getBarLinePositions(systemIndex: number): number[];
}
```

### セル幅の計算

```typescript
const BASE_CELL_WIDTH_PX = 24; // 32分音符 = 1単位 = 24px

function getCellWidth(noteValue: NoteValue): number {
  return NOTE_VALUE_UNITS[noteValue] * BASE_CELL_WIDTH_PX;
}
```

### SystemBreak計算ロジック

```typescript
function getSystemBarCounts(project: Project): number[] {
  const result: number[] = [];
  let systemIndex = 0;
  const totalBars = getTotalBarCount(project);
  let barOffset = 0;

  while (barOffset < totalBars) {
    const barCount = project.localSystemBreaks[systemIndex]
      ?? project.defaultSystemBreak;
    result.push(Math.min(barCount, totalBars - barOffset));
    barOffset += barCount;
    systemIndex++;
  }
  return result;
}
```

## 正確性プロパティ

### プロパティ1: NoteValueとセル幅の比例関係

任意の2つのNoteValueに対して、音価の比率とセルのピクセル幅の比率が等しい。

### プロパティ2: SystemBreakによる全トラック統一折り返し

任意のトラック数とSystemBreak値に対して、全てのトラックが同一のSystemBreak位置で折り返される。

### プロパティ3: LocalSystemBreakの局所適用

任意のsystemIndexとLocalSystemBreak値に対して、そのsystemIndexの段のみが指定された小節数で折り返され、他の段はDefaultSystemBreakを使用する。

### プロパティ4: セルの時間軸位置不変性

任意の操作の後も、操作対象外のセルのBar上の位置は変化しない。

## テスト方針

| 種別 | 対象 |
|---|---|
| ユニットテスト | DefaultSystemBreak=4の初期値、各NoteValueの具体的なピクセル幅 |
| プロパティテスト | プロパティ1〜4 |
| ベンチマーク | 100,000セルでの描画時間計測 |
