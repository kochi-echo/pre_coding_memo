# lsコマンド2

## 意識すること

- 目標時間を決め、タイマーをかける
- ゴールと方針を日本語で書いてからコードに手をつける
- 時間内に終わらなかったら、もう一度やりたいことを書き出してみる&オーバーした理由を書く

## 書く前に考えること

### 仕様を分解する

 1. 現状を把握する（既存のコードがあれば）
    .から始まるフォルダとファイルは表示されない
 2. 追加仕様を把握する
    ls -aの実装
 3. 機能をタスク化して優先順位を決める
    1. どこの段階で.から始まるファイル名が取得できていないか確認
    2. 引数取得方法を調べる
    3. -aのテストを書く
    4. -aのメインを修正

### ベタに書く

Fake It(ロジックらしいロジックがない固定の値を返すような仮実装のこと(メソッドとテストコードがリンクしているかの疎通確認）)を意識する。

1. テストコードを書く
2. メインコードを書く
   - 15分ハマってChatGPTに聞いてもダメなら、他の人に聞く
   - 正常系完成したらレビューしてもらう

### クラスに分ける

### 清書する

## 書くとき

### おまじない

```ruby
#!/usr/bin/env ruby
# frozen_string_literal: true
```

## push前にチェックすること

- [ ] rubocopパスしているか？
- [ ] 組み込み・標準添付ライブラリの単語を使っているか？
- [ ] クラスは名詞、メソッドは動詞か？
- [ ] 頻出名詞をクラスに抜き出す
- [ ] ハッシュテーブルによる分岐数削減
- [ ] push先とpushするbranchは正しいか？
- [ ] PR先は正しいか？
- [ ] 引数のないメソッドは可読性のためインスタンス変数っぽく名詞の名前をつけているか？
- [ ] 1メソッドは五行以内に収めるているか？
- [ ] DRY原則("Don't repeat yourself")に従っているか？

## 仕様

### 実装内容

### 実装していない内容

- 引数が複数の場合の表示  
実際のlsは引数が複数でも全て表示するが、ここでは一番最後の引数だけ表示することとする  
ex. ls -a one two  
上記は以下と同意になる  
ls -a two

### 注意点
