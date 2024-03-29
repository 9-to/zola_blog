+++
title = "深夜に書く"
date = 2021-06-22
[taxonomies]
tags =["diary"]
+++

豚の角煮を作った。

<!-- more -->

### レシピ

1. 塩を3摘み入れて角切りにした豚バラ肉を1時間蓋をして中弱火で下茹でする。
2. 茹で汁:水：日本酒=4:4:3の割合で入れて、醤油大さじ4、砂糖大さじ4入れる。生姜も一塊輪切りにして入れる。（ここは好みの問題だが生姜を使った料理を作る予定がなければ使い切るのがフードロス的観点からも良いだろう。生姜を冷蔵庫に眠らせて半年後処理に困るよりも今使い切って生姜の効いた角煮を食べた方が幾分マシだ）
3. 蓋をして1時間、中弱火で煮込む。
4. **水分がなくなり鍋の中は悲惨なことになる。鍋底には醤油がダマになってこびり付き、油塗れになり、角煮には醤油が染みていない。**
5. うっかりシンクに油を流して排水口が詰まり、後処理のタスクが増えた事実を憂うことになる。
6. 油まみれになった食器や調理器具をお湯で温めながら洗う。この工程が一番面倒だ。

豚の角煮の味自体は悪くはない。タンパク質と脂質だから味付けが雑でも大抵美味しく食べられる料理になる。

敗因は、煮込む際の火を強くしすぎたからだろう。弱火、いっそチョロ火でもよかったかもしれない。

前回は白ごはん.comの角煮レシピを試したが、豚バラではなく赤みの多い豚肉（恐らく肩）で調理したため、非常に肉質な角煮になってしまった。やはり豚の角煮はバラ肉でコッテリとした角煮が食べたいものだ。今回はその反省を生かして調理できたので何はともあれよかった。豚肉であれば何でも良いわけではないのだ。

## 情報備忘録

[ABC203c](https://atcoder.jp/contests/abc203/tasks/abc203_c)：`pair`型を初めて使った。二次元配列をソートしたい時に頼っていきたい。

[ABC188c](https://atcoder.jp/contests/abc188/tasks/abc188_c)：問題を見たとき2分探索法が脳にチラついたがそんなことなかった。`*max_element(all(ar))`でベクトルの最大要素を取得。`max_element(all(al))-al.begin()`で最大要素のインデックスを取得できる。

競プロを敬遠してきたが、流石に無能なまま公言するのもダサいし避けては通れない道なので、過去問に目を通すくらいはやりたい。

競プロが苦手な理由：

- 初学者にとってライブラリ覚えゲーみたいな部分がある(アルゴリズムよりまずライブラリを教えた方が助かりそう)
- 原因のわからないWAで頭が沸騰する（しない）。そういう時に限ってしょーもないミスなので悔しくなって寝込む。