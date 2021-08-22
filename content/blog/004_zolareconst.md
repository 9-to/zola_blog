+++
title = "改修"
date = 2021-07-10
+++

ブログを改装している。

<!-- more -->


### やりたいこと


* ~~ページングのページ送りをもっと使いやすくする~~

前回の改修でうまくいった。`pagenator`クラスを弄るといい感じになる。

* ~~一覧に概要を載せたい~~

これはmd内の`<!--more -->`コメントアウトで解決した。

* ~~ページングの挙動（表示の順序）がおかしい~~

`{% for page in page_set | reverse %}`を`{% for page in page_set%}`にしたら治った。

当たり前と言えば当たり前だが……。

* ~~About部分の改造~~
* 記事のタグ付け

<br><br><br>


以下参考。

[https://glatan.vercel.app/blog/started-a-zola-site/](https://glatan.vercel.app/blog/started-a-zola-site/)