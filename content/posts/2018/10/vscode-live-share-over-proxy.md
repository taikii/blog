---
title: "VSCode Live Shareでプロキシを越える"
date: 2018-10-11T23:09:50+09:00
tags: ["VSCode"]
series: []
---

[YoshinoriN](https://twitter.com/yoshinorin24)さんがVSCode Live Share pluginがすごいと言ってるのを見て私もやってみたくなったわけですが、会社のプロキシ環境下ではリモートホストに繋がらねぇと怒られてしまいました。

<!--more-->

ログインはできているのですが、Shareボタンを押すと以下のエラーが発生します。

{{< figure src="https://gyazo.com/e59877074376e2582b490e88b536b1c9.png" >}}

```text
 Unable to connect to the remote server. TrackingId:********, Address:sb://vsls-prod-ins-euw-private-relay.servicebus.windows.net/********, Timestamp:2018/10/11 **:**:**
```

VSCodeの `Application > Proxy` にはプロキシを設定しているんですが。。。

調べたところ、あまり新しくはないですが、VSCodeのプロキシ設定見てない的なIssueを発見。

> [Users may be unable to connect to Live Share services via a proxy · Issue #86 · MicrosoftDocs/live-share](https://github.com/MicrosoftDocs/live-share/issues/86)

環境変数 `HTTP_PROXY` と `HTTPS_PROXY` を見てるっぽいので、（私はWindows機なので）ユーザー環境変数にこれらを設定しました。ちゃんとログオフ→ログオンし直して確認すると、見事繋がるじゃないですか！ヤッター！！

ちなみに、VSCodeの `Application > Proxy` 設定は、空欄だと環境変数 `HTTP_PROXY` と `HTTPS_PROXY` を見に行くので、この設定をする必要は無くなります。わざわざ消すこともないですが。

### Live Shareの感想

全員で議事録を書ける！（宝の持ち腐れ）

