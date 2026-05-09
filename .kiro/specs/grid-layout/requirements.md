# 要件: グリッド表示

## ユーザーストーリー

音楽家として、格子状のグリッドを見ながら音符を入力したい。そうすることで、時間軸と音の並びを直感的に把握できる。

## 受け入れ基準

1. THE App SHALL グリッドを格子状のセル配列として画面に表示する
2. THE Grid SHALL 横軸を時間軸、縦軸をトラック軸として構成する
3. WHEN Appが起動する, THE Grid SHALL 少なくとも1つのMelodyTrackを初期状態として表示する
4. THE Cell SHALL 横幅をNoteValueに対応した相対的なピクセル幅で表示する
5. THE App SHALL 小節の境界にBarLineを太線で表示する
6. THE App SHALL 一定の小節数ごとに全トラックをまとめて折り返し、次の段（System）として下方向に表示する
7. WHEN プロジェクトが新規作成される, THE App SHALL DefaultSystemBreakを4小節に設定する
8. THE App SHALL DefaultSystemBreakをユーザーが変更できる手段を提供する
9. THE App SHALL 特定の段に対してLocalSystemBreakを設定し、その段のみ任意の小節数で折り返せる手段を提供する
10. WHEN 複数のTrackが存在する, THE App SHALL 全てのTrackを同一のSystemBreakで揃えて折り返す
11. THE App SHALL いかなる操作によっても、既存のCellが時間軸（Bar）上の位置をシフトすることを許容しない

## パフォーマンス要件

- THE App SHALL グリッドのセル数が100,000を超える場合でも、16ms以内に再描画を完了する
