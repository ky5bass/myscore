# 設計: NoteName入力・オクターブ自動決定・縦方向表示

## コンポーネント

### IMEEngine（ローマ字→カタカナ変換）

OSのIMEを使わず、アプリ内でローマ字入力をカタカナに変換するエンジン。

```typescript
interface IMEEngine {
  feed(key: string): NoteName | null;
  reset(): void;
  readonly pending: string;
  readonly octaveDirectionPrefix: 'up' | 'down' | null;
}
```

#### ローマ字マッピング

| NoteName | ローマ字シーケンス |
|---|---|
| ド | do |
| ノ | no |
| レ | re |
| ネ | ne |
| ミ | mi |
| ハ | ha |
| バ | ba |
| ソ | so |
| ゾ | zo |
| ラ | ra |
| ジ | zi / ji |
| シ | si / shi |
| x | x |
| ー | - (ハイフン) |
| ッ | ltu / ltsu / tu / tsu |

#### プレフィックス

- `/` → 上方向プレフィックス（octaveDirectionPrefix = 'up'）
- `¥` → 下方向プレフィックス（octaveDirectionPrefix = 'down'）

### OctaveResolver（オクターブ自動決定）

```typescript
interface OctaveResolver {
  resolve(
    noteName: Exclude<NoteName, SpecialNoteName>,
    precedingPitch: Pitch | null,
    direction: 'up' | 'down' | null,
    trackAveragePitch: number | null
  ): number; // octave番号
}

interface Pitch {
  noteName: Exclude<NoteName, SpecialNoteName>;
  octave: number;
}
```

#### 決定アルゴリズム

1. PrecedingPitchが存在しない場合 → バ4（F#4）に最も近いオクターブ
2. direction='up' → PrecedingPitchより上で最小移動距離
3. direction='down' → PrecedingPitchより下で最小移動距離
4. direction=null → 最小移動距離。等距離の場合は平均Pitchに近い方。それでも決定不能ならバ4に近い方
5. 結果を音域（A1〜C6）にクランプ

### VerticalOffsetCalculator

```typescript
interface VerticalOffsetCalculator {
  calculate(cell: MelodyCell, precedingCells: MelodyCell[]): number;
}

// 計算式
// offset = (C6の半音数 - cellの半音数) * SEMITONE_HEIGHT_PX
// SpecialNoteName 'x' → 固定値 0
// SpecialNoteName 'ー','ッ' → PrecedingNoteName（ー・ッ除く）のoffset
const SEMITONE_HEIGHT_PX = 4;
```

### InputHandler（入力モード管理）

```typescript
interface InputHandler {
  // セルのアクティブ化
  activateCell(trackId: string, cellId: string): void;
  // キー入力処理
  handleKeyDown(event: KeyboardEvent): void;
  // 入力モード終了
  deactivate(): void;

  readonly isActive: boolean;
  readonly activeCell: CellRef | null;
}
```

## コマンド

```typescript
interface SetNoteNameCommand extends Command {
  trackId: string;
  cellId: string;
  newNoteName: NoteName | null;
  newOctave: number | null;
}
```

## 正確性プロパティ

### プロパティ5: IMEエンジンのローマ字→NoteName変換

任意の有効なローマ字シーケンスに対して、IMEエンジンは対応するNoteNameを返す。無効なシーケンスにはnullを返す。

### プロパティ6: NoteName表示のカタカナ統一

任意のNoteName（12音）に対して、表示文字はカタカナ。xは半角。

### プロパティ7: VerticalOffsetの単調性

任意の2つのPitchに対して、高い方のVerticalOffsetは低い方より小さい（上方向）。音域内の全Pitchは互いに異なるVerticalOffsetを持つ。

### プロパティ8: SpecialNoteName（ー・ッ）のVerticalOffset継承

ー または ッ のVerticalOffsetは、PrecedingNoteName（ー・ッ除く）のVerticalOffsetと等しい。

### プロパティ17: オクターブ自動決定の音域クランプ

任意のNoteName入力に対して、オクターブ自動決定の結果は常にA1〜C6の範囲内。

### プロパティ18: プレフィックスなし入力の最小移動距離

決定されたオクターブによるPitchとPrecedingNoteNameのPitchとの移動距離は、音域内の他のいかなるオクターブによる移動距離以下。

### プロパティ19: 上方向プレフィックスの方向保証

上方向プレフィックス付き入力の結果PitchはPrecedingNoteNameのPitchより高い。

### プロパティ20: 下方向プレフィックスの方向保証

下方向プレフィックス付き入力の結果PitchはPrecedingNoteNameのPitchより低い。

## エラーハンドリング

| エラー種別 | 対応 |
|---|---|
| 無効なローマ字シーケンス | IMEエンジンがバッファをリセット。セルを移動しない |
| PrecedingNoteNameが存在しない | バ4（F#4）に最も近いオクターブをフォールバックとして使用 |

## テスト方針

| 種別 | 対象 |
|---|---|
| ユニットテスト | 各NoteNameの変換例、無効シーケンス、プレフィックス認識、キーボードナビゲーション |
| プロパティテスト | プロパティ5, 6, 7, 8, 17, 18, 19, 20 |
