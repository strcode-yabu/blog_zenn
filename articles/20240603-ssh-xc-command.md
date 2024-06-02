---
title: "SSH で GUI アプリを実行する"
emoji: "🖥️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linux", "ssh"]
published: true
published_at: 2024-06-03 12:00
---

## はじめに

私的なセットアップメモなども兼ねて、 Linux 端末に SSH で接続し GUI アプリ (VS Code など) を使うための設定やコマンドなどをまとめた記事を執筆します。

筆者の環境が、普段使用するデスクと開発用の Linux 端末の位置が離れており、いちいち移動して作業をするのが面倒なので、その手間を省くために設定や手法を調べました。

## 筆者の環境

筆者の環境です。  
普段使用しているのは Windows 11 Pro 搭載のディスクトップ PC と Chromebook になります。  
※ いずれも記事執筆時点

### Windows 端末

| OS | ターミナル |
| ---- | ---- |
| Windows 11 Pro 23H2 | Windows ターミナル (WSL2:Ubuntu) |

### Chromebook

| OS | ターミナル |
| ---- | ---- |
| Chrome OS 125 | 付属ターミナルアプリ (Debian Linux) |

また、接続される側は下記の環境は **Endeavor OS** を今回は用意します。

## Linux 側の準備

まずは接続される Endeavor OS 側の設定をします。  
必要なソフトは一通り入っているのでインストールは割愛します。

またセキュリティの観点から SSH キーの導入が本来望ましいですが、同じ部屋の中で必要なときだけ起動している Linux 端末に接続するので今回はそちらも割愛します。

### `ssh_config` の設定

```bash:/etc/ssh/ssh_config
Port #0-65535 の範囲の任意のポート
PermitRootLogin no
X11Forwarding yes
```

`/etc/ssh/ssh_config` を開き `Port`, `PermitRootLogin`, `X11Forwarding` を設定し保存します。

- `Port` : SSH で接続するポート番号です。デフォルト (`22`) のままだと攻撃のリスクがあるので `0` ~ `65535` の任意の番号に変更します。
- `PermitRootLogin` : Root ログインを許可するかの設定です。 `no` が推奨です。
- `X11Forwarding` : X11 の転送を許可する設定になります。 `yes` にすることで SSH 接続で GUI アプリを使うことができるようになります。

上記の設定をしたら `ssh_config` を保存します。

### `sshd` を再起動する

```bash
sudo systemctl restart sshd
```

`sshd` のサービスを立ち上げ直します。  
これで先ほど設定した `ssh_config` の設定が有効になります。

### `Firewall` の設定

`firewall-cmd` コマンドを使います。

```bash
sudo firewall-cmd --add-port=[ssh_configで設定したポート番号]/tcp --permanent --zone=public
```

`ssh_config` で設定した SSH ポートを `Firewall` に追加します。

- `--add-port` : 追加するポート番号を指定します。先に `ssh_config` で設定した番号を設定します。
- `--permanent` : 恒久的に設定を適用します。
- `--zone=public` : パブリックエリア向けにポートを開放する設定です。

```bash
sudo firewall-cmd --permanent --remove-service=ssh
```

`--remove-service=ssh` でデフォルトの SSH ポート (`22`) を OFF にします。

### `Firewall` の再起動

```bash
sudo systemctl restart firewalld
```

`Firewall` のサービスを立ち上げ直します。  
`firewall-cmd` での設定が有効になります。

## 接続側準備

Chromebook 側はそのままコマンドを叩けば接続可能ですが、 WSL2 (Ubuntu) で接続する Windows 側はアプリのインストールが必要です。

```bash
sudo apt install sshd -y
```

`ssh` コマンドを使うために `sshd` パッケージをインストールします。

## `SSH` 接続

```bash
ssh -XC user@192.168.1.10 -p 12345
```

接続先の IP アドレスが `192.168.1.10` 、ポート番号が `12345` 、使用するユーザーアカウントが `user` だった場合、上記のような `ssh` コマンドを使って接続します。  
コマンドのオプションは下記のような意味になります。

- `-X` : GUI アプリケーションを使用できるようにします。
- `-C` : すべての通信を圧縮します。
- `-p` : ポート番号を指定します。

接続がうまくいけば、使用するユーザーアカウントのパスワードを入力するようになります。  
パスワードを入力し、ログインします。

## GUI アプリを使う

SSH 接続がうまくいったら次は GUI アプリを起動します。  
たとえば MousePad を起動する場合は下記のようにコマンドを入力します。

```bash
mousepad
```

すると MousePad が立ち上がり使用できるようになります。

## 最後に

いかがでしたか。  
このように思ったよりも少ない手順かつ簡単な設定で、同じネットワーク内にある Linux 端末に接続し、 GUI のアプリを使うことができました。  
最初は Chrome Remote Desktop を使うことなどを検討しましたが、こちらのほうが導入が楽です。

この記事が同じようなことを実行しようとしている方の助けとなれば幸いです。

## 参考記事

- Crostini で X11 を使って Linux サーバのGUIアプリを Chromebook のディスプレイに表示して利用
  - [https://blog.ideastorage.net/posts/chromebook-with-crostini-and-x11/](https://blog.ideastorage.net/posts/chromebook-with-crostini-and-x11/)
