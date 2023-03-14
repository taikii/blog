---
title: "ムームードメインからCloudflareにドメインを移管した"
date: 2023-01-09T01:42:00+09:00
tags: ["Column"]
---

これまでこのサイトは、Circle CIでビルドしGitHub Pagesでホスティングしてドメインはムームードメインで管理していました。[ムームードメインが2月からサービス維持調整費をとるようになる](https://muumuu-domain.com/information/news/4901) というのと、同じく2月から `.net` の価格が上がるということで、いい機会なのでCloudflareにドメインを移管することにしました。ついでにホスティングもCloudflare Pagesに変更しました。

<!--more-->

## Cloudflare Pagesについて

最初からビルド構成に「Hugo」が定義されているので、GitHubのリポジトリへのアクセス権の設定をするだけで利用ができます。カスタムドメインを使わないのであれば無料枠だけで十分に使えます。すごい。

* デプロイは月 `500回` まで
* ファイル数は `20,000` まで
* ファイルサイズ `20MiB` まで
* Bandwidthの制限がない
* 広告も制限なさそう？

[Limits · Cloudflare Pages docs](https://developers.cloudflare.com/pages/platform/limits/)

ただし、デフォルトのHugoのバージョンが `0.54.0` と古く、そのままだとビルドできない可能性があります。環境変数に `HUGO_VERSION` を指定することで任意のバージョンのHugoでビルドすることができます。

[Build configuration · Cloudflare Pages docs](https://developers.cloudflare.com/pages/platform/build-configuration/)


## ドメイン移管

[ムームードメインからCloudflareへドメイン移管 – Toto Note](https://totonote.com/2022/02/20/domain-transfer-from-mumu-domain-to-cloudflare/) が大変参考になりました。ありがとうございます。

