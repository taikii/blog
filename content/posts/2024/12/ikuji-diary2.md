---
title: "育児日記のハードルを下げたい"
date: 2024-12-03T00:00:00+09:00
tags: ["Bluesky", "GitHub Actions"]
series: []
---

[子育てエンジニア Advent Calendar 2024](https://adventar.org/calendars/10811) 3日目の記事です。

<!--more-->

育児日記って続かないですよね？私は続かないです。

でも、子供は「今」が一番小さくて、「今」が一番かわいいです。だから、「今」の些細なことを残したい。育児日記を書くのが億劫で大切な記憶が消えていくなんてもったいない。なんとかして情報を残すハードルを、育児日記を書くハードルを下げたい。

そう思って、Slackで一行メモを書いて、それを GitHub Actions で Markdown ファイルとして残すようにしています。Markdownにさえなっていれば、GitHub上でもそれなりに見えますし、ZennでもSSGでもなんとでもできます。

Slack からの連携方法は [育児日記をSlackとGitHub Actionsでつけている話]({{< relref "/posts/2023/11/ikuji-diary.md" >}}) を見ていただければと思います。

でも、それでも漏れてしまうものがあって。それは「グチ」です。ネガティブなものも多いけど、きっと数年後に見たら笑い話になってるものも多いはず。

私は日頃Blueskyで呟いていて、先日 [BlueskyのAPI叩くCLIを作った]({{< relref "/posts/2024/02/bsky-sh.md" >}}) ので、これを使ってSlackと同じようにブルスコのグチをMarkdownとして残すようにします。ネガティブな話題が多い気がするので、Slackのものとは別のファイルにまとめるようにしました。

自分のポストから「子供」「長男」「次男」のキーワードで検索しています。

## GitHub Actions の内容

Slackと同じようにGitHub Actionsのシークレット変数を登録します。

* `BSKY_HANDLE` : Blueskyのハンドルです。
* `BSKY_PASSWORD` : Blueskyの公式で発行するApp passwordです。

`.github/workflows/bsky2md.yml`

```yaml
name: Bluesky Conversation to Markdown

on:
  schedule:
    - cron:  '15 0 * * *'
  workflow_dispatch:

jobs:
  main:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Get bsky.sh
        run: |
          curl -s -o ~/bsky.sh https://raw.githubusercontent.com/taikii/bsky-sh/main/bsky.sh
          chmod a+x ~/bsky.sh

      - id: count_messages
        env:
          BSKY_HANDLE: ${{ secrets.BSKY_HANDLE }}
          BSKY_PASSWORD: ${{ secrets.BSKY_PASSWORD }}
        continue-on-error: true
        run: |
          [[ 0 -lt $({ ~/bsky.sh search "from:me since:$(date '+%Y-%m-%d' --date '2 days ago')T15:00:00Z until:$(date '+%Y-%m-%d' --date '1 days ago')T15:00:00Z 子供"; ~/bsky.sh search "from:me since:$(date '+%Y-%m-%d' --date '2 days ago')T15:00:00Z until:$(date '+%Y-%m-%d' --date '1 days ago')T15:00:00Z 次男"; ~/bsky.sh search "from:me since:$(date '+%Y-%m-%d' --date '2 days ago')T15:00:00Z until:$(date '+%Y-%m-%d' --date '1 days ago')T15:00:00Z 長男"; } | wc -l) ]]

      - if: ${{ steps.count_messages.outcome == 'success' }}
        env:
          BSKY_HANDLE: ${{ secrets.BSKY_HANDLE }}
          BSKY_PASSWORD: ${{ secrets.BSKY_PASSWORD }}
        run: |
          echo "---" > "$(date '+%Y-%m-%d' --date '1 days ago')_bluesky.md"
          echo "title: \"$(date '+%Y/%m/%d' --date '1 days ago')\"" >> "$(date '+%Y-%m-%d' --date '1 days ago')_bluesky.md"
          echo "date: $(date '+%Y-%m-%d' --date '1 days ago')T23:59:59+09:00" >> "$(date '+%Y-%m-%d' --date '1 days ago')_bluesky.md"
          echo "---" >> "$(date '+%Y-%m-%d' --date '1 days ago')_bluesky.md"
          echo >> "$(date '+%Y-%m-%d' --date '1 days ago')_bluesky.md"
          { ~/bsky.sh search "from:me since:$(date '+%Y-%m-%d' --date '2 days ago')T15:00:00Z until:$(date '+%Y-%m-%d' --date '1 days ago')T15:00:00Z 子供"; ~/bsky.sh search "from:me since:$(date '+%Y-%m-%d' --date '2 days ago')T15:00:00Z until:$(date '+%Y-%m-%d' --date '1 days ago')T15:00:00Z 次男"; ~/bsky.sh search "from:me since:$(date '+%Y-%m-%d' --date '2 days ago')T15:00:00Z until:$(date '+%Y-%m-%d' --date '1 days ago')T15:00:00Z 長男"; } | sort -t $'\t' -k 2 | uniq | cut -f 4 | sed -e 's/$/\'$'\n/' >> "$(date '+%Y-%m-%d' --date '1 days ago')_bluesky.md"

      - if: ${{ steps.count_messages.outcome == 'success' }}
        env:
          GIT_AUTHOR_NAME: "github-actions[bot]"
          GIT_AUTHOR_EMAIL: "github-actions[bot]@users.noreply.github.com"
          GIT_COMMITTER_NAME: "github-actions[bot]"
          GIT_COMMITTER_EMAIL: "github-actions[bot]@users.noreply.github.com"
        run: |
          git add .
          git commit -m "Add files"
          git pull
          git push origin main
```

そんな感じです。
