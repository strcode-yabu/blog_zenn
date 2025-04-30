---
title: "GAS を使ってメールフォームのバックエンドを作る"
emoji: "📧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["gas", "web"]
published: true
published_at: 2024-06-26 12:00
---

## はじめに

Web の学習で HTML・CSS でサイト制作を行うときに、お問い合わせフォームを作成することがあるかと思います。  
HTML・CSS を用いて外装までは作れるかもしれないですが、その先の送信処理は PHP などのプログラム言語を使ってサーバーサイドにコードを設置しないといけないので、かなり気合の入った人でないと作ったことはないのではと想像します。

ですが、その常識も変わって行っています。  
Google が提供する **Google Apps Script** を活用すれば PHP などが使えるサーバーを用意しなくても、送信の処理を実装することが可能となります。  
早速見てみましょう。

:::message
2025.4.30 テスト用の HTML、実行権限設定に関する記述を追記。
:::

## Google Apps Script とは

Google が提供する JavaScript をベースにしたスクリプト言語です。

Google ドキュメントやスプレッドシートのマクロのような使われ方がメインですが、 Gmail やカレンダーを操作することもできるので様々な活用をすることができます。

なお利用には Google (Gmail) のアカウントが必要になります。  
※ 本記事では Google のアカウント作成については割愛します。

**Google Apps Script** について、詳しくは [こちら](https://developers.google.com/apps-script/) をご覧ください。

## 実装の流れ

### HTML を確認

今回のテストには下記の HTML フォームを使用します。

:::details テストフォーム HTML

```html
<form action="GAS Web App URL" method="post">
  <p>
    <label>
      名前:<br>
      <input type="text" name="name" id="name">
    </label>
  </p>
  <p>
    <label>
      メールアドレス:<br>
      <input type="email" name="email" id="email">
    </label>
  </p>
  <p>
    <label>
      内容:<br>
      <textarea name="content" id="content" cols="32" rows="5"></textarea>
    </label>
  </p>
  <button type="submit">
    送信
  </button>
</form>
```

:::

送信されるデータは下記の通りです。

- `name` (送信者名前)
- `email` (送信者メールアドレス)
- `content` (送信内容)

### 新規のプロジェクトを作成する

![""](/images/20240626-form-backend-on-gas/img001.png)

Google ドライブにアクセスして新規作成を行います。

![""](/images/20240626-form-backend-on-gas/img002.png)

作成するファイルは **Google Apps Script** を選択します。

### プロジェクト内に `doPost()` 関数を作成する

```javascript
const doPost = e => {
  Logger.log(e);
};
```

上記のコードを書くことによりフォームから送られてきた `POST` データを受け取ることができます。  
`e.parameter` にフォームのデータが含まれているので、そちらからデータを取り出します。

今回のテスト用フォームでは、下記のデータを受信します。

- `name` (送信者名前)
- `email` (送信者メールアドレス)
- `content` (送信内容)

データを受け取ったときのコードは下記のようになります。

```javascript
const doPost = e => {
  const name = e.parameter.name;
  const email = e.parameter.email;
  const content = e.parameter.content;
};
```

その他の詳しいパラメーターは [公式ドキュメント](https://developers.google.com/apps-script/guides/web) を参照してみましょう。

### 送信用の関数を作成する

POST データを受信できたので、次は Gmail を使ってメールを送信する処理を書いていきます。  
メールの送信には `GmailApp.sendEmail()` 関数を使用します。

```javascript
const sendMail = (name, email, content) => {
  const recipient = email;
  const recipientName = name;
  const subject = "Test mail title.";

  const body = `${recipientName} 様より\n\nテストメールです。\n\n${content}`;

  const options = {
    from: "test@example.com",
    name: "テスト太郎"
  }

  GmailApp.sendEmail(recipient, subject, body, option);
};
```

上記のコードで送信の処理を行います。

今回オプションには `from` と `name` を設定していますが、 `cc` や `bcc` なども設定が可能です。  
その他の詳しいオプションについては [公式ドキュメント](https://developers.google.com/apps-script/reference/gmail/gmail-app) を参照してみましょう。

送信用の関数ができたら、そちらに値を渡せるように `doPost()` 関数を修正します。

```javascript
const doPost = e => {
  const name = e.parameter.name;
  const email = e.parameter.email;
  const content = e.parameter.content;

  sendMail(name, email, content);
};
```

### コードが完成したら Web アプリとして公開する

![""](/images/20240626-form-backend-on-gas/img003.png)

エディター画面右上の **デプロイ** ボタンをクリックし、 **新しいデプロイ** を実施します。

![""](/images/20240626-form-backend-on-gas/img004.png)

新しいデプロイ画面が出てきたら **種類の選択** の歯車マークをクリックして **ウェブアプリ** を選択します。  
わかりやすいように説明の部分は入れておくとあとから管理がしやすくなります。

![""](/images/20240626-form-backend-on-gas/img005.png)

またフォームの送信先として使う場合は、 **次のユーザーとして実行** の項目を **ウェブアプリケーションにアクセスしているユーザー** に設定しておく必要があります。

デプロイが終わったら Web アプリ用の URL が発行されるので、そちらを送信用フォームの `<form>` タグの `action` 属性の値に設定します。  
※ この際に `method` 属性が `post` になっていることも確認しておきましょう。

### 動作テスト

設定ができたら実際に送信できるか確認してみましょう。  
メールを受信できたら OK です。

ちなみに今回はサンクス画面への遷移などは設定していませんが、実際に活用するときにはそちらも設定するようにしましょう。

## 最後に

いかがでしたか。  
簡易な例を用いての解説でしたが、かなりかんたんにフォームのバックエンド処理が実装できました。  
PHP などサーバー側はまだ学習中だけど、実際に作ったフォームを動かしてみたいという場合、 GAS はひとつの選択肢となるのではないでしょうか。

## 参考サイト

- [GAS で web アプリの作成とパラメータの確認方法(doGet、doPost)](https://breezegroup.co.jp/201906/gas-get/)
- [GAS で GET,POST を受け取る方法](https://kin29.info/gas%E3%81%A7getpost%E3%82%92%E5%8F%97%E3%81%91%E5%8F%96%E3%82%8B%E6%96%B9%E6%B3%95/)
- [【GAS】Gmail でメールを送信するには？](https://masagoroku.com/%E3%80%90gas%E3%80%91gmail%E3%81%A7%E3%83%A1%E3%83%BC%E3%83%AB%E3%82%92%E9%80%81%E4%BF%A1%E3%81%99%E3%82%8B%E3%81%AB%E3%81%AF%EF%BC%9F)
