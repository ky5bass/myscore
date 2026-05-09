# 要件: 音符の長さ（NoteValue）変更

## ユーザーストーリー

音楽家として、入力モードに入る前に音符の長さ（NoteValue）を切り替えておき、様々なリズムを同じグリッド上で表現したい。

## 受け入れ基準

1. THE App SHALL 全音符・2分音符・4分音符・8分音符・16分音符・32分音符・2分3連符・4分3連符・8分3連符・16分3連符をNoteValueとしてサポートする
2. WHEN MelodyTrackが作成される, THE App SHALL CurrentNoteValueを8分音符に設定する
3. WHEN ユーザーがCurrentNoteValueを変更する, THE App SHALL 以降の新規入力セルに変更後のNoteValueを適用する
4. THE App SHALL 入力モード中（Cellがアクティブな状態）はCurrentNoteValueの変更操作を受け付けない
5. WHEN ユーザーが既存のCellのNoteValueを長い音価に変更する（例：16分音符→4分音符）, THE App SHALL 操作したCellの直後から音価の差分に相当する数のCellを吸収して削除し、それより後のCellの時間軸位置を変えない
6. WHEN 吸収されるCellに入力済みのNoteNameが存在する, THE App SHALL そのNoteNameを消去する
7. WHEN ユーザーが既存のCellのNoteValueを短い音価に変更する（例：4分音符→16分音符）, THE App SHALL 操作したCellを分割し、音価の比率から算出した数の新規Cellを直後に追加する
8. WHEN 分割によって追加されたCellが作成される, THE App SHALL そのCellを未記入状態にする
9. WHEN 既存のCellのNoteValueが変更される, THE App SHALL 操作したCellの内容（NoteName）を変更前と同じ状態で保持する
10. WHEN 既存のCellのNoteValueが変更される, THE App SHALL 変更後16ms以内にグリッドレイアウトを更新する
