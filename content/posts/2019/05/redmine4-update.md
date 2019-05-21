---
title: "Redmine 4.0.3 にバージョンアップした話"
date: 2019-05-22T06:45:00+09:00
tags: ["Redmine"]
---

Redmineを `3.4` から `4.0.3` にバージョンアップしたのでそのメモです。

<!--more-->

私はオンプレ環境にDockerで建てています。 `docker-compose.yml` はこんな感じ。

{{< gist taikii 6523171ae846ebb2bc8d56d8d0477a94 >}}

Redmine自体のバージョンアップは、コンテナイメージのタグを `4.0.3` に変更して `docker-compose up -d` するだけです。ラクチン。BundlerやRailsの恩恵があるとはいえ、これだけでちゃんとメジャーバージョンアップされるのはすごい。改めてRedmineはキチンと作られてると感じます。

## プラグイン

問題はプラグインですね。これを機に使用をやめたりしたものもあります。

* [Persist wfmt](https://github.com/iquiw/redmine_persist_wfmt): メンテされていないのと、古いチケット以外ほぼMarkdown化できたので、バージョンアップを機にアンイストール。マイグレーションが必要なのでバージョンアップ前にアンインストールしました。
* [Mylyn Connector](https://github.com/danmunn/redmine_mylyn_connector): 誰も使ってなかったので、これもこの機会にアンイストール。
* [Github Style Fenced Code Block](https://github.com/taikii/redmine_github_style_fenced_code_block): 4.0で本体に取り込まれて不要になったのでアンイストール。
* [Rouge Highlighter](https://github.com/alexbevi/redmine_rouge_highlighter): これも4.0で本体に取り込まれて不要になったのでアンイストール。
* [Local Avatars](https://github.com/ncoders/redmine_local_avatars): オリジナルがメンテされていなっぽいので、[@taqueciさんのFork](https://github.com/taqueci/redmine_local_avatars) に変更。
* [Github Hook](https://github.com/koppen/redmine_github_hook): これもメンテされていないので、 [@eeaさんのFork](https://github.com/eea/redmine_github_hook) に変更。
* [Agile](https://www.redmineup.com/pages/plugins/agile): 問題があったのでとりあえずアンインストールしました。（後述）
* [Mentions](https://github.com/arkhitech/redmine_mentions): これも問題があったのでアンインストール。結構使ってたのでキツイ。（後述）
* その他プラグインを最新化
  * [Lightbox2](https://github.com/paginagmbh/redmine_lightbox2)
  * [Tags](https://github.com/ixti/redmine_tags)
  * [Default custom query](https://github.com/hidakatsuya/redmine_default_custom_query)
  * [Emoji](https://github.com/suer/redmine_emoji)
  * [Time logger](https://github.com/speedy32129/time_logger)
  * [Custom auto complete](https://github.com/taikii/redmine_custom_auto_complete)
  * [Checklists](https://www.redmineup.com/pages/plugins/checklists): Redmine UPから最新のものを入手。

### Agileプラグインについて

現時点の最新版 `1.4.10` を落としてきたのですが、チェックリストを表示するとエラーになってしまいました。

また、 `/users/new` `/users/:id/edit` `/projects/new` `/projects/:id/edit` が404になってしまいます。ルーティングがおかしくなっちゃうんですかね？Agileプラグインをアンインストールして問題は解消しています。

### Mentionsプラグインについて

コメント欄で`@ユーザ名`するとエラーになりました。メール送信周りに修正が必要っぽいですね。

オリジナルのリポジトリのものは元々JavaScriptに問題があったのでForkされたものを使わせてもらってたんですが、久しぶりに確認したらFork先のリポジトリがなくなってました。／(^o^)＼

Forkがたくさんあるので、メンテされてそうなやつを見繕って改めて導入しようと思います。

## その他

チケットコメントのアバターアイコンが左側欄外に飛び出してます。Local Avatarsだからでしょうか？

予想はしていましたが、メンテされてないプラグインがちらほらありますね。思うところはありますが…余計なことをいいそうなのでやめます。

とりあえずこれで `##チケット番号` とか `[[#セクションリンク]]` とかができるようになったわけですね！関わった方々お疲れ様でした＆ありがとうございます。ひとまず4.0ライフを楽しみたいと思いまーす。
