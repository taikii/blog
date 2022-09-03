---
title: "Gitlab OmnibusのMattermostでHSTSが有効になった話"
date: 2022-09-04T01:05:00+09:00
tags: ["GitLab", "Mattermost"]
---

[GitLabにRCE脆弱性](https://news.mynavi.jp/techplus/article/20220826-2435525/) ニュースをきっかけに、重い腰を上げてGitLab Omnibusを 14.6 → 15.3.1 にアップデートしました。そうしたところ、GitLab Mattermost で HSTS（Strict-Transport-Security） が機能するようになってしまい、同じホスト上動いているHTTPのサービス（HTTPSではなく）にアクセスできなくなってしまう問題が発生しました。

これ、いつから…？（更新サボってるのが悪い）

<!--more-->

HSTSとは、サーバから `Strict-Transport-Security` というヘッダを返却すると、このヘッダを受け取ったブラウザは、そのWebサイトに対してHTTPSを強制するようになる機能です。

今回は、HTTPSで運用しているMattermostにアクセスすると `Strict-Transport-Security: max-age=63072000` （有効期限2年）が返却され、これによって同じホスト上でHTTPのみで運用しているGitLabにアクセスすると `307` でHTTPSにリダイレクトしてしまい、アクセスができなくなってしまいました。

結論から言うと、 `gitlab.rb` （または `GITLAB_OMNIBUS_CONFIG`） で `mattermost_nginx['hsts_max_age'] = 0` を指定することでHSTSを無効化できました。
手持ちの `gitlab.rb` には該当オプションがなかったけど、脈々と受け継がれてるファイルだからなぁ…最新化が必要かな？

* [Custom Mattermost nginx configuration params not picked up (#5371) · Issues · GitLab.org / omnibus-gitlab · GitLab](https://gitlab.com/gitlab-org/omnibus-gitlab/-/issues/5371)
* [Nginx: implement HSTS support in the mattermost configuration template (!6033) · Merge requests · GitLab.org / omnibus-gitlab · GitLab](https://gitlab.com/gitlab-org/omnibus-gitlab/-/merge_requests/6033/diffs)

また、通常のブラウザ設定メニューにはHSTSに関するものはないので、アドレスバーに `chrome://net-internals/#hsts` や `edge://net-internals/#hsts` を直打ちして設定画面を開き、ブラウザに保存されたHSTSの情報を削除する必要があります。以下参考にさせていただきました。ありがとうございます！

[HSTS が原因で、ウェブサイトが勝手にhttps接続しないようにする – ラボラジアン](https://laboradian.com/disable-hsts-for-domain/)

`Strict-Transport-Security: max-age=0` をブラウザに返せばブラウザに保存されたHSTSの情報を失効することができますが、GitLab Omnibusでは `mattermost_nginx['hsts_max_age'] = 0` を設定するとそもそも `Strict-Transport-Security` を返さなくなってしまいます。

> なお、 Strict-Transport-Security ヘッダーがブラウザーへ送られるたびに、そのウェブサイトに対する有効期限が更新されるので、サイトはこの情報を更新して期限切れを防ぐことができます。 Strict-Transport-Security を無効にする必要がある場合は、 HTTPS 通信時に max-age の値を 0 に設定することで Strict-Transport-Security ヘッダーが失効し、ブラウザーからの HTTP 接続が許されるようになります。

（[Strict-Transport-Security - HTTP | MDN](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Strict-Transport-Security#%E3%83%96%E3%83%A9%E3%82%A6%E3%82%B6%E3%83%BC%E3%81%A7%E3%81%AE%E6%89%B1%E3%81%84)）

上の方法で個別にブラウザからHSTSの情報を削除してもらったのですが、今考えれば `mattermost_nginx['hsts_max_age'] = 1` （1秒）を設定してしばらく運用すれば、ブラウザからHSTSの設定を一掃できましたね。それはそれで面倒な話か…

本来は、全部HTTPSにするべきなんですが、まぁその辺はお察しください…
