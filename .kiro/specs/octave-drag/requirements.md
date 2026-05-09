# 要件: オクターブのドラッグ変更

## ユーザーストーリー

音楽家として、表示されているNoteNameをドラッグしてオクターブを変更したい。そうすることで、キーボードを使わず直感的にオクターブを修正できる。

## 受け入れ基準

1. WHEN Cellが入力モードである, THE App SHALL ドラッグによるOctaveShift操作を受け付けない
2. WHEN ユーザーがMelodyTrackのCellのNoteNameを上方向にドラッグする, THE App SHALL ドラッグ量に応じたオクターブ数だけそのNoteNameのオクターブを上げる
3. WHEN ユーザーがMelodyTrackのCellのNoteNameを下方向にドラッグする, THE App SHALL ドラッグ量に応じたオクターブ数だけそのNoteNameのオクターブを下げる
4. WHEN 複数のCellが選択されている状態でドラッグによるOctaveShift操作を実行する, THE App SHALL 選択中の全CellにOctaveShiftを適用する
5. WHEN 複数のCellが選択されている状態でOctaveShiftを適用する, THE App SHALL 選択中の全Cellに同一のオクターブ変化量を適用し、変化量は選択中のCell群の中で音域（A1〜C6）の制約が最も厳しいCellによって上限が決まる
6. WHEN OctaveShiftを適用する際、一部のCellのNoteNameがSpecialNoteNameである, THE App SHALL そのCellをスキップして残りのCellのみOctaveShiftを適用する
7. WHEN ユーザーがドラッグ操作中（左クリック押下中）である, THE App SHALL OctaveShiftを未確定のまま選択中の全CellをグレーでプレビューしてVerticalOffsetの仮位置を表示する
8. WHEN ユーザーがドラッグ操作を完了する（左クリックを離す）, THE App SHALL OctaveShiftを確定してVerticalOffsetを更新し再描画する

## パフォーマンス要件

- WHEN ユーザーがOctaveShiftのドラッグを行う, THE App SHALL ドラッグ中の各フレームを16ms以内に再描画する
