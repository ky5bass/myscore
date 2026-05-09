# 要件: 保存/読み込み・Undo/Redo・コピー/ペースト

## ユーザーストーリー

音楽家として、作成した採譜データを保存・読み込みし、操作を取り消したりやり直したり、入力済みのセルをコピー&ペーストしたい。そうすることで、後から編集を再開でき、入力ミスを素早く修正しながら試行錯誤できる。

## 受け入れ基準: データの保存と読み込み

1. WHEN ユーザーが保存操作を実行する, THE App SHALL グリッドデータをファイルにシリアライズして保存する
2. WHEN ユーザーが読み込み操作を実行する, THE App SHALL ファイルからデータをデシリアライズしてグリッドを復元する
3. THE Serializer SHALL グリッドデータをJSON形式に変換する
4. THE Deserializer SHALL JSON形式のファイルをグリッドデータに変換する
5. IF 読み込んだファイルが不正な形式である, THEN THE App SHALL エラーメッセージを表示してグリッドを変更しない
6. FOR ALL 有効なグリッドデータに対して, 保存→読み込みの結果は元のデータと等価である（ラウンドトリップ特性）
7. THE App SHALL 未保存の状態であっても5分ごとに自動バックアップとして保存する
8. THE App SHALL バックアップを最大36世代まで保持し、超過分は最も古いものを削除する

## 受け入れ基準: Undo/Redo

1. WHEN ユーザーがUndo操作を実行する, THE App SHALL 直前の操作を取り消してグリッドを1つ前の状態に戻す
2. WHEN ユーザーがRedo操作を実行する, THE App SHALL 直前のUndoを取り消してグリッドを1つ後の状態に戻す
3. THE App SHALL NoteNameの入力・変更・削除をUndo/Redoの対象として記録する
4. THE App SHALL OctaveShiftをUndo/Redoの対象として記録する
5. THE App SHALL NoteValueの変更をUndo/Redoの対象として記録する
6. THE App SHALL トラックの追加・削除をUndo/Redoの対象として記録する
7. THE App SHALL TextTrackのテキスト入力・変更・削除をUndo/Redoの対象として記録する
8. THE App SHALL コピー・ペースト・複製操作をUndo/Redoの対象として記録する
9. THE App SHALL SystemBreakの変更をUndo/Redoの対象として記録する
10. IF Undo可能な操作が存在しない, THEN THE App SHALL Undo操作を無視する
11. IF Redo可能な操作が存在しない, THEN THE App SHALL Redo操作を無視する
12. WHEN 新たな操作が実行される, THE App SHALL それ以降のRedo履歴を破棄する
13. WHEN Undo/Redo操作が実行される, THE App SHALL 16ms以内に画面を更新する

## 受け入れ基準: コピー・ペースト・複製

1. THE App SHALL 単一CellのコピーをCmd+Cキーで提供する
2. THE App SHALL 複数の連続するCellを範囲選択してまとめてコピーする操作を提供する
3. THE App SHALL Bar単位でのコピー操作を提供する（全Track対象、ペースト先も全Track上書き）
4. WHEN コピー後にペースト先を選択してCmd+Vキーを押す, THE App SHALL コピー内容をペースト先に上書きする（時間軸はシフトしない）
5. WHEN ペースト対象の範囲がペースト先の起点から超える, THE App SHALL はみ出した分も後続のCellに上書きする
6. THE App SHALL トラックを横断した範囲選択は、位置として連続しているTrack間のみ許容する
7. WHEN 複数Trackにまたがるコピーを行う, THE App SHALL コピー元のTrackタイプの並び順を記録する
8. WHEN ペーストを実行する, THE App SHALL ペースト先のTrackタイプの並び順がコピー元と完全に一致する場合のみ許容する
9. IF ペースト先のTrackタイプの並び順がコピー元と一致しない, THEN THE App SHALL ペースト操作を無視する
10. THE App SHALL MelodyTrackのCell範囲をTextTrackへペーストする操作を許容しない（逆も同様）
11. THE App SHALL トラックの複製操作を提供する
12. WHEN ユーザーがトラックを複製する, THE App SHALL 同一内容の新しいTrackをそのTrackの直下に追加する
