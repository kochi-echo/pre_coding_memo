# lsコマンド4コーディングメモ

ls -lコマンドの詳細

`man ls`の内容
> (The lowercase letter “ell”.) List files in the long format, as described in the The Long Format subsection
below.

>BLOCKSIZE
>
>If this is set, its value, rounded up to 512 or down to a multiple of 512, will be used as the block size in bytes by the -l and -s options.  See The Long Format subsection for more information.

The Long Formatの内容
`ls -l`は以下の構成となっている
BLOCKSIZE: ファイルシステム上の領域を扱う最小単位。各ファイルサイズは`ls -s`で確認可能
file mode:
number of links:
owner name: 
group name: 
number of bytes in the file: 
abbreviated month: 
day-of-month file was last modified:
hour file last modified:
minute file last modified:
pathname:


## 仕様を分解する

 1. 現状を把握する
 my-lsコマンドにより、横３列で3行に
 2. 追加仕様を把握する
    1. 追加仕様（あるべき姿）
    2. 追加仕様を機能に分割化
    3. 現状の機能との差分
 3. 機能をタスク化して優先順位を決める
    1. タスク一覧
    2. タスクの作業時間見積もり

## コードの構成を考える

1. 追加機能に見合うコードの構成を検討
   1. 入力と出力は何か？
   2. データ構造はどうすればいいか？
2. テストの構成を検討（正常系、異常系、セキュリティ）
3. 検討したコード構成に合うライブラリは？
