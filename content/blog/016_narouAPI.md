+++
title = "小説家になろうAPIを使って更新通知botを作った"
date = 2022-02-17
[taxonomies]
tags =["programing"]
+++

驚くべきことに、小説家になろうにはAPI機能がある。

<!-- more -->

具体的には[なろう小説API](https://dev.syosetu.com/man/api/)として用意されている。これを利用するとデータサイエンス的なことがゴニョゴニョできたり、更新を取得して通知botを作れたりする。

---

今回はこれを使ってdiscordに更新通知を流すbotを作った。

一応、公式で更新通知機能は用意されており、有志によってメール通知サービスが開発されていたりもするが、今どきメールで通知されても旨みがないし、毎日サイトにアクセスして確認するのもだるい。そこで、discordに通知を送信するbotを作ることにした。

botの仕様として、小説情報から最新部分掲載日をスクレイピングする仕様も考えたが、短時間に連続してサイトにアクセスすると制限が掛かるらしい。そのため、APIを利用するのが丸いと思う。

なろうAPI→GAS→webhookという流れで構築。

---

### クエリ

`https://api.syosetu.com/novelapi/api/?ncode=n0776dq&of=gl-n&out=json`

大好きな作品の一つ、『ギスギスオンライン』を例にする。

+ `ncode=n0776dq`でギスギスオンラインの小説情報を取得する。
+ `of=gl-n`で小説情報の小説コードと最終掲載日を取得する。
+ `out=json`で取得する型をjson形式に指定する。

公式ドキュメントが日本語なのでとてもわかりやすい。母国語のドキュメントを読む平易さを考えると、英語圏のデベロッパーは楽してるなと思う。

### GAS

例によって例の如く、google App Scriptを使用する（楽だから）。

[Discordに通知するRSSチェッカー](https://simulacre-9to.netlify.app/blog/005-feed/)のコードを流用した。また、[この記事](https://kokuyokugetter.github.io/post/web_comic_notification/)も参考にした。

```javascript
function notify_update_narou(sheetID) {
  const {range, lastrow} = readRange(sheetID,'narou');
  //日付の取得
  var LIMIT_TIME = 24*60*60;
  var NOW_UNIX_TIME = Math.floor((new Date().getTime())/1000);

  for(var i = 1; i <= lastrow - 1; i++){
    var l = range.getValues()[i];
    var url = l[1];
    var data = UrlFetchApp.fetch(url);
    var json = JSON.parse(data);
    var lastup = json[1].general_lastup;
    var update_time = new Date(lastup);
    var updateUnixTime = Math.floor(update_time.getTime()/1000);
    Logger.log(l[0]);
    Logger.log(json[1].general_lastup);

    if (NOW_UNIX_TIME - updateUnixTime < LIMIT_TIME) {
      title = l[0];
      pubDate = update_time;
      url = "https://ncode.syosetu.com/novelview/infotop/ncode/" + json[1].ncode;
      siteTitle = "小説家になろう";
      sendDiscord(title,pubDate,url,siteTitle);
    }
    Utilities.sleep(1000);
  }
}
```

`readRange()`関数や`sendDiscord()`関数、googleスプレッドシートの用意は上記の記事を参考にしてください。

これを漫画更新通知botと並行して、1日一回走らせるといい感じに通知が届く、はず。

---

### まとめ

暮らしが少し楽になった。

漫画更新通知botの方は静的サイトにしか有効じゃないのでどうにか改良したいと考えている。