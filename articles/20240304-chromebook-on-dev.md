---
title: "Chrome OS の Linux アプリで日本語入力可能な環境を構築する"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["chromebook", "chromeos"]
published: true
published_at: 2024-03-04 12:00
---

## はじめに

私的なセットアップメモと Zenn の投稿のテストも兼ねて記事を執筆します。  

Chrome OS で Linux アプリが稼働するようになりすでに数年が経過しました。  
初期は Linux アプリ上での日本語入力の対応にかなりの手間をかける必要がありましたが、それもかなり軽減されたのでどのようにセットアップをすればいいのか手順をまとめておきます。

## 筆者環境について

筆者は **ASUS Chromebook Flip C436FA** にてこちらの確認を行っております。  
詳しいスペックなどは下記をご覧ください。

### ASUS Chromebook Flip C436FA

#### 簡易スペック

| CPU | RAM | SSD |
| ---- | ---- | ---- |
| インテル Core i7-10510U | 16GB LPDDR3-2133 | 512GB (PCI Express 3.0 x2 接続) |

#### 他詳細スペック

- ASUS 製品ページよりスペックへ
  - [https://www.asus.com/jp/laptops/for-home/chromebook/asus-chromebook-flip-c436/techspec/](https://www.asus.com/jp/laptops/for-home/chromebook/asus-chromebook-flip-c436/techspec/)

## Linux をセットアップする

まずは Chrome OS で Linux 開発環境を有効にします。  
Chrome OS の設定画面から

`詳細設定 -> デベロッパー -> Linux 開発環境`

へ進みます。  
そこにある `オンにする` のボタンを選択し画面上の手順にそってセットアップを行います。  
時間にして 10 分以上かかることもあります。

Linux のセットアップが終わったら日本語入力のための設定を行っていきます。

## `Crostini IME Support` フラグを有効化

日本語の対応が簡便化された背景には、Chrome OS の設定に `Crostini IME Support` が追加され、 Chrome OS 側の IME が使えるようになったことが大きく関係しています。  
まだ **Experimental (実験的な) 機能** (2024.3.4 執筆現在) ではあるので安定性を欠く可能性がないとも言えません。  
ですが Chrome OS は リセットからの復旧もかんたんに行えるのが特徴なので、いざとなれば端末を **Powerwash (初期状態へのリセット)** してしまえば済みます。

では手順を見てみましょう。

1. アドレスバーに `chrome://flags` を入力し、 **Experiments** を開く
2. *Search flags* に `crostini` と入力し検索
3. `Crostini IME Support` フラグを `Enable` にする
4. Chrome OS を再起動する

以上再起動が完了すれば Linux のアプリで Chrome OS の IME を使って、日本語入力を使えるようになります。  

### US 配列キーボードユーザーへの注意

上記の手順では JIS 配列キーボードユーザーなら問題ないですが、 US 配列キーボードを使っているユーザーはもうひと手間、作業をする必要があります。 (2024.3.4 時点)  

具体的に何が問題なのか。

IME の ON/OFF の切り替えを行うキーボードショートカット `Ctrl + Space`, `Ctrl + Shift + Space` を使うことができません。  
現在開発チームも公式な対応を模索している最中とのことですが、暫定的に下記の設定を施すことで使用できるようになります。

```bash:~/.config/systemd/user/sommelier@.service.d/cros-sommelier-override.conf
[Service]
Environment="SOMMELIER_ACCELERATORS=Super_L,<Alt>bracketleft,<Alt>bracketright,<Control>space,<Control><Shift>space"
```

```bash:~/.config/systemd/user/sommelier-x@.service.d/cros-sommelier-x-override.conf
[Service]
Environment="SOMMELIER_ACCELERATORS=Super_L,<Alt>bracketleft,<Alt>bracketright,<Control>space,<Control><Shift>space"
```

上記のファイルを作成します。  
*※ フォルダとファイルの作成は必要に応じて行ってください。*  
あとは Linux アプリをすべて閉じ、シェルフの **ターミナル** のアイコンを右クリックし、 **Linux をシャットダウン** してから *Crostini* を再起動すれば設定が適用されます。  

## 日本語フォントを追加 / 文字コード・ロケールを日本に設定

初期の状態では中華フォントなど見た目に違和感のあるフォントが表示される場合があります。  
その場合は下記のフォントのインストールや文字コード、ロケールの設定を行いましょう。  

```bash
# 日本語フォント (定番ではあるが Noto Fonts CJK) を導入
sudo apt install fonts-noto-cjk -y
# 文字コード・ロケールを設定
sudo localectl set-locale LANG=ja_JP.UTF-8
```

## 実際の動作を確認

試しになにか動作を確認できる Linux アプリを入れてみましょう。  
手頃なところだとやはりテキストエディタでしょうか。

[Xfce](https://docs.xfce.org/) などのデスクトップ環境でもおなじみの [Mousepad](https://docs.xfce.org/apps/mousepad/start) をインストールしてテストしてみましょう。  

```bash
sudo apt install mousepad
```

インストールが完了したらランチャーから Mousepad を起動して動作確認です。  

## まとめ

今回の手順は [Chrome OS Flex](https://chromeenterprise.google/intl/ja_jp/os/chromeosflex/) でも使うことができます。  
このあと [Visual Studio Code](https://azure.microsoft.com/ja-jp/products/visual-studio-code) を導入してコーディング環境を構築することもできますし、 [Typora](https://typora.io/) などのワープロツールを入れてモバイルの執筆環境として活用もできます。

この記事が私と同じことをやろうとしている方の一助になればと考えます。

## 参考記事

### \[2023年5月版\]Chromebook Linux でIMEの設定がめっちゃ楽になってた

[https://zenn.dev/asopitech/articles/20230516-103621_1](https://zenn.dev/asopitech/articles/20230516-103621_1)

### ChromeOS IME on Crostini

[https://qiita.com/Daru-IBN5100/items/a32cb35238bd968d2a4b](https://qiita.com/Daru-IBN5100/items/a32cb35238bd968d2a4b)
