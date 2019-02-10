---
title: "GitLabにカバレッジバッジを貼る（GitLab CI+Gradle+JaCoCo）"
date: 2019-02-10T23:03:24+09:00
tags: ["GitLab", "Gradle", "JaCoCo"]
series: []
---

GitLabにカバレッジバッジを貼りたいと思い、試行錯誤した結果です。GitLabのバージョンは `11.6`です。カバレッジはGitLab CI上で実行されるGradleのJaCoCoプラグインのものを使用します。

カバレッジバッジが正常に表示されるまでには3つのステップがあります。

* GitLabにカバレッジバッジを貼る
* GitLab CIでカバレッジを取得する設定をする
* GradleのJaCoCoプラグインでカバレッジを出力する

<!--more-->

### GitLabにカバレッジバッジを貼る

通常、バッジは `README.md` に貼ることが多いですが、GitLabには [バッジを貼る専用の機能](https://docs.gitlab.com/ee/user/project/badges.html) があります（`10.7`以降）。バッジの設定はプロジェクトまたはグループの **Settings > General > Badges** で行います。グループに設定するとグループ配下のプロジェクトすべてに適用されて便利です。私もグループに設定しています。

バッジの設定は「バッジ画像のURL」と、「バッジをクリックした際に飛ぶリンク先URL」を記述するだけです。また、URLにはいくつかのプレースホルダを使うことができます。詳しくは [Placeholders](https://docs.gitlab.com/ee/user/project/badges.html#placeholders) を参照してください。

GitHubでカバレッジバッジを貼る場合は[Coveralls.io](https://coveralls.io/)や[Codecov.io](https://codecov.io/)が有名ですが、[GitLab CIには予め「ビルドステータス」と「カバレッジ」の2つのバッジが用意されています](https://docs.gitlab.com/ee/user/project/pipelines/settings.html#pipeline-badges)。

GitLab CIのカバレッジバッジ画像のURLは以下のようになります。

```
https://example.gitlab.com/<namespace>/<project>/badges/<branch>/coverage.svg
```

先程のプレースホルダを当てはめると以下のようになるので、これをBadge image URLに設定します。（ホスト名は変更してね）

```
https://example.gitlab.com/%{project_path}/badges/%{default_branch}/coverage.svg
```

Linkはコミットページへのリンクでいいと思います。

```
https://example.gitlab.com/%{project_path}/commits/%{default_branch}
```

これでプロジェクトのオーバービューページにカバレッジバッジが表示されるようになりますが、ステータスは「unknown」となっています。カバレッジを表示するためにはGitLab CIにカバレッジを認識させる設定をする必要があります。

### GitLab CIでカバレッジを取得する設定をする

**Setting > CI / CD > General pipelines** の下の方に **Test coverage parsing** という項目があります。この項目にジョブのログからカバレッジ結果を抽出する正規表現を記述します。（ジョブのログとはテスト実行時に標準出力に出力される内容です。）

この項目の下にはメジャーな言語やフレームワークでの記述例が書かれているので、基本はそれを踏襲すればOKです。なんですが、GradleのJaCoCoプラグインはここに書かれている `Total.*?([0-9]{1,3})%` という出力をしてくれません。

### GradleのJaCoCoプラグインでカバレッジを出力する

そのもので大変恐縮ですが、[JaCoCoのカバレッジをコンソールに出力する - Qiita](https://qiita.com/oohira/items/d9dc72a0276834243bfd) をそのままパクらせていただいて、カバレッジを出力するようにします。ちなみに `instruction`が命令網羅率、`branch`が分岐網羅率です。

ブランチカバレッジを採用するとして、上記の **Test coverage parsing** に記述する内容は以下のような感じになります。

```
^ - branch\s*:\s+(\d+\.\d+)%$
```

これでmasterブランチでテストジョブが実行されると、カバレッジバッジが適切に表示されていることが確認できると思います。
