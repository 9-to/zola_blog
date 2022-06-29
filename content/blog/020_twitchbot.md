+++
title = "TwitchAPIとGoogle Cloud Platformで配信通知Discord botを作る(python)"
date = 2022-04-12
[taxonomies]
tags =["programing"]
+++

Twitchのチャンネルが配信を始めたらそれを知らせるdiscord botを作った。

<!-- more -->

<br><br>

## 動機
+ 見たいDJライブ配信を見逃したくない
+ google cloud platform(以下,GCP)を使って何か作りたかった

<br>

## ゴール
+ あるTwitchチャンネルが配信を開始したらDiscordのテキストチャンネルに通知するbotを作る
+ 無料で実現させる

<br>

## 用意したもの
+ discordアカウント
    + bot用のwebhookURLを発行しておく
+ googleアカウント
    + GCP
    + Firebase
+ Twitchアカウント
    + API利用に使う
+ 電話番号
+ クレジットカード
+ python環境
+ 自己効力感

*注：GCPには無料枠が存在する。この範囲でアプリを動かすならば実質無料でbotを運用できる。アプリを回しすぎたり悪意ある他者に攻撃されて悪用されたりした場合は料金が発生する可能性がある。*

<br>

## 製作期間
2日(慣れていれば1日で作れる)

<br>

## 出来上がったもの
[githubに上げている](https://github.com/9-to/twitchBot_GCP)

<br>

## 主に参考にしたもの
+ [Twitchのチャンネルが配信を開始したらDiscordへ通知する](https://qiita.com/syakesalmon/items/dabb90a3bf66fb548c81)
    + 機構はこれを参考にしている
    + この記事ではjavaだが丸パクリでは面白くないのでpythonを使って書いている
+ [TwitchAPI](https://dev.twitch.tv/docs/api/)
+ [Flask](https://flask.palletsprojects.com/en/2.1.x/)
    + GCP上でこのフレームワークを動かせる
    + httpトリガーの機構を作るときに自環境でテストできる

<br>

## 作り方
### 1. 設計
[この仕組み](https://qiita.com/syakesalmon/items/dabb90a3bf66fb548c81)をそのまま使う。

*注：Twitchには二つのサブスクリプションがある。チャンネルに有料登録して様々な特典を受け取るサブスクと、**イベントの通知をAPIを利用して（無料で）おこなうサブスク**だ。紛らわしいので以下では前者を（狭義の）サブスク、後者を**EventSub**と呼称する。*

### 2. 下準備
『Twitchのチャンネルが配信を開始したらDiscordへ通知する』の人がだいたい書いてくれてるのでそれを補填する感じで若干書く。

#### 0. discord webhookを発行する
チャンネルの編集>連携サービス>ウェブフック>新しいウェブフック で作ることができる。

後々使うのでwebhook URLを手元に控えておく。
#### 1. Tiwtch Developersアカウントからアプリを登録する
[Tiwtch Developers](https://dev.twitch.tv/console)からアプリを登録する。アプリケーションを登録するには、tiwtchアカウントで2段階認証をセットアップする必要がある。そのためには電話番号が必要。

2段階認証が終わったらアプリを登録する。

Tiwtch Developersからコンソールに飛び、『アプリケーションを登録』をクリック。

名前、リダイレクトURL、カテゴリーを入力して作成。リダイレクトURLは`http://localhost`で良い。

作成したら、『クライアントID』と『クライアントの秘密』（つまり秘密鍵）を手元に控える。（これらはAPIの利用に必要となる）

#### 2. Firebaseプロジェクトを立てる
GCPでプロジェクトを立てる前にFirebaseでプロジェクトを立てる。正直どちらが先でもいい。

プロジェクトを立ち上げたらやることは2つある。

##### Ⅰ. Firebase Admin SDKの用意
コンソール画面>⚙(歯車マーク)>プロジェクトの設定>サービス アカウント>新しい秘密鍵の生成

ここで生成したFirestoreの秘密鍵はローカル環境でFirebaseにアクセスするときに使う。

##### Ⅱ. DBを作る
コンソール画面>構築>Firestore Database>データ

DBの構成はコレクション>ドキュメント>フィールド(辞書型)になっている。

今回は、users>hogehogeというドキュメントに
+ OAuthトークン（後述）
+ twitch クライアントID
+ twitch 秘密鍵
+ eventSub秘密鍵（後述）

を収納する。

また、users>subsというドキュメントに通知してほしいtwitchチャンネルのuser idを収納する。（user idは整数で構成されている。つまりアドレスで表示されているuser_loginとは別に存在するためAPIを叩いて調べる必要がある）key=user_login、value=user_idで収納する。

![photo020_01](https://simulacre-9to.netlify.app/images/020_01.png)

### 3. GCPでCloud Functionsを作成する(サブスクライバ編)
コーディングするだけ。実際のコードは[これ](https://github.com/9-to/twitchBot_GCP)を見てください。

#### やること
+ DBからtwitchの鍵を受け取る
    + この鍵はtwitchのクライアントIDとクライアント秘密鍵のこと
+ Oauthトークンを取得する
    + OauthトークンとはAPIで通信するための鍵のこと
    + 一定期間で有効期限が切れる
+ Oauthトークンの有効性を確認する
+ 現在のEventSubの一覧を確認する
+ 現在のEventSubを全て取り消す
+ 視聴したいチャンネルのEventSubを申請する

#### 気をつけること
+ トークンを生成するときと使用するときとで、twitchのクライアントIDの表記が異なる
    + `https://id.twitch.tv/oauth2/token`にアクセスするとき：`client_id`
    + `https://api.twitch.tv/helix/eventsub/subscriptions`にアクセスするとき：`client-id`
+ Oauthトークンを使用するときは頭に`Bearer `を付ける
+ GCPで動かす関数は引数が2つあるので用意しておく

### 4. GCPでCloud Functionsを作成する(バックエンド編)
#### やること
+ シグネチャから通信の信頼性を検証する
+ HTTPヘッダの`Twitch-Eventsub-Subscription-Version`の種類を確認する
+ EventSubで受信したチャンネルの情報を収集する
+ discordに送信する

#### 気をつけること
+ シグネチャの計算方法
    + hmacsha256を使っている
    ```python
    def getHmacMessage(request):
    #メッセージを生成
    a = request.headers['Twitch-Eventsub-Message-Id']
    b = request.headers['Twitch-Eventsub-Message-Timestamp']
    c = request.data.decode('utf-8')
    return (a + b + c)
    ```
    ```python
    EXsignature = hmac.new(bytearray(keys['sub_secret'], "ASCII"), bytearray(message, "ASCII"), hashlib.sha256).hexdigest()
    if isVaildSignature(TWsignature,"sha256="+EXsignature) == False :
        return Response(response=body['challenge'],headers={'content-type':'text/plain'}, status=500)
    ```
+ GCPで動かすときの関数は引数が1つ必要
    + この引数はflaskのrequest(**requests**ではない)オブジェクト
+ GCPで動かすときの関数の戻り値としてflaskのresponseオブジェクトが必要

<br>

## 雑感
Google Cloud Platformを使うのが初めてで苦労した。googleアナリティクスの時も思ったけれどUIとレスポンスが悪すぎる。

感想としては、常時動くアプリをサーバやVPSを借りずとも作れるものなんだな、と感心した。

自分の作りたかったものが作れたので満足している。

適当に書いたので足りない部分が多いと思うが、覚書くらいにはなるだろう。