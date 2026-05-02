# 要件定義書

## はじめに

本ドキュメントは、MacBook向け独自採譜アプリ「score-notation-app」の要件を定義する。
本アプリは、格子状のセルに音名を入力することでアイデアを素早く記録できる軽量デスクトップアプリである。1曲ごとに1プロジェクトを作成して管理する。
浮かんだ音楽的アイデアをストレスなく即座に書き留めることを最優先とする。
音声再生機能は持たず、完全に無音のアプリとする。

## 用語集

- **App**: score-notation-appアプリケーション全体
- **Project**: 1曲分の採譜データの単位。グリッド・トラック・SystemBreak設定などをまとめて管理する
- **Grid**: 格子状に並んだセルの集合体（プロジェクト全体のキャンバス）
- **Cell**: 音符1つ分を表す最小単位のマス
- **NoteValue**: 音符の種類（全音符・2分音符・4分音符・8分音符・16分音符・32分音符・2分3連符・4分3連符・8分3連符・16分3連符）。セルの横幅に対応する
- **CurrentNoteValue**: 現在選択中の音価モード。トラックへの新規入力時に使用されるNoteValue
- **Pitch**: NoteNameとオクターブ番号の組み合わせで決まる絶対的な音の高さ。SpecialNoteNameにはPitchの概念はない
- **NoteName**: 音名。ド・ノ・レ・ネ・ミ・ハ・バ・ソ・ゾ・ラ・ジ・シ の12音と、音階のない特殊音 x・ー・ッ の計15種類。入力時にオクターブ番号は含まない
- **SpecialNoteName**: 音階のない特殊音（x・ー・ッ）。オクターブの概念を持たない。x はドラッグ不可の固定位置表示。ー と ッ はドラッグ不可で、そのCellより前にあるCellのうち最後に登場する12音またはxのVerticalOffsetに揃えて表示する
- **PrecedingNoteName**: あるCellより前にあるCellのうち、最後に登場するNoteName。除外対象を括弧で明示して使用する（例：PrecedingNoteName（ー・ッ除く）= 12音またはx、PrecedingNoteName（SpecialNoteName除く）= 12音のみ）
- **VerticalOffset**: Pitchに応じたセルの縦方向の表示位置
- **OctaveShift**: ドラッグ操作によるオクターブの上下変更
- **Track**: グリッド内の1行に対応する入力レーン。メロディトラックまたはテキストトラックのいずれか
- **MelodyTrack**: 1セルに1つのNoteNameを入力でき、Pitchに応じて縦方向に表示位置がずれるトラック
- **TextTrack**: 1セルに複数文字のテキストを入力でき、縦方向の表示位置がずれないトラック
- **Bar**: 小節。拍子に基づいた時間軸上の区切り単位
- **BarLine**: 小節の境界を示す太線
- **SystemBreak**: 複数小節をまとめて折り返す単位。1段（System）に含める小節数を定義する
- **DefaultSystemBreak**: プロジェクト全体のデフォルトの折り返し小節数（初期値: 4小節）
- **LocalSystemBreak**: 特定の段に局所的に設定された折り返し小節数

---

## 要件

### 要件1: グリッド表示

**ユーザーストーリー:** 音楽家として、格子状のグリッドを見ながら音符を入力したい。そうすることで、時間軸と音の並びを直感的に把握できる。

#### 受け入れ基準

1. THE App SHALL グリッドを格子状のセル配列として画面に表示する
2. THE Grid SHALL 横軸を時間軸、縦軸をトラック軸として構成する
3. WHEN Appが起動する, THE Grid SHALL 少なくとも1つのMelodyTrackを初期状態として表示する
4. THE Cell SHALL 横幅をNoteValueに対応した相対的なピクセル幅で表示する
5. THE App SHALL 小節の境界にBarLineを太線で表示する
6. THE App SHALL 一定の小節数ごとに全トラックをまとめて折り返し、次の段（System）として下方向に表示する
7. WHEN プロジェクトが新規作成される, THE App SHALL DefaultSystemBreakを4小節に設定する
8. THE App SHALL DefaultSystemBreakをユーザーが変更できる手段を提供する
9. THE App SHALL 特定の段に対してLocalSystemBreakを設定し、その段のみ任意の小節数で折り返せる手段を提供する
10. WHEN 複数のTrackが存在する, THE App SHALL 全てのTrackを同一のSystemBreakで揃えて折り返す（トラックごとに個別に折り返さない）
11. THE App SHALL いかなる操作によっても、既存のCellが時間軸（Bar）上の位置をシフトすることを許容しない（セルは常に時間軸上の固定位置に対応する）

---

### 要件2: NoteName入力

**ユーザーストーリー:** 音楽家として、セルにNoteNameを素早く入力したい。そうすることで、思い浮かんだメロディをすぐに記録できる。

#### 受け入れ基準

1. WHEN ユーザーがMelodyTrackのCellをダブルクリックする, THE App SHALL そのCellをアクティブ状態（入力モード）にする
2. WHEN Cellがアクティブであるとき, THE App SHALL ローマ字かな入力によるキー入力を受け付け、有効なNoteName（ド・ノ・レ・ネ・ミ・ハ・バ・ソ・ゾ・ラ・ジ・シ・x・ー・ッ のいずれか）が確定した時点で変換キーを押すことなく即座にそのCellに記録する
3. WHEN NoteNameがCellに記録される, THE App SHALL IMEの変換・確定操作を介さず、記録と同時に自動的に次のCellをアクティブにする
4. THE App SHALL MelodyTrackのCellに表示する文字をカタカナに統一する（ひらがな入力状態であってもカタカナで表示する。x は半角で表示する）
5. WHEN 入力されたキーシーケンスがいずれのNoteNameにも対応しない, THE App SHALL その入力を取り消しCellを移動しない
6. WHEN ユーザーがアクティブなCellでスペースキーを押す, THE App SHALL そのCellを未記入のまま次のCellをアクティブにする
7. WHEN 入力モード中にユーザーがTabキーまたはSpaceキーを押す, THE App SHALL 現在のCellを確定して次のCellをアクティブにする
8. WHEN 入力モード中にユーザーがShift+Tabキーを押す, THE App SHALL 現在のCellを確定して前のCellをアクティブにする
9. WHEN 入力モード中にユーザーが矢印キーを押す, THE App SHALL その入力を無視する
10. WHEN 入力モード中にユーザーがEscキーを押す, THE App SHALL 入力モードを終了しCellの選択状態を維持する
11. WHEN 入力モード中にユーザーがDeleteキーを押し、かつCellにNoteNameが記入されている, THE App SHALL そのNoteNameを消去して未記入にする（Cellの移動は発生しない）
12. WHEN 入力モード中にユーザーがDeleteキーを押し、かつCellが未記入である, THE App SHALL 1つ前のCellへ移動しその内容を消去して未記入にする
13. WHEN 入力モードではなくCellが選択されている状態でユーザーがTabキーまたは右矢印キーを押す, THE App SHALL 次のCellを選択する
14. WHEN 入力モードではなくCellが選択されている状態でユーザーがShift+Tabキーまたは左矢印キーを押す, THE App SHALL 前のCellを選択する
15. WHEN 入力モードではなくCellが選択されている状態でユーザーがEnterキーまたはF2キーを押す, THE App SHALL そのCellの入力モードに切り替える
16. WHEN 入力モードではなくBarが選択されている状態でユーザーがTabキーまたは右矢印キーを押す, THE App SHALL 次のBarを選択する
17. WHEN 入力モードではなくBarが選択されている状態でユーザーがShift+Tabキーまたは左矢印キーを押す, THE App SHALL 前のBarを選択する

---

### 要件2b: MelodyTrackにおけるオクターブ自動決定

**ユーザーストーリー:** 音楽家として、MelodyTrackのセルにNoteNameを入力するだけでオクターブが文脈に応じて自動的に決まってほしい。そうすることで、オクターブを意識せずに素早く入力できる。

#### 受け入れ基準

1. WHEN 上方向プレフィックス（`/`）に続けてNoteNameが入力される, THE App SHALL PrecedingNoteName（SpecialNoteName除く）のPitchより上で、かつ移動距離が最小になるオクターブを決定する
2. WHEN 下方向プレフィックス（`¥`）に続けてNoteNameが入力される, THE App SHALL PrecedingNoteName（SpecialNoteName除く）のPitchより下で、かつ移動距離が最小になるオクターブを決定する
3. WHEN プレフィックスなしでNoteNameが入力される, THE App SHALL PrecedingNoteName（SpecialNoteName除く）のPitchとの移動距離（半音数）が最小になるオクターブを決定する
4. WHEN プレフィックスなしで上下の移動距離が等しい場合, THE App SHALL そのトラック全体のNoteName（SpecialNoteName除く）の平均Pitchとの差が小さくなるオクターブを優先し、それでも決定できない場合は基準5を適用する
5. WHEN 上記の規則によってオクターブが一意に決定できない場合（移動距離が等しい・平均Pitchとの距離も等しい・PrecedingNoteNameが存在しない等）, THE App SHALL バ4（F#4）に最も近いPitchになるオクターブを決定する
6. IF 決定されたオクターブが対応音域（A1〜C6）の範囲外になる, THEN THE App SHALL 音域の上限または下限のオクターブに丸める

---

### 要件3: Pitchに応じた縦方向表示

**ユーザーストーリー:** 音楽家として、入力したNoteNameがPitchに応じて上下にずれて表示されることを望む。そうすることで、グリッドを見るだけで音の高低を視覚的に把握できる。

#### 受け入れ基準

1. WHEN NoteNameがMelodyTrackのCellに記録される, THE App SHALL 要件2bで決定されたオクターブとNoteNameからVerticalOffsetを算出する
2. THE App SHALL VerticalOffsetに基づいてCellのテキストを縦方向にずらして表示する
3. THE App SHALL Pitchが高いほど上方向、低いほど下方向にVerticalOffsetを設定する
4. THE App SHALL 対応音域（A1〜C6）の全Pitchを視覚的に識別できる十分なVerticalOffsetの範囲を確保する
5. WHEN CellのNoteNameがSpecialNoteName（x）である, THE App SHALL VerticalOffsetを適用せず固定位置に表示する
6. WHEN CellのNoteNameがSpecialNoteName（ー または ッ）である, THE App SHALL PrecedingNoteName（ー・ッ除く）のVerticalOffsetと同じ縦方向位置に表示する
7. IF CellのNoteNameがSpecialNoteName（ー または ッ）であり、PrecedingNoteName（ー・ッ除く）が存在しない, THEN THE App SHALL VerticalOffsetを適用せず固定位置に表示する

---

### 要件4: オクターブのドラッグ変更

**ユーザーストーリー:** 音楽家として、表示されているNoteNameをドラッグしてオクターブを変更したい。そうすることで、キーボードを使わず直感的にオクターブを修正できる。

#### 受け入れ基準

1. WHEN Cellが入力モードである, THE App SHALL ドラッグによるOctaveShift操作を受け付けない
2. WHEN ユーザーがMelodyTrackのCellのNoteNameを上方向にドラッグする, THE App SHALL ドラッグ量に応じたオクターブ数だけそのNoteNameのオクターブを上げる
3. WHEN ユーザーがMelodyTrackのCellのNoteNameを下方向にドラッグする, THE App SHALL ドラッグ量に応じたオクターブ数だけそのNoteNameのオクターブを下げる
4. WHEN 複数のCellが選択されている状態でドラッグによるOctaveShift操作を実行する, THE App SHALL 選択中の全CellにOctaveShiftを適用する
5. WHEN 複数のCellが選択されている状態でOctaveShiftを適用する, THE App SHALL 選択中の全Cellに同一のオクターブ変化量を適用し、変化量は選択中のCell群の中で音域（A1〜C6）の制約が最も厳しいCellによって上限が決まる
6. WHEN OctaveShiftを適用する際、一部のCellのNoteNameがSpecialNoteNameである, THE App SHALL そのCellをスキップして残りのCellのみOctaveShiftを適用する
7. WHEN ユーザーがドラッグ操作中（左クリック押下中）である, THE App SHALL OctaveShiftを未確定のまま選択中の全CellをグレーでプレビューしてVerticalOffsetの仮位置を表示する
8. WHEN ユーザーがドラッグ操作を完了する（左クリックを離す）, THE App SHALL OctaveShiftを確定してVerticalOffsetを更新し再描画する

---

### 要件5: 音符の長さ（NoteValue）変更

**ユーザーストーリー:** 音楽家として、入力モードに入る前に音符の長さ（NoteValue）を切り替えておき、様々なリズムを同じグリッド上で表現したい。

#### 受け入れ基準

1. THE App SHALL 全音符・2分音符・4分音符・8分音符・16分音符・32分音符・2分3連符・4分3連符・8分3連符・16分3連符をNoteValueとしてサポートする
2. WHEN MelodyTrackが作成される, THE App SHALL CurrentNoteValueを8分音符に設定する
3. WHEN ユーザーがCurrentNoteValueを変更する, THE App SHALL 以降の新規入力セルに変更後のNoteValueを適用する
4. THE App SHALL 入力モード中（Cellがアクティブな状態）はCurrentNoteValueの変更操作を受け付けない
5. WHEN ユーザーが既存のCellのNoteValueを長い音価に変更する（例：16分音符→4分音符）, THE App SHALL 操作したCellの直後から音価の差分に相当する数のCellを吸収して削除し、それより後のCellの時間軸位置を変えない
6. WHEN 吸収されるCellに入力済みのNoteNameが存在する, THE App SHALL そのNoteNameを消去する
7. WHEN ユーザーが既存のCellのNoteValueを短い音価に変更する（例：4分音符→16分音符）, THE App SHALL 操作したCellを分割し、音価の比率から算出した数の新規Cellを直後に追加する
8. WHEN 分割によって追加されたCellが作成される, THE App SHALL そのCellを未記入状態にする
9. WHEN 既存のCellのNoteValueが変更される, THE App SHALL 操作したCellの内容（NoteName）を変更前と同じ状態で保持する
10. WHEN 既存のCellのNoteValueが変更される, THE App SHALL 変更後16ミリ秒以内にグリッドレイアウトを更新する

---

### 要件6: 複数トラック管理

**ユーザーストーリー:** 音楽家として、メロディ・歌詞・コードなど複数の情報を同一グリッド上で管理したい。そうすることで、楽曲全体の構成を1画面で把握できる。

#### 受け入れ基準

1. THE App SHALL MelodyTrackとTextTrackの2種類のトラックをサポートする
2. WHEN ユーザーがトラックを追加する, THE App SHALL MelodyTrackまたはTextTrackのいずれかを選択できる手段を提供する
3. WHEN MelodyTrackが作成される, THE App SHALL CurrentNoteValueを8分音符に設定する
4. THE MelodyTrack SHALL 1つのCellに対してNoteNameの記入または未記入のみを許容する
5. THE TextTrack SHALL 1つのCellに対してNoteName以外のテキストを2文字以上含む入力を許容する
6. WHILE TextTrackのCellにテキストが表示される, THE App SHALL VerticalOffsetを適用せず固定位置に表示する
7. THE App SHALL 複数のTrackを同時に画面上に表示する

---

### 要件7: TextTrackへの入力操作

**ユーザーストーリー:** 音楽家として、TextTrackのセルにGoogleスプレッドシートのような感覚でテキストを入力したい。そうすることで、歌詞・コード・補足情報を直感的に記入できる。

#### 受け入れ基準

1. WHEN ユーザーがTextTrackのCellをダブルクリックする, THE App SHALL そのCellを編集モードにしカーソルを表示する
2. WHEN TextTrackのCellが編集モードである, THE App SHALL 任意の文字列（複数文字含む）の入力・編集・削除を受け付ける
3. WHEN 編集モード中にユーザーがEnterキーを押す, THE App SHALL 入力内容を確定して編集モードを終了し、Cellの移動は発生しない
4. WHEN 編集モード中にユーザーがTabキーを押す, THE App SHALL 入力内容を確定して次のCellを選択する
5. WHEN 編集モード中にユーザーがShift+Tabキーを押す, THE App SHALL 前のCellを選択する
6. WHEN 編集モード中にユーザーがEscキーを押す, THE App SHALL 入力内容を破棄して編集前の内容に戻し編集モードを終了する
7. WHEN 編集モードではなくCellが選択されている状態でユーザーが文字キーを入力する, THE App SHALL そのCellを編集モードにして入力内容で既存の内容を上書きする
8. WHEN 編集モードではなくCellが選択されている状態でユーザーがEnterキーまたはF2キーを押す, THE App SHALL そのCellを編集モードにして既存の内容を保持したままカーソルを末尾に表示する
9. WHEN 編集モードではなくCellが選択されている状態でユーザーがDeleteキーを押す, THE App SHALL そのCellの内容を消去して未記入にする

---

### 要件8: パフォーマンス

**ユーザーストーリー:** 音楽家として、入力操作に対してアプリが即座に反応することを望む。そうすることで、思考の流れを止めずにアイデアを記録できる。

#### 受け入れ基準

1. WHEN ユーザーがNoteNameを入力する, THE App SHALL 入力から16ミリ秒以内に画面を更新する
2. WHEN ユーザーがOctaveShiftのドラッグを行う, THE App SHALL ドラッグ中の各フレームを16ミリ秒以内に再描画する
3. WHEN ユーザーがNoteValueを変更する, THE App SHALL 変更から16ミリ秒以内にグリッドレイアウトを更新する
4. THE App SHALL グリッドのセル数が100000を超える場合でも、上記のパフォーマンス基準を維持する

---

### 要件9: データの保存と読み込み

**ユーザーストーリー:** 音楽家として、作成した採譜データを保存・読み込みしたい。そうすることで、後から編集を再開できる。

#### 受け入れ基準

1. WHEN ユーザーが保存操作を実行する, THE App SHALL グリッドデータ（全トラックを含む）をファイルにシリアライズして保存する
2. WHEN ユーザーが読み込み操作を実行する, THE App SHALL ファイルからデータをデシリアライズしてグリッドを復元する
3. THE Serializer SHALL グリッドデータを構造化テキスト形式（JSON）に変換する
4. THE Deserializer SHALL JSON形式のファイルをグリッドデータに変換する
5. IF 読み込んだファイルが不正な形式である, THEN THE App SHALL エラーメッセージを表示してグリッドを変更しない
6. FOR ALL 有効なグリッドデータに対して, 保存してから読み込んだ結果は元のグリッドデータと等価である（ラウンドトリップ特性）
7. THE App SHALL 未保存の状態であっても5分ごとに現在のプロジェクト内容を自動バックアップとして保存する
8. THE App SHALL バックアップを最大36世代まで保持し、36世代を超えた場合は最も古いバックアップを削除する

---

### 要件10: Undo/Redo

**ユーザーストーリー:** 音楽家として、操作を取り消したり、取り消した操作をやり直したい。そうすることで、入力ミスを素早く修正しながら試行錯誤できる。

#### 受け入れ基準

1. WHEN ユーザーがUndo操作を実行する, THE App SHALL 直前の操作を取り消してグリッドを1つ前の状態に戻す
2. WHEN ユーザーがRedo操作を実行する, THE App SHALL 直前のUndo操作を取り消してグリッドを1つ後の状態に戻す
3. THE App SHALL NoteNameの入力・変更・削除をUndo/Redoの対象操作として記録する
4. THE App SHALL OctaveShiftをUndo/Redoの対象操作として記録する
5. THE App SHALL NoteValueの変更をUndo/Redoの対象操作として記録する
6. THE App SHALL トラックの追加・削除をUndo/Redoの対象操作として記録する
7. IF Undo可能な操作が存在しない, THEN THE App SHALL Undo操作を無視する
8. IF Redo可能な操作が存在しない, THEN THE App SHALL Redo操作を無視する
9. WHEN 新たな操作が実行される, THE App SHALL それ以降のRedo履歴を破棄する
10. WHEN Undo/Redo操作が実行される, THE App SHALL 16ミリ秒以内に画面を更新する

---

### 要件11: コピー・ペースト・複製

**ユーザーストーリー:** 音楽家として、入力済みのセルや小節をコピー&ペーストまたは複製したい。そうすることで、繰り返しのフレーズを素早く記録できる。

#### 受け入れ基準

1. THE App SHALL 単一CellのコピーをサポートするためCmd+Cキーによるコピー操作を提供する
2. THE App SHALL 複数の連続するCellを範囲選択してまとめてコピーする操作を提供する
3. THE App SHALL Bar単位でのコピー操作を提供する。このとき全てのTrackが対象となり、ペースト先も全てのTrackに対して上書きされる
4. WHEN ユーザーがコピー後にペースト先のCellまたはBarを選択してCmd+Vキーを押す, THE App SHALL コピーした内容をペースト先に上書きする（時間軸はシフトしない）
5. WHEN ペースト対象の範囲がペースト先の起点から超える, THE App SHALL ペースト先の起点から順に上書きし、はみ出した分も続けて後続のCellに上書きする
6. THE App SHALL トラックを横断した範囲選択は、位置として連続しているTrack間のみ許容する
7. WHEN 複数Trackにまたがるコピーを行う, THE App SHALL コピー元のTrackタイプの並び順（MelodyTrack/TextTrackの組み合わせ）を記録する
8. WHEN ペーストを実行する, THE App SHALL ペースト先のTrackタイプの並び順がコピー元と完全に一致する場合のみペーストを許容する
9. IF ペースト先のTrackタイプの並び順がコピー元と一致しない, THEN THE App SHALL ペースト操作を無視する
10. THE App SHALL MelodyTrackのCell範囲をTextTrackへペーストする操作を許容しない（逆も同様）
11. THE App SHALL トラックの複製操作を提供する
12. WHEN ユーザーがトラックを複製する, THE App SHALL 同一内容の新しいTrackをそのTrackの直下に追加する
13. THE App SHALL コピー・ペースト・複製操作をUndo/Redoの対象として記録する
