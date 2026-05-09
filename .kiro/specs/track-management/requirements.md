# 要件: 複数トラック管理・TextTrack入力

## ユーザーストーリー

音楽家として、メロディ・歌詞・コードなど複数の情報を同一グリッド上で管理したい。そうすることで、楽曲全体の構成を1画面で把握できる。

## 受け入れ基準: トラック管理

1. THE App SHALL MelodyTrackとTextTrackの2種類のトラックをサポートする
2. WHEN ユーザーがトラックを追加する, THE App SHALL MelodyTrackまたはTextTrackのいずれかを選択できる手段を提供する
3. WHEN MelodyTrackが作成される, THE App SHALL CurrentNoteValueを8分音符に設定する
4. THE MelodyTrack SHALL 1つのCellに対してNoteNameの記入または未記入のみを許容する
5. THE TextTrack SHALL 1つのCellに対してNoteName以外のテキストを2文字以上含む入力を許容する
6. WHILE TextTrackのCellにテキストが表示される, THE App SHALL VerticalOffsetを適用せず固定位置に表示する
7. THE App SHALL 複数のTrackを同時に画面上に表示する

## 受け入れ基準: TextTrack入力操作

1. WHEN ユーザーがTextTrackのCellをダブルクリックする, THE App SHALL そのCellを編集モードにしカーソルを表示する
2. WHEN TextTrackのCellが編集モードである, THE App SHALL 任意の文字列（複数文字含む）の入力・編集・削除を受け付ける
3. WHEN 編集モード中にEnterキーを押す, THE App SHALL 入力内容を確定して編集モードを終了し、Cellの移動は発生しない
4. WHEN 編集モード中にTabキーを押す, THE App SHALL 入力内容を確定して次のCellを選択する
5. WHEN 編集モード中にShift+Tabキーを押す, THE App SHALL 入力内容を確定して前のCellを選択する
6. WHEN 編集モード中にEscキーを押す, THE App SHALL 入力内容を破棄して編集前の内容に戻し編集モードを終了する
7. WHEN 編集モードではなくCellが選択されている状態で文字キーを入力する, THE App SHALL そのCellを編集モードにして入力内容で既存の内容を上書きする
8. WHEN 編集モードではなくCellが選択されている状態でEnterキーまたはF2キーを押す, THE App SHALL そのCellを編集モードにして既存の内容を保持したままカーソルを末尾に表示する
9. WHEN 編集モードではなくCellが選択されている状態でDeleteキーを押す, THE App SHALL そのCellの内容を消去して未記入にする
