---
title: "みんな～！どうやってCIに変数を渡してるの？"
date: 2018-12-13T21:57:31+09:00
tags: ["CI","GitLab"]
series: []
draft: true
---

CIで自動テストをする際にDBなどの接続情報をどうやって渡してますか？

<!--more-->

GitLab-CIはDockerコンテナ上で実行されますよね。ビルドツールにGradleを使っているので[Docker LibraryのGradleイメージ](https://hub.docker.com/_/gradle/)を使ってCIを実行しています。もちろんローカルでもテストを実行しますので、ローカルでもGitLab-CI上でも同じようにテストをするために、以下のような環境にしています。

![](https://gyazo.com/45a6d567f0c3e7e436a2f5c4c8cccfb8/thumb/1000)

```mermaid
graph LR
  subgraph GitLab
    a["GitLab-CIのシークレット変数"]
    b["CIコンテナ上の~/.gradle/gradle.properties"]
  end
  subgraph PC
    c["~/.gradle/gradle.properties"]
  end
  subgraph Gradleの世界
    d("build.gradle")
    e["datasource.yml"]
  end
  a -- .gitlab-ci.yml --> b
  b --> d
  c --> d
  d --> e
```

シークレット変数はコンテナの環境変数として展開されるので、それを`gradle.properties`に吐いて、PC上と同じ環境を装う感じです。

`.gitlab-ci.yml`は以下のようになっています。

```yaml
before_script:
  - export GRADLE_USER_HOME=`pwd`/.gradle
  - mkdir -p ${GRADLE_USER_HOME}
  # gradle.propertiesが残ってたらクリア
  - echo -n > ${GRADLE_USER_HOME}/gradle.properties
  - printenv | grep "^MYSECRETS_" >> ${GRADLE_USER_HOME}/gradle.properties
```

試行錯誤の結果なんですが、みなさんどんな感じでやってるんでしょうかね？？？
