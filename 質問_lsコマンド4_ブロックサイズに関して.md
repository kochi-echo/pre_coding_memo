# 質問

## 質問概要

「lsコマンドを作る4」の課題に取り組むにあたり、`ls -l`コマンドで、一番最初に出てくるブロック数の計算方法を教えてください（以下の図の'total 16'）。

図

## 詳細

### 環境

PC: M2 Macbook Air
OS: MacOS Ventura 13.4
Shell: zsh

### ブロックサイズとブロック数の認識確認

まず、私のブロックサイズの認識から確認させてもらいます。

`man ls`および参照したリンク先から、Unix系のファイルシステムではメモリへの書き込み時に1bitごとに書き込むのではなく、効率化のためある決まったbyte数をまとめて１単位として扱って書き込みを行います。この1単位として扱うファイルのbyte数をブロックサイズと呼びます。例としてブロックサイズが256 bytesの場合、ファイルが1024 bytesなら、このファイルは4ブロックとして扱われます。

Mac terminal上において`man ls`コマンドの説明では、デフォルトのブロックサイズは512 bytesとして書いてありますが、私のPCのOSでは、`diskutil info / | grep "Block Size"`で確認したところ、ブロックサイズは4096 bytesでした。

ブロック数が表示される時、ファイルのByte数はブロックサイズに丸め込みされて表示されます。例として、ブロックサイズが256 bytesの場合、ファイルが1000 bytesや1050 bytesだとしても、このファイルは4ブロックとして扱われます（実際のソフトでの処理で何らかのメモリ圧縮処理がされているのか、単なる表示の問題なのかは不明）。

<details><summary>参照</summary>

`man ls`の内容抜粋

The Long FormatでのBLOCKSIZEの記述
> The default block size is 512 bytes.  The block size may be set with option -k or environment variable BLOCKSIZE.  Numbers of blocks in the output will have been rounded up so the numbers of bytes is at least as  many as used by the corresponding file system blocks (which might have a different size).

BLOCKSIZEの説明
> If this is set, its value, rounded up to 512 or down to a multiple of 512, will be used as
the block size in bytes by the -l and -s options.  See The Long Format subsection for more information.

参考にしたリンク
- [Linuxのファイルシステムについて勉強したメモ \#Linux \- Qiita](https://qiita.com/toshihirock/items/ca717f8ad9c66ced4047)
- [【Linuxのしくみ】8章 ストレージデバイス を読んで自分なりにまとめ \- フラミナル](https://blog.framinal.life/entry/2020/04/22/030533#:~:text=%E3%83%96%E3%83%AD%E3%83%83%E3%82%AF%E3%82%B5%E3%82%A4%E3%82%BA%E3%81%A8%E3%81%AF%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB,%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%81%A7%E7%A2%BA%E8%AA%8D%E3%81%A7%E3%81%8D%E3%81%BE%E3%81%99%E3%80%82&text=%E3%81%93%E3%81%AE%E7%AD%90%E4%BD%93%E3%81%A7%E3%81%AF4096byte,%E3%81%A7%E5%91%BC%E3%81%B3%E5%87%BA%E3%81%97%E3%82%92%E8%A1%8C%E3%81%84%E3%81%BE%E3%81%99%E3%80%82)
- [Linuxのファイルシステムについて勉強したメモ \#Linux \- Qiita](https://qiita.com/toshihirock/items/ca717f8ad9c66ced4047)

</details>

### 質問の詳細

`ls -l`コマンドにおいて、一番最初に出てくるブロックサイズの計算方法に関して、自分の認識では'ファイルサイズの合計/ブロックサイズ'に対してブロックサイズの倍数で近いものに丸め込みしたものが、totalで表示されるブロック数だと考えています。
そのため、次のコードを用いて計算すると以下の図ではブロック数は1になると考えました。

```ruby
block_size = 4096
file_size_total = (4847 + 320).to_f
num_block = (file_size_total/block_size).round
=> 1
```

図

念の為`ls -a -l`でも計算してみると、やはりこちらもブロック数は1となります。

```ruby
block_size = 4096
file_size_total = (160 + 192 + 192 + 4847 + 320).to_f
num_block = (file_size_total/block_size).round
=> 1
```

図

ここまで考えて何か大きな誤解をしていると思っているのですが、それが何か分かっていません。ご指摘頂きたいです。
よろしくお願いいたします。
