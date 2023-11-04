---
title: "育児日記をSlackとGitHub Actionsでつけている話"
date: 2023-11-02T22:45:13+09:00
tags: ["Slack", "GitHub Actions"]
---

育児の記録をなんらかの形で残したいとは思うものの、紙の日記やブログはどうしても続かないので、気が向いたときに個人Slackにつぶやいて、それを日次のGitHub ActionsでGitHubリポジトリにMarkdownファイルに落とし込んで残すようにしています。

<!--more-->

何でこんなことをやってるかというと、

* 紙の日記だと…
  * ネタに気づいた時に日記帳が手元になくて書けない。書こうと思った時には忘れてる。
  * なかなか続かない上、白紙ページが増えるとモチベーションが下がる。
  * さらに、もったいなくて買わなくなってしまう。
* ブログだと…
  * スマホで書けるブログサービスもあるものの、気づいたときにささっと書くのはやはり難しい。
  * カッチリした記事を書かないといけない脅迫観念にとらわれてしまう。でも、まとまった時間が取れないので結局書けない。
  * パブリックなブログだと身バレしないように気にしないといけない。そんなことに気を配りたくないし、フェイクを混ぜたくないし、何より思い出として残せるものにしたい。
  * 無料で非公開にできるブログサービスでなかなかいいものがない。
  * 現物を手元に残したい。（SaaSはサ終があるので信用できない）
* Slackなら…
  * プライベートSlackなら身バレ気にせず書ける。
  * つぶやき感覚でダラダラ書ける。
  * スマホさえあれば、その場でその時の気持ちを書ける。
  * デメ：まとまってる感がない。
  * デメ：90日でアーカイブされる。

そして、Slackのデメリットを補うために、GitHubのプライベートリポジトリにMarkdownファイルとして残していて、それをGitHub Actionsのデイリー実行でやっています。

Markdownとして保存されていれば、GitHub上でプレビューできたり、やろうと思えばSSGでブログっぽく見せることも可能ですし。

## Slack API設定

[Slack API: Applications | Slack](https://api.slack.com/apps) で `Create New App` ボタンをクリックして、目的のワークスペースを選択します。

`OAuth & Permissions` をクリック、 `Scopes` で権限を設定します。

`Bot Token Scopes` はBotとして動く場合の権限、 `User Token Scopes` はユーザとして動く場合の権限のようです。今回の用途であれば`User Token Scopes` でいいかと思って、私はそちらで以下の権限を設定しています。

* `channels:history` - 投稿を読むだけであれば、たぶんこれだけでいい
* `channels:read` - なんとなくあったほうがいいかなと思って付けた権限
* `files:read` - 写真も引っ張ってくれば楽ちんかなと思って付けた権限
* `reactions:read` - リアクションで何かしようとしてつけた権限（忘れた）
* `users:read` - 複数名で投稿するときに、誰の投稿かわかるようにまとめるのであれば必要かな

権限を設定したあと `Install to Workspace` ボタンをクリックしてワークスペースにインストールするとAPIキーが生成されます。

## GitHub Actions設定

まとめる先のプライベートリポジトリを作ります。

`Settings` ＞ `Secrets and variables` で変数とシークレットを登録します。全部シークレットでいいと思います。

* `SLACK_TOKEN` : 発行したトークンを `Authorization: Bearer xoxp-xxxxxxxxx-xxxx` の形式で設定おきます。これをHTTP Headerに乗せます。
* `SLACK_CHANNEL` : チャンネルIDを指定します。 `channels:read` をつけていれば `https://slack.com/api/conversations.list` の `id` で確認できますが、普通にURLとして見えるのでわかるでしょう。
* `GIT_EMAIL` : コミットユーザのEmail
* `GIT_NAME` : コミットユーザの名前

 `.github/workflows/slack2md.yml` みたいなファイルを作ります。日本時間9:15に定期実行するようにしていますが、負荷状況によってずれます。

* `https://slack.com/api/conversations.history` を見て前日の投稿があるか確認します。
* 投稿があれば、登校時間でソートして一つのファイルに出力してコミットします。

投稿がなければ `count_messages` がエラーに見えちゃうとか課題もあって、もっとうまいやり方はありそうですが、大体こんな感じでやってます。リポジトリへのコミットもちゃんとしたお作法がありそうな気がします。

```yaml
name: Slack Conversation to Markdown

on:
  schedule:
    - cron:  '15 0 * * *'
  workflow_dispatch:

jobs:
  main:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - id: count_messages
      env:
        SLACK_TOKEN: ${{ secrets.SLACK_TOKEN }}
        SLACK_CHANNEL: ${{ secrets.SLACK_CHANNEL }}
      continue-on-error: true
      run: |
        [[ 0 -lt $(printenv SLACK_TOKEN | curl -s -H @- "https://slack.com/api/conversations.history?channel=${SLACK_CHANNEL}&oldest=$(date +%s --date "$(date '+%Y-%m-%d' --date '1 days ago')T00:00:00+09:00")&latest=$(date +%s --date "$(date '+%Y-%m-%d')T00:00:00+09:00")" | jq '.messages | length') ]]

    - if: ${{ steps.count_messages.outcome == 'success' }}
      env:
        SLACK_TOKEN: ${{ secrets.SLACK_TOKEN }}
        SLACK_CHANNEL: ${{ secrets.SLACK_CHANNEL }}
      run: |
        echo "---" > "$(date '+%Y-%m-%d' --date '1 days ago').md"
        echo "title: \"$(date '+%Y/%m/%d' --date '1 days ago')\"" >> "$(date '+%Y-%m-%d' --date '1 days ago').md"
        echo "date: $(date '+%Y-%m-%d' --date '1 days ago')T23:59:59+09:00" >> "$(date '+%Y-%m-%d' --date '1 days ago').md"
        echo "---" >> "$(date '+%Y-%m-%d' --date '1 days ago').md"
        echo >> "$(date '+%Y-%m-%d' --date '1 days ago').md"
        printenv SLACK_TOKEN | curl -s -H @- "https://slack.com/api/conversations.history?channel=${SLACK_CHANNEL}&oldest=$(date +%s --date "$(date '+%Y-%m-%d' --date '1 days ago')T00:00:00+09:00")&latest=$(date +%s --date "$(date '+%Y-%m-%d')T00:00:00+09:00")" | jq -r '.messages | sort_by(.ts) | .[].text + "\n"' >> "$(date '+%Y-%m-%d' --date '1 days ago').md"

    - if: ${{ steps.count_messages.outcome == 'success' }}
      env:
        GIT_AUTHOR_NAME: ${{ secrets.GIT_NAME }}
        GIT_AUTHOR_EMAIL: ${{ secrets.GIT_EMAIL }}
        GIT_COMMITTER_NAME: ${{ secrets.GIT_NAME }}
        GIT_COMMITTER_EMAIL: ${{ secrets.GIT_EMAIL }}
      run: |
        git add .
        git commit -m "Add files"
        git pull
        git push origin main
```

どんなに簡易化しようと書かないものは書かないんですよね…
