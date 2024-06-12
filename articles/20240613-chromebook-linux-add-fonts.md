---
title: "Chrome OS の Linux アプリで使用するフォントを追加する"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["chromebook", "chromeos"]
published: true
published_at: 2024-06-13 12:00
---

## はじめに

2024年3月4日に [Chrome OS の Linux アプリで日本語入力可能な環境を構築する](/articles/20240304-chromebook-on-dev) を執筆しました。  
日本語の入力も行える VS Code 環境などを整えることができましたが、実はまだ足りていないものがあります。  
そう、フォントが初期で入っているものしか設定できないのです。  
この記事では Linux アプリで任意のフォントが設定できるようにどのようにしたら良いか、手順をまとめていきます。

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

## 対象となるフォント

今回は [tawara](https://github.com/yuru7) 氏 制作の [UDEV Gothic](https://github.com/yuru7/udev-gothic#readme) を導入してみます。

## フォントをダウンロード

今回はユーザーディレクトリ直下に **Download** ディレクトリを作成しそこに対象となるファイルをダウンロードします。

```bash
mkdir ~/Download
```

対象となるファイルの URL を取得し、 `wget` コマンドを使ってダウンロードを行います。

```bash
cd ~/Download
wget https://github.com/yuru7/udev-gothic/releases/download/v1.3.1/UDEVGothic_v1.3.1.zip
```

今回の例では成功すると **UDEVGothic_v1.3.1.zip** がダウンロードされます。

## `.zip` ファイルを展開する

ダウンロードされた **UDEVGothic_v1.3.1.zip** を展開します。  
展開には `unzip` コマンドを使用します。

```bash
unzip UDEVGothic_v1.3.1.zip
```

展開が完了すると **UDEVGothic_v1.3.1** ディレクトリが作成され、その下に展開されたファイルが配置されます。

## 展開されたフォントデータを `~/.local/share/fonts` 下のディレクトリに移動する

無事にフォントデータが展開できたらいよいよ対象となるフォントの設置を行います。

まずは `mkdir` コマンドを使ってフォントを設置するためのディレクトリを作成します。

```bash
mkdir -p ~/.local/share/fonts/UDEVGothic
```

今回の例では **UDEVGothic** というディレクトリを `~/.local/share/fonts/` の下に作成しました。  
`-p` コマンドを用いているのは間のディレクトリがなくても自動で作成を行ってもらうためです。

ディレクトリの作成が終わったらいよいよフォントデータを移動します。  
移動には `mv` コマンドを使用します。

```bash
mv UDEVGothic_v1.3.1/**.ttf ~/.local/share/fonts/UDEVGothic
```

以上でフォントデータの設置は完了です。

ダウンロードされたファイルや、展開で発生したディレクトリが不要な場合は削除を行います。

```bash
cd
rm -rf ~/Download/*
```

## フォントキャッシュを更新し、利用できるフォントを確認する

ここまでできたらあとはフォントキャッシュを更新し、フォントが利用できるようになったか確認します。  
キャッシュの更新には `fc-cache`  、フォント一覧確認には `fc-list` のコマンドを使用します。

```bash
fc-cache --force --verbose
```

`--force` はタイムスタンプにかかわらずすべてのフォントを対象に、 `--verbose` は確認中のディレクトリを表示しながら処理をするというオプションです。  

最後にフォントが利用できるようになったか確認します。

```bash
fc-list | grep UDEV
```

うまく行っていれば **UDEV Gothic** のフォントファミリー名が表示されます。

## 動作確認

フォントの追加が完了したら、 Linux アプリでフォントが利用できるか確認してみましょう。  
前回の [Chrome OS の Linux アプリで日本語入力可能な環境を構築する](/articles/20240304-chromebook-on-dev) でも動作の確認に使用した **MousePad** で設定できれば OK です。

## 最後に

いかがだったでしょうか。  
Chromebook で Linux アプリを動作させるところまではかなり記事が多い印象がありますが、好みのフォントを設定してみるというところを網羅した記事はなかなかないのではないかなと考えます。

今回の記事が同じようなことで悩んでいる方の参考になれば幸いです。

## 参考記事

### Chromebook の Linux 開発環境におけるフォント設定

[https://acclimal.blogspot.com/2022/02/chromebook-linux.html](https://acclimal.blogspot.com/2022/02/chromebook-linux.html)
