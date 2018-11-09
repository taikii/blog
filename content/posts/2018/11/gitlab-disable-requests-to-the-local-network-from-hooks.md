---
title: "GitLab: ローカルを指した外向きWebhookはデフォルト禁止（10.6以降）"
date: 2018-11-10T00:09:26+09:00
tags: ["GitLab"]
series: []
---

Redmine上のGitリポジトリ更新されてなくない…？という事象が発生。

[Redmine GitHub Hook Plugin](https://github.com/koppen/redmine_github_hook)を使って、GitLabにPush/Mergeが発生したらRedmineコンテナ上のBareリポジトリをfetchさせてるハズなんですが…。

<!--more-->

GitLabのIntegrationを確認してみると…ああ！？コケとる！

```text
Hook execution failed: URL 'http://redmine_host/github_hook?project_id=redmine_prj&repository_id=redmine_repo' is blocked: Requests to the local network are not allowed
```

ぐぐってみると、ありました。`10.6`でローカルを指した外向きWebhookがデフォルトで禁止になったようです。元々存在した設定のデフォルト値が変わったってことなのかな？Issueも上がっていたようです。`10.6`は2018年3月リリースですので、半年間失敗し続けてたんですねぇ…orz

[New "Allow requests to the local network from hooks and services" should be ENABLED by default (#45134) · Issues · GitLab.org / GitLab Community Edition · GitLab](https://gitlab.com/gitlab-org/gitlab-ce/issues/45134)

> To prevent this type of exploitation from happening, starting with GitLab 10.6, all Webhook requests to the current GitLab instance server address and/or in a private network will be forbidden by default. That means that all requests made to 127.0.0.1, ::1 and 0.0.0.0, as well as IPv4 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 and IPv6 site-local (ffc0::/10) addresses won’t be allowed.
> 
> （[Webhooks and insecure internal web services | GitLab](https://docs.gitlab.com/ee/security/webhooks.html)）

確かに `Admin Area > Settings > Network > Outbound requests > Allow requests to the local network from hooks and services` のチェックが入っていませんでしたので、このチェックを入れてテストすると…

```text
Hook executed successfully: HTTP 200
```

(๑•̀ㅂ•́)و✧

一応補足すると、ローカルを指した外向きWebhookはGitLab自身に破壊的なナニガシがアレなのでキケンキケーン、ということのようですね。私はどちらも同じホスト上にDockerコンテナとして立ててしまっているので、まぁしょうがないかな。

いやー、なんというか、長い間気づいてなかった私もだいぶ悪いんですが…Redmineのリポジトリ、使われてないのね…。Issueの横っちょに関連コミットが表示されるんだけどな…。
