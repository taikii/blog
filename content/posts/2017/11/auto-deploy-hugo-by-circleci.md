---
title: "Auto Deploy Hugo by Circle CI 2.0"
date: 2017-11-01T22:23:47+09:00
tags: ["Hugo", "Circle CI", "TIL"]
series: ["Hugo 構築"]
---
以前、[Hugoでブログサイトを作る](/posts/2017/10/build-site-by-hugo)記事を書きましたが、Circle CI 2.0を使って自動デプロイできるようにしたので、自分のやったことについてまとめます。
<!--more-->

## 前提として
私はCircle CIについてほとんど知識がありません。また、業務でCIを触っていないので、CIそのものに対する知見も狭いです。この状態でネット上の記事を切り貼りして書いていますので、間違っている箇所もあるかもしれません。その際はご指摘いただけると幸いです。

## Circle CI 2.0でのHugo自動デプロイ
Ciecle CIでHugoのビルドをして自動でデプロイする方法について、インターネット上にはいくつかの記事がありますが、現時点で多くがCircle CI 1.0ベースのものです。どうせなら今年7月に正式リリースされた2.0ベースでやりたいと思います。

今回は、GitHub Pagesのユーザページサイトを作ることを前提に考えます。ユーザページサイトのリポジトリにはHugoでビルドして生成したHTMLファイルのみがコミットされているとし、ビルド前のファイルは別のリポジトリ（blogリポジトリとします）があるものとします。ざっと以下のようなフローで自動デプロイを行います。

1. PCでblogリポジトリに記事を作成
2. コミット＆Push
3. Circle CI：GitHub上のblogリポジトリへのPushを検知
4. Circle CI：自身にblogリポジトリを`git clone`
5. Circle CI：ユーザページサイトのリポジトリを`public`フォルダに`git clone`
6. Circle CI：`hugo`する
7. Circle CI：`public`フォルダをコミット＆Push
8. ユーザページサイトが更新されている

### Circle CI 2.0の設定ファイルを作成する
Circle CI側の設定はあとでやるとして、まずblogリポジトリにCircle CI 2.0の設定ファイルを作成します。

※Circle CI 1.0と2.0では設定ファイルの書き方が変わっていますので、1.0の設定ファイルでは2.0ベースで実行することはできません。（このためインターネット上に多く転がっているCircle CI 1.0でHugoの自動デプロイする記事をそのまま利用できません）

ちなみに、1.0ではCircle CI側で用意されているUbuntu上でビルドを行っていましたが、2.0ではDockerコンテナ上でビルドを実行するようになりました。あらかじめテストやビルド用のソフトウェアやライブラリが導入されたDockerイメージを準備しておけるようになり、実行環境の準備が容易になったり、実行時間の短縮にもなっているそうです。このように1.0と2.0ではかなり環境が変わっているため設定ファイルのフォーマットも全く違っています。

しかし、ありがたいことに、Circle CIの中の人がHugoのビルド用Dockerイメージを作成してくれています。ありがたくFelicianoさんのDockerイメージを使用することにします。

[Introducing Docker Hugo - a CircleCI 2.0 Ready Docker Image - Community / Projects - CircleCI Community Discussion](https://discuss.circleci.com/t/introducing-docker-hugo-a-circleci-2-0-ready-docker-image/12420)


Circle CI 2.0ではリポジトリのルードディレクトリに`.circleci/config.yml`というファイルを作成します。（1.0の設定ファイルは`circle.yml`でした）

こんなんでいいんだろうか・・・

```yaml
version: 2
jobs:
  build:
    docker:
      - image: felicianotech/docker-hugo:latest
    steps:
      # blogリポジトリのデータを取得する
      - checkout

      # Gitの設定
      - run: git config --global user.name "Your name"
      - run: git config --global user.email "Your email"

      # テーマをサブモジュールのしている場合は最新化
      - run: git submodule sync
      - run: git submodule update --init --recursive

      # ユーザページサイトのリポジトリをpublicフォルダにcloneする
      - run: git clone ユーザページサイトリポジトリ public

      # ビルドする
      - run: hugo

      # HTMLProoferでテストする
      - run:
          name: "Test Website"
          command: htmlproofer public --allow-hash-href --check-html --empty-alt-ignore

      # ユーザページサイトにPushする
      - deploy:
          command: cd public && git add . && git diff --cached --exit-code --quiet || git commit -m "Rebuilding site" && git push origin master
```

### Ciecle CIの準備
Circle CIのアカウントを作成します。GitHubのアカウントでログインできるため、GitHubのアカウントでログインします。

https://circleci.com/

Circle CIにログインすると、自分のGitHub上のプロジェクトの一覧が表示され、セットアップしろと言われるので、blogリポジトリの右側に表示されている`Setup project`をクリックして設定画面に進みます。その後、Operating systemは`Linux`、Platformは`2.0`を選択して（どちらもデフォルト）、`Start building`をクリックします。

この時点ではCircle CIにユーザページサイトリポジトリへの書き込み権限を与えていないためビルドが失敗します。このため、Circle CIに書き込み権限を与える必要があります。

BUILDS画面の右上の歯車アイコンをクリックして設定画面を開きます。`PERMISSIONS` > `Checkout SSH keys` と選択し、`Add User key`にある `Create and Add User key`ボタンをクリックします。GitHubの認証画面が表示されるので認証を行えば、権限付与の完了です。

これで何か記事を`blog`リポジトリにPushすると自動的にユーザページサイトにデプロイされるはずです。

かーんたーん！