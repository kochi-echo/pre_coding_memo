# lsコマンド4コーディングメモ

`ls -l`では以下の結果が表示される。

- 合計ブロック数
[Q＆A: \`ls \-l\`コマンドで、一番最初に出てくるブロック数の計算方法を教えてください。 \| FBC](https://bootcamp.fjord.jp/questions/1903) を参照。

- ファイルタイプ
- パーミション
- ハードリンクの数
- オーナー名
- グループ名
- バイトサイズ
- タイムスタンプ
- ファイル名

## 仕様を分解する

 1. 現状を把握する
 my-lsコマンドにより、横３列で表示。
 2. 追加仕様を把握する
    1. 追加仕様（あるべき姿）
    細かい情報を縦１列で
    2. 追加仕様を機能に分割化
      - ファイルの細かい情報取得
      - 合計ブロック数計算
      - 縦１行で表示

## コードを構成する部品の検討

### ファイルの細かい情報取得

#### 各ファイルの情報に関して

File::Statを使用。

- ファイルタイプ
[【Ruby】File::Stat\#modeが返すファイルモードの数値を記号表記に変換する \- あまブログ](https://ama-tech.hatenablog.com/convert-file-permissions-numeric-to-symbolic-in-ruby) を参考に
- パーミション
同上
- ハードリンクの数
nlink
- オーナー名
Etc.#getpwuid(uid).name
- グループ名
Etc.#getpwuid(gid).name
- バイトサイズ
.size
- タイムスタンプ
mtime（またはctime）
- ファイル名
パスの末尾

#### ls -lコマンドの表示に関して

- 合計ブロック数計算
(blksizeの合計/OSのブロックサイズ).round
- 縦１行で表示

## 方針整理

### 追加機能の方針

ファイル情報の一覧表示には、見た目が崩れないため、各情報の比較が必要となる（例：`File::Stat.size`において`String#rjust(String#size.max)`をする必要がある）。そのため、以下の当初以下の方法を検討していた。

- 各ファイルの`File::Stat`をlist化
- 情報のタイプごとに表示テキストのリストを作るメソッドを作成（例；`convert_files_info_to_size_text_list`
  - `each`で各ファイルのサイズを取得し、テキストリストに入れる
  - テキストリストからmaxサイズを出して、`rjust`する
- 情報タイプごとのテキストリストに対して、各リストの先頭から一つのテキストとなるように結合して、lオプションで表示するファイル情報のテキストのリストを作成
- 既存のコードの表示機能に渡す

## テスト内容兼タスクリスト

- [x] オプションにより1列と3列を切り替え
- [x] 空の値を1列表示
- [ ] ファイル情報リスト表示整形
  - [ ] 英数字右端揃え整形メソッド`align_str_list_to_right`
    - 引数：`File::Stat#size`
    - 戻り値：ファイルサイズの整形されたテキストリスト
    - 中身：リストの中の`String#size.max`をもとにリストの要素を
  `rjust`
  - [ ] 日本語左端揃え整形メソッド`align_jp_str_list_to_left`
    - 中身：リストの中の`String#size.max`をもとにリストの要素を
  `ljust_jp`
- [ ] ファイル情報内容取得
  - [ ] ファイルタイプ&パーミション
  - [ ] ハードリンクの数
    - 引数：nlink
    - `align_str_list_to_right`
  - [ ] オーナー名
    - 引数：Etc.#getpwuid(uid).name
    - `align_str_list_to_right`
  - [ ] グループ名
    - 引数：Etc.#getpwuid(gid).name
    - `align_str_list_to_right`
  - [ ] バイトサイズ
    - 引数：size
    - `align_str_list_to_right`
  - [ ] タイムスタンプ
    - 引数：mtime
    - 月、日、時間でそれぞれリスト化
    - `align_str_list_to_right`
    - ファイルごとに月、日、時間を一つのテキストにしてリスト化
  - [ ] ファイル名
    - 引数：target_file or target_dirの末尾
    - `align_jp_str_list_to_left`
- [ ] 取得したファイル情報リスト群を各ファイルごとに一つのテキストとなるように結合
  - それぞれのファイル情報リストの先頭から繋ぎ合わせる(`Array#zip.map`)
- [ ] 合計ブロック数計算メソッド
  - バイトサイズsum/512をround
- [ ] 5を合体して表示（ls -lコマンド表示）
