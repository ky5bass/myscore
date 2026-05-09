# 要件: NoteName入力・オクターブ自動決定・縦方向表示

## ユーザーストーリー

音楽家として、セルにNoteNameを素早く入力し、オクターブが文脈に応じて自動的に決まり、Pitchに応じて上下にずれて表示されることを望む。そうすることで、思い浮かんだメロディをすぐに記録し、音の高低を視覚的に把握できる。

## 受け入れ基準: NoteName入力

1. WHEN ユーザーがMelodyTrackのCellをダブルクリックする, THE App SHALL そのCellをアクティブ状態（入力モード）にする
2. WHEN Cellがアクティブであるとき, THE App SHALL ローマ字かな入力によるキー入力を受け付け、有効なNoteNameが確定した時点で変換キーを押すことなく即座にそのCellに記録する
3. WHEN NoteNameがCellに記録される, THE App SHALL IMEの変換・確定操作を介さず、記録と同時に自動的に次のCellをアクティブにする
4. THE App SHALL MelodyTrackのCellに表示する文字をカタカナに統一する（xは半角で表示する）
5. WHEN 入力されたキーシーケンスがいずれのNoteNameにも対応しない, THE App SHALL その入力を取り消しCellを移動しない
6. WHEN ユーザーがアクティブなCellでスペースキーを押す, THE App SHALL そのCellを未記入のまま次のCellをアクティブにする
7. WHEN 入力モード中にTabキーまたはSpaceキーを押す, THE App SHALL 現在のCellを確定して次のCellをアクティブにする
8. WHEN 入力モード中にShift+Tabキーを押す, THE App SHALL 現在のCellを確定して前のCellをアクティブにする
9. WHEN 入力モード中に矢印キーを押す, THE App SHALL その入力を無視する
10. WHEN 入力モード中にEscキーを押す, THE App SHALL 入力モードを終了しCellの選択状態を維持する
11. WHEN 入力モード中にDeleteキーを押し、かつCellにNoteNameが記入されている, THE App SHALL そのNoteNameを消去して未記入にする
12. WHEN 入力モード中にDeleteキーを押し、かつCellが未記入である, THE App SHALL 1つ前のCellへ移動しその内容を消去して未記入にする
13. WHEN 入力モードではなくCellが選択されている状態でTabキーまたは右矢印キーを押す, THE App SHALL 次のCellを選択する
14. WHEN 入力モードではなくCellが選択されている状態でShift+Tabキーまたは左矢印キーを押す, THE App SHALL 前のCellを選択する
15. WHEN 入力モードではなくCellが選択されている状態でEnterキーまたはF2キーを押す, THE App SHALL そのCellの入力モードに切り替える
16. WHEN 入力モードではなくBarが選択されている状態でTabキーまたは右矢印キーを押す, THE App SHALL 次のBarを選択する
17. WHEN 入力モードではなくBarが選択されている状態でShift+Tabキーまたは左矢印キーを押す, THE App SHALL 前のBarを選択する

## 受け入れ基準: オクターブ自動決定

1. WHEN 上方向プレフィックス（`/`）に続けてNoteNameが入力される, THE App SHALL PrecedingNoteName（SpecialNoteName除く）のPitchより上で最小移動距離のオクターブを決定する
2. WHEN 下方向プレフィックス（`¥`）に続けてNoteNameが入力される, THE App SHALL PrecedingNoteName（SpecialNoteName除く）のPitchより下で最小移動距離のオクターブを決定する
3. WHEN プレフィックスなしでNoteNameが入力される, THE App SHALL PrecedingNoteName（SpecialNoteName除く）のPitchとの移動距離（半音数）が最小になるオクターブを決定する
4. WHEN プレフィックスなしで上下の移動距離が等しい場合, THE App SHALL そのトラック全体のNoteName（SpecialNoteName除く）の平均Pitchとの差が小さくなるオクターブを優先し、それでも決定できない場合はバ4（F#4）に最も近いPitchを選択する
5. WHEN 上記の規則でオクターブが一意に決定できない場合, THE App SHALL バ4（F#4）に最も近いPitchになるオクターブを決定する
6. IF 決定されたオクターブが対応音域（A1〜C6）の範囲外になる, THEN THE App SHALL 音域の上限または下限に丸める

## 受け入れ基準: Pitchに応じた縦方向表示

1. WHEN NoteNameがMelodyTrackのCellに記録される, THE App SHALL オクターブとNoteNameからVerticalOffsetを算出する
2. THE App SHALL VerticalOffsetに基づいてCellのテキストを縦方向にずらして表示する
3. THE App SHALL Pitchが高いほど上方向、低いほど下方向にVerticalOffsetを設定する
4. THE App SHALL 対応音域（A1〜C6）の全Pitchを視覚的に識別できる十分なVerticalOffsetの範囲を確保する
5. WHEN CellのNoteNameがSpecialNoteName（x）である, THE App SHALL VerticalOffsetを適用せず固定位置に表示する
6. WHEN CellのNoteNameが ー または ッ である, THE App SHALL PrecedingNoteName（ー・ッ除く）のVerticalOffsetと同じ位置に表示する
7. IF ー または ッ のPrecedingNoteName（ー・ッ除く）が存在しない, THEN THE App SHALL 固定位置に表示する

## パフォーマンス要件

- WHEN ユーザーがNoteNameを入力する, THE App SHALL 入力から16ms以内に画面を更新する
