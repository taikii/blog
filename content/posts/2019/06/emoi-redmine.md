---
title: "Redmineのエモい話"
date: 2019-06-03T00:15:00+09:00
tags: [Redmine]
draft: true
---

1年半前くらいに [Redmine is Dead]({{< relref "/posts/2018/01/redmine-is-dead.md" >}}) というエントリを書いたのですが、当時とは状況が変わってきていますし、今掘り起こされるとただの感じの悪い人になってしまう気がするので（笑）、その後のエモい感じのことを書きたいと思います。

<!--more-->

Redmine 4.0は多くの機能追加がありました。変化点については https://redmine.jp/ 等を見ていただたくとして、以前のエントリで挙げたことも改善されていたりします。どう考えても[@g_maeda](https://twitter.com/g_maeda)さんがコミッターになったことが影響を与えているように思います。ゴイスーっすね。

4.0に取り込まれていなくても、現在大きな変更が走ってます。これらが実現すればUIも機能的にも本当にモダンなシステムになりますね！ぜひ頑張ってほしいです！（他力本願）

* Opal -- Redmine TokyoでMaxさんがDiscordといっしょに紹介した、RedmineをモダンなUI/UXにしようというプロジェクト。どこで開発してるんでしょうか？
* [Feature #1448: Add tags to issues](https://www.redmine.org/issues/1448) -- タグ機能をRedmine本体に取り込もうという企画。
* [Feature #13919: Mention user on comment/description using @user with autocomplete](https://www.redmine.org/issues/13919) -- メンション機能をRedmine本体に取り込もうという企画。
* [Patch #23980: Replace images with icon fonts](https://www.redmine.org/issues/23980) -- アイコンをFont Awesomeにしようという企画。
* [Feature #30069: Integrate Redmine with GitLab (or other free CI system for open source) to run tests](https://www.redmine.org/issues/30069) -- テストをGitLab-CIにやらせる企画。
* [Redmine + GitBucketで実現する Redmineでプルリクエスト](https://speakerdeck.com/kounoike/redmine-plus-gitbucketteshi-xian-suru-redminetehururikuesuto) -- 先日のRedmine Tokyoで登壇された[@ko_noike](https://twitter.com/ko_noike)さんのGitBucketと連携することでPRを実現しようという企画。

これ以外にも私が知らないだけでいろいろありそう。ただ、ここに挙げたものでも最近少し停滞気味のもありそうなので、このエントリが誰かの目に止まって誰かのフォローが入ればいいなぁと思います。（完全他力本願）

## GitLabについて

最近はK8sに注力しているようで、IssueやWikiのようなプロマネ系の機能はあまり熱心ではない感じ。

External Issue TrackerでIssueやWikiはRedmine、GitリポジトリやマージリクエストはGitLabという使い方をしていますが、できれば統一したいという思いはあるものの、イマイチ完全に乗り換えられないでいます。まぁ今の運用でそこまで面倒に感じてもいないし、急ぐことはないんですが。

## まとめ

突貫制作プラグインシリーズをちょこちょこ出している通り、私自身ゴリゴリのRedmineユーザですので、どんどん便利になっていくのはすごく嬉しく思ってます。

[他にもいくつかやりたいことはある](https://scrapbox.io/taikiix-drafts/redmine)ので、暇を見つけてプラグインなりパッチなりを書きたいと思います。横取り40萬していただいてもいいです。（他力略）
