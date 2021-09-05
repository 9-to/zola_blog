+++
title = "discordと連携するRSSリーダー"
date = 2021-08-23
+++

「人様のブログを読みたい！」という欲求はここ半年で急激に強まりつつあった。

<!-- more -->

しかし、インターネットに散らばった、更新されているかもわからないブログ群を毎日巡回するのは時間の無駄で、そのようなルーチンを日課に加えるのには些か抵抗があった。

RSSリーダー(例えばFeedly)に頼ればいいのだがあまりこういうサービスを使いたくないと個人的に思っている。理由は管理が面倒だから。

私はdiscordをよく見るのでwebhookでうまいことbotを作ればスマホでもパソコンでもブログ更新を確認できて嬉しいと思い、**魔術師の真似事** をすることになった。

---
用意する物

+ googleアカウント
    + Google Apps Scriptソースコード
    + Googleスプレッドシート
+ discordアカウント
    + webhook URL

IFTTTや類似のサービスを使えば簡単なのだが月額かかる場合があるのでGAS(Google Apps Script)でどうにかした。

インターネットの先駆者様の方々の記事を参考にキメラを作り上げた。以下がソースコード。

```Javascript
function setTrigger() {
 let setTime = new Date();//今日の月日を取得
 setTime.setHours(22);//22時
 setTime.setMinutes(00);//00分
 ScriptApp.newTrigger('myFunction').timeBased().at(setTime).create();//指定日時にmyFunctionのトリガーを作成
}

function delTrigger() {
  const triggers = ScriptApp.getProjectTriggers();
  for (const trigger of triggers) {
    if (trigger.getHandlerFunction() == "myFunction") {
      ScriptApp.deleteTrigger(trigger);//myFunctionのトリガーを削除
    }
  }
}

//main
function myFunction() {
  var LIMIT_TIME = 24*60*60;
  var NOW_UNIX_TIME = Math.floor((new Date().getTime())/1000);
  var my_rss = readRange("google spreadsheetのid", "A1:A21(任意の行列を指定)");//sheet idを入力
  for(var j = 0; j<my_rss.length; j++){
    var rss_data = UrlFetchApp.fetch(my_rss[j]);
    Logger.log(my_rss[j]);
    var rss_xml = XmlService.parse(rss_data.getContentText());
    var rss_entries = rss_xml.getRootElement().getChildren('channel')[0].getChildren('item');//itemの行列=entriesを取得
    var blogTitle = rss_xml.getRootElement().getChild('channel').getChild("title").getText();//ブログタイトルを取得
    for(var i = 0; i < rss_entries.length; i++){//rss_entries.length
      Logger.log(rss_entries[i].getChildText("title"));
      try{
        var pubDate = new Date(rss_entries[i].getChild("pubDate").getText());//RSS2.0の場合
      }catch(error){
        var pubDate = new Date(rss_entries[i].getChild("date").getText());
      }
      var pubDateUnixTime = Math.floor(pubDate.getTime()/1000);
　    if(NOW_UNIX_TIME-pubDateUnixTime<LIMIT_TIME){//もしitemの配信時刻と現在時刻の差が24時間以内だったら
        var title = rss_entries[i].getChild("title").getText();
        var url = rss_entries[i].getChild("link").getText();
        var description = rss_entries[i].getChild("description").getText();
        var content = '[{\n "title": '+title+',\n"url": '+url+',\n"description": '+description+'\n}]'//送信
        sendDiscord(title,pubDate,url,blogTitle);
      }
    }
  }
}

//discord webhookにメッセージを送信する
function sendDiscord(body, pubDate, url, blogTitle) {
  // discord webhookのURL
  const discordWebHookURL = "お求めのwebhook";
  // 投稿するチャット内容と設定
  const message = {
    "content": "**"+body+"     -     "+ blogTitle +"**\n"+pubDate,
    "tts": false,  // ロボットによる読み上げ機能を無効化
    "embeds": [{
      "title": body,
      "color": 1127128,
      "url": url
    }]
  }
  const param = {
    "method": "POST",
    "headers": { 'Content-type': "application/json" },
    "payload": JSON.stringify(message)
  }
  UrlFetchApp.fetch(discordWebHookURL, param);
}

//sheet id から rssURLのarrayを取得
function readRange(spreadsheetId, arr) {
  let spreadSheet = SpreadsheetApp.openById(spreadsheetId);
  let sheet = spreadSheet.getSheetByName("sheet1");
  let ranges = sheet.getRange(arr);//範囲を指定
  Logger.log(spreadSheet.getName());
  Logger.log(sheet.getName());
  Logger.log(ranges.getValues());
  return ranges.getValues();
}
```

これをgoogle driveのどこかにGoogle Apps Scriptとして保存してトリガーを追加する。

![photo005_1](https://simulacre-9to.netlify.app/blog/005_1.png)

巡回したいブログ一覧はgoogleスプレッドシートに保存すれば良い。

<br>

わからないことがあったら私のtwitterのアカウントにリプライを飛ばしてください。私も出来る限り頑張って「わかりません」と答えます。

---

これで、更新されたブログのログが決められた時間にdiscordに流れるようになった。快適。

しかし問題点もあって、このソースコードだとRSS1.0に対応しておらず、livedoor系やgoo系、niftyココログ系のブログの更新を捕捉できない。改善点だ。