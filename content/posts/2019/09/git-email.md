---
title: "Gitの user.email に実在するメールアドレスを設定してみるテスト"
date: 2019-09-09T00:47:57+09:00
tags: ["Git"]
series: []
---

私はこれまでGitの `user.email` は [GitHubのnoreplyアドレス](https://help.github.com/ja/articles/setting-your-commit-email-address) を使っていたのですが、思うところがあって実在するメールアドレスに変更しました。

<!--more-->

* GitHubだけで完結できればいいけど、GitLab.comやBitBucketにForkされている場合にそれぞれのサービスにある自分のアカウントと紐付けられない。（GitLab.comやBitBucketで `*@users.noreply.github.com` は登録できないので）
* そもそもGitは分散リポジトリなのでGitHub固有のサービスに依存する運用はするべきではないのでは。
* 将来、GitHubのユーザ名を変えたくなるかもしれないし。（現在は `id+username@users.noreply.github.com` というemailでコミットされていれば、ユーザ名を変えてもコミットとの紐づきは維持されそうです。 [GitHubのユーザー名を変更するとnoreplyメールアドレスのコミット履歴はどうなるのか？ - Qiita](https://qiita.com/the_red/items/240fdea1a132b7843afc) が参考になりました。ありがとうございます。）

ただ、コミットにメールアドレスと入れるということはメアドを全世界に公開することでもあり、つまりスパムメールが来ることになるわけで。現代のスパムフィルタが優秀とはいえ、やっぱりスパムは少ないほうがいいし、実際どのくらいスパムがくるのか検討がつかないので、Gitコミット専用のメアドを発行することにしました。

新しくどこかにアカウントを作るのは面倒だったので、Outlook.comでエイリアスを作ることにしました。Gmailの「エイリアス」はアドレスのローカル部の後ろに `+hogehoge` のようにプラス記号と任意の文字列をくっつけることができるものですが、これはあくまでフィルタリングをやりやすくするためのもので、自分のメールアドレスを隠蔽することはできません。それに対してOutlook.comは全く新規のアドレスをエイリアスとして発行することができます。たぶんExchangeのセカンダリアドレスですね、`smtp`が小文字のやつ。確か、プライマリアドレス含めて10個まで作れたと思います。

[Outlook.com でメール エイリアスを追加または削除する - Outlook](https://support.office.com/ja-jp/article/outlook-com-%E3%81%A7%E3%83%A1%E3%83%BC%E3%83%AB-%E3%82%A8%E3%82%A4%E3%83%AA%E3%82%A2%E3%82%B9%E3%82%92%E8%BF%BD%E5%8A%A0%E3%81%BE%E3%81%9F%E3%81%AF%E5%89%8A%E9%99%A4%E3%81%99%E3%82%8B-459b1989-356d-40fa-a689-8f285b13f1f2)

もし、スパムフィーバーするようだったら、このエイリアス宛のメールは全部捨てるか、`users.noreply.github.com` に戻してエイリアス自体を削除するか、考えたいと思います。

つか、GitHubに日本語のヘルプができてることが衝撃…