# lsコマンド4コーディングメモ

## lオプションに関して

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
- [x] ファイル情報リスト表示整形
  - [x] 英数字右端揃え整形メソッド`align_str_list_to_right`
    - 引数：strのリスト
    - 戻り値：ファイルサイズの整形されたテキストリスト
    - 中身：リストの中の`String#size.max`をもとにリストの要素を
  `rjust`
  - [x] 日本語左端揃え整形メソッド`align_jp_str_list_to_left`
    - 中身：リストの中の`String#size.max`をもとにリストの要素を
  `ljust_jp`
- [x] ファイル情報内容取得
  - [x] ファイルタイプ&パーミション
    - [x] Linuxファイルモードの意味を調べる
    結局ただのbitをマスクして、8進数で状態判別しているだけだった
    - [x] ファイルモードをbit分割し、各要素でhashに入れる
    - [x] ファイルタイプ：4bitに対して、8進数の対応記号ハッシュと照らし合わせて、テキスト出力
    - [x] 特殊権限：4bitに対して、8進数の対応記号ハッシュと照らし合わせて、テキスト出力
    - [x] パーミッション：3bitに対して、対応bitが1の時、対応記号をテキストに書き込む
    - [x] 最後に@をつける
  - [x] ハードリンクの数
    - 引数：nlink
    - `align_str_list_to_right`
  - [x] オーナー名
    - 引数：Etc.#getpwuid(uid).name
    - `align_jp_str_list_to_left`
  - [x] グループ名
    - 引数：Etc.#getpwuid(gid).name
    - `align_jp_str_list_to_left`
  - [x] バイトサイズ
    - 引数：size
    - `align_str_list_to_right`
  - [x] タイムスタンプ
    - 引数：mtime
    - 月、日、時間でそれぞれリスト化
    - `align_str_list_to_right`
    - ファイルごとに月、日、時間を一つのテキストにしてリスト化
  - [x] ファイル名
    - 引数：target_file or target_dirの末尾
    - `align_jp_str_list_to_left`
- [x] 取得したファイル情報リスト群を各ファイルごとに一つのテキストとなるように結合
  - 取得した内容を`files_info_for_each_info_type`に追加していく
  - transposeとjoinでリストの先頭から繋ぎ合わせる
- [x] 合計ブロック数計算メソッド
  - バイトサイズsum/512をround->block.sumで
- [x] 合計ブロック数とファイル情報を合体して表示（ls -lコマンド表示）
- [x] ファイル分割(method_and_parts_of)
- [x] コマンドでの動作確認

## 提出チェック表

- [x] rubocopをパスしているか？
- [x] 組み込み・標準添付ライブラリの単語を使っているか？
- [x] クラスは名詞、メソッドは動詞か？
- [ ] ~~引数のないメソッドは可読性のためインスタンス変数っぽく名詞の名前をつけているか？~~
- [ ] 1メソッドは五行以内に収めるているか？
→未達成（githubのConversation参照）
- [ ] ~~頻出名詞をクラスに抜き出しているか？~~
- [x] ハッシュテーブルによる分岐数削減しているか？
- [x] 3つの原則(KISS, YAGNI, DRY)に従っているか？
- [x] push先とpushするbranchは正しいか？
- [x] PR先は正しいか？

## 修正依頼を受けてメソッド構成変更

- get_file_names
  - 入力：コマンド引数
  - path_to_directory_and_file
  - select_files
  - 出力：表示するテキストリスト
- path_to_directory_and_file
  - 入力：絶対パス
  - 出力：ディレクトリとファイル名
- select_files
  - 入力：ファイル名、ディレクトリ名
  - get_files_info_text
  - 出力：表示するテキストリスト
- get_files_info_text
  - 入力：ディレクトリ、全ファイル名
  - get_files_info_each_type
  - 出力：表示するテキストリスト
- get_files_info_each_type
  - 入力：ディレクトリ、全ファイル名
  - 出力：各ファイルの情報のテキストリスト

### 課題

- メソッド名にgetを使わない
- get_file_namesがファイル名ではなく、表示するテキストリストを出力している
- select_filesがget_file_namesからしか、呼び出されていない
- get_files_info_textとget_files_info_each_typeを分ける必要がない
- get_files_info_textでテストをするより、get_file_namesでテストをするべき

### 修正案

#### get_file_namesがファイル名ではなく、表示するテキストリストを出力している

名前変更

get_file_names -> argument_to_files_info_list
generate_name_list_text -> files_info_list_to_displayed_text

#### 不必要なメソッド分割

select_filesをget_file_namesに挿入
get_files_info_each_typeをget_files_info_textに挿入

#### get_files_info_textでテストをするより、get_file_namesでテストをするべき

get_file_namesでテスト修正

#### get_files_info_textとget_files_info_each_typeを分ける必要がない

#### その他

conver_はいらない

### タスクリスト

- [x] メソッド名と戻り値名変更
- [x] covert_削除
- [x] select_filesをget_file_namesに挿入し、テスト確認
- [ ] get_files_info_each_typeをget_files_info_textに挿入し、テスト確認->テストしにくかったため、そのままにした
