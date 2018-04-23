---
title: "PeerTubeのススメ"
date: 2018-03-20T17:15:00+09:00
slug: "peertube-no-susume"
---

# PeerTubeのススメ
Nafです。  
[PeerTube](https://github.com/Chocobozzz/PeerTube)を建ててみました。  
建てたのは[ここ](https://tube.otogamer.me)  
本稿ではPeerTubeのアレソレについて書きます。  

## What is "PeerTube"?

>Federated (ActivityPub) video streaming platform using P2P (BitTorrent) directly in the web browser with WebTorrent. 

(Google翻訳)  

>P2P（BitTorrent）をWebブラウザでWebTorrentで直接使用するFederated（ActivityPub）ビデオストリーミングプラットフォーム。

完全に理解した。


## 用意したサーバー
他のアプリーケションも同居しているので結構リッチです。

* さくらのクラウド 石狩リージョン
* OS: Fedora27
* CPU: 4core
* RAM 4GB
* SSD 100GB

ちゃんと測っていないのでアレですが、平常時でRAMは200MB程使っているみたいです。

## 建て方
[ここ](https://github.com/Chocobozzz/PeerTube/blob/develop/support/doc/production.md)に書いてます。  
ドキュメントが充実しており、簡単に設置・アップデートできると思われます。  

詰まったとこは特にないのですが、nginxを`Peertube`ユーザーで動かす時のパーミッション設定を忘れずにしましょう。  

## 利点
* ActivityPubを採用しているため、マストドンに流れてくる。

こんな感じで来ます。

![PeerTube](https://media.otogamer.me/media/media_attachments/files/000/144/013/original/816dc98bd39bb6d4.png)

また、動画が埋め込まれるので、Tootの中で動画が見れます。

![PeerTube](https://media.otogamer.me/media/media_attachments/files/000/143/904/original/5f5ff58a4611278d.png)

* 軽い

最初はエンコードなどで結構重いのかな？と思っていましたが、意外と軽いです。  
また、「動画のエンコードを行なうかどうか」という設定があるのでCPUが貧弱なサーバーでも立ち上げれます。  
エンコードするならCPUのコア数は2以上がおすすめです。  

## 気になるところ
* 60fpsの動画をアップロードできない

[Issue](https://github.com/Chocobozzz/PeerTube/issues/310)にもあるんですが、できません。(30fpsに下げられる)  
試しにエンコ無し設定にして上げてみましたが、プチプチになって駄目でした。  
今後の開発に期待です。  

## まとめ

あまり建てている人がいなくて寂しいのでみんな建ててください。ハイ。


