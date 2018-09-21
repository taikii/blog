---
title: "Docker LibraryのPostgreSQLで日本語がうまくソートされない"
date: 2018-09-21T23:48:03+09:00
tags: ["Redmine", "Docker", "Postgres"]
---

私はRedmineをDocker Libraryのイメージで立てており、DBには同じくDocker LibraryのPostgreSQLを使用しています。以前はMuSQLを使ってたんですが、PostgreSQLに移行してからソートしても日本語がうまくソートされない問題が発生するようになりました。

<!--more-->

調べたところ、照合順序（`lc_collate`）が適切でないとこの問題が発生するとのこと。以下のページを参考にさせていただきました。

[PostgreSQLで並び順がおかしい時の修正方法](http://se.ykysd.com/2015/09/28/orderby/)

`lc_collate`が`C`だと文字コードのバイナリ順、`ja_JP.UTF-8`だと辞書順でのソートになるようですね。ポスグレコンテナに入って確認してみます。

ポスグレコンテナへの入り方はこんな感じ。

```text
$ docker exec -it redmine_db_1 /bin/bash
```

ポスグレコンテナに入ったら、`postgres`ユーザにスイッチしてから`psql`を叩きます。

```text
$ su - postgres
$ psql
postgres=# select name, setting, context from pg_settings where name like 'lc%';
    name     |  setting   |  context
-------------+------------+-----------
 lc_collate  | en_US.utf8 | internal
 lc_ctype    | en_US.utf8 | internal
 lc_messages | en_US.utf8 | superuser
 lc_monetary | en_US.utf8 | user
 lc_numeric  | en_US.utf8 | user
 lc_time     | en_US.utf8 | user
(6 rows)
```

`lc_collate=en_US.utf8`ですね。[Docker LibraryのPostgreSQL](https://hub.docker.com/_/postgres/)のデフォルトロケールは`en_US.utf8`みたいです。

これを`C`か`ja_JP.UTF-8`に変更する必要があるのですが、このポスグレのイメージには`ja_JP.UTF-8`が入っていません。`ja_JP.UTF-8`をインストールするのも面倒な話なので、特別な理由が無い限り`C`でいいんじゃないでしょうか。Linuxの`sort`コマンドとかも同じ文字コード順だったはずなので馴染みが無い訳ではないと思います。`ja_JP.UTF-8`がよければ`Dockerfile`を作ってどうのこうのする必要があります。

ではこの`lc_collate`をどうやって変更するかというと、これは`initdb`の`--locale`オプションで指定するものとのこと。

しかし、`initdb`はデータベースクラスタを作成する際の初期化コマンドですので、何も考えずに実行すると綺麗さっぱりデータがなくなっちゃいます。普通にインストールしたポスグレであれば、上記のページにあるようにダンプを取ってリストアとなるんでしょうが、こちとらDockerコンテナですので、そうは問屋がおろさない。クラスタを落とした時点でコンテナが落ちちゃう気がするし。試してないですけど。

とりあえず`initdb`で指定するのはわかったとして、このDockerコンテナに対してどうやってこのオプションを渡すのかという話なんですが、[Docker Hubのページ](https://hub.docker.com/_/postgres/)を確認すると、`POSTGRES_INITDB_ARGS`という環境変数でinitdbのオプションを渡すことができるようです。まぁ、initdb自体、まっさらな状態からコンテナを初めて作るときしか実行されないんでしょうけど。

つまり、`docker-compose.yml`ではこんな感じで指定することになります。

```docker-compose.yml
version: '2'

services:
  db:
    image: postgres:9.6
    environment:
      POSTGRES_INITDB_ARGS: "--encoding=UTF-8 --locale=C"
```

新しくRedmineを立てるのならこれで終わりです。しかし、私は既に運用を開始してしまっていますので、ここからは既存のポスグレのロケールを変更するのに四苦八苦していきます。

で、やっぱりダンプとってリストアしようかとも考えたのですが、Dockerとして運用しているポスグレにはなーんかそぐわない気がして。通常だとまっさらなポスグレにはRedmineがマイグレーションしてテーブルを作るわけですし。

そこで「これってMySQLからポスグレに移行するのと一緒やん？」と思いつきました。ポスグレ移行の際は以下のRedmine.JPの記事を参考にしたのですが、今回も同じようにyaml_dbを使って移行できるんじゃないかと。これなら文字コード関係ないし、実績あるし。

[Redmineで使うデータベースを変更する | Redmine.JP Blog](http://blog.redmine.jp/articles/change-database/)

そんなわけでこのような手順でやっていきます。もっとうまい方法はあると思いますけど許してください。

1. 兎にも角にもデータボリュームをバックアップ
2. Redmineにyaml_dbを入れる
3. yaml_dbでダンプを取得
4. ポスグレコンテナを初期化
5. lc_collate=Cクラスタの作成とRedmineのマイグレーション
6. yaml_dbでリストア

## ポスグレロケール変更大作戦

ではやっていきましょう。ちなみに、Redmineのバージョンは`3.4`、PostgreSQLは`9.6`を使っています。いずれもDocker LibraryのDockerイメージです。

### 兎にも角にもデータボリュームをバックアップ

です。後にyaml_dbでダンプを取得しますが、これはバックアップではありません。やっとかないと絶対後悔します。

また、ハマりどころを後述しますが、ここで[ゴミテーブルがある場合は予め削除しておく](#ゴミテーブルがある場合は予め削除しておく)ことをおすすめします。

### Redmineにyaml_dbを入れる

Redmine上のGemfileの適当なところに書きます。

Redmineコンテナへの入り方はこうですね。

```text
$ docker exec -it redmine_web_1 /bin/bash
```

Gemfileにはこんな行を追加したい。

```Gemfile
gem "yaml_db"
```

ここでRedmineコンテナにvimが入ってない問題が発生。わざわざvimを入れるのもアホくさいので`docker cp`コマンドでホストOSにGemfileをコピーして編集しました。

```text
$ docker cp redmine:/usr/src/redmine/ ./Gemfile
$ vim Gemfile
$ docker cp ./Gemfile redmine:/usr/src/redmine/
```

Gemfileの編集ができたので、改めて`bundle install`を実行してyaml_dbをインストールします。（Redmineコンテナで実行）

```text
$ bundle install --without development test
```

### yaml_dbでダンプを取得

yaml_dbがインストールできたのでダンプを取ります。（Redmineコンテナで実行）

```text
$ RAILS_ENV=production bundle exec rake db:data:dump
```

ダンプファイルは`db/data.yml`に出力されます。この後Redmineコンテナも再作成を行うので、永続化しているディレクトリにコピーしときます。（Redmineコンテナで実行）

```text
$ cp -pi db/data.yml files/
```

### ポスグレコンテナを初期化

ホストOSに戻って、コンテナを停止・削除します。これもハマりどころとして後述しますが、[無理にポスグレコンテナだけ作り直そうとせずRedmineコンテナも含めて削除](#postgresqlコンテナだけを削除しない)します。

```text
$ docker-compose stop
$ docker-compose rm -f
```

`docker-compose.yml`を編集します。前述の通り、`--locale=C`です。

```diff
   db:
     image: postgres:9.6
     environment:
+      POSTGRES_INITDB_ARGS: "--encoding=UTF-8 --locale=C"
```

それでは、`initdb`が実行されるように、**永続化されているポスグレのデータボリュームを削除**（またはmove）して空の状態にします。怖えぇぇー！！！

### lc_collate=Cクラスタの作成とRedmineのマイグレーション

改めてコンテナを作成します。ポスグレはデータボリュームが無くなっているので`initdb`が実行され、空のクラスタが`lc_collate=C`で作られる…はず。そしてRedmineはDBが空になっているのでマイグレーションしてくれる…はず。

```text
$ docker-compose up -d
```

### yaml_dbでリストア

Redmineコンテナが再作成されているので、もう一回Gemfileを配置～`bundle install`してyaml_dbをインストールします。（割愛）

先程永続化されたディレクトリに退避した`data.yml`を`db/`ディレクトリに戻して、ロードします。（Redmineコンテナで実行）

```text
$ mv -i files/data.yml db/
$ RAILS_ENV=production bundle exec rake db:data:load
```

これで`lc_collate=C`に変更できました！

```text
postgres=# select name, setting, context from pg_settings where name like 'lc%';
    name     | setting |  context
-------------+---------+-----------
 lc_collate  | C       | internal
 lc_ctype    | C       | internal
 lc_messages | C       | superuser
 lc_monetary | C       | user
 lc_numeric  | C       | user
 lc_time     | C       | user
(6 rows)
```

ブラウザでRedmineを見てもそれっぽくソートできるようになっているはずです。やったね！

## ハマりどころ

いくつかハマりどころがあったので解説。

### ゴミテーブルがある場合は予め削除しておく

私は以前ナレッジベースプラグインを使っていたのですが、これを削除した際に`VERSION=0`でマイグレーションしていなかったようです。このため、プラグインは存在しないのにテーブルが残留している状態となっていました。

この状態でyaml_dbでダンプを取ると、当然ナレッジベースプラグインのテーブルデータもエクスポートされますが、その後のマイグレーションではプラグインがないためテーブルは作成されません。そうすると、yaml_dbでデータをロードする際にテーブルが存在しないエラーが発生してしまいます。こうなると、もう一度プラグインをインストールしてテーブルを作るか、`data.yml`をシコシコ編集して該当データを削除する必要があります。

ですので、マイグレーションで作成されないテーブルが存在しないか、一度チェックしておくことをおすすめします。

チェック方法ですか？同じプラグイン配置された状態で空のRedmineインスタンスを立ててみて、DB上に作成されたテーブルを比較してみるとかですかね…。面倒ですがやったほうがいいと思います…。

### PostgreSQLコンテナだけを削除しない

ポスグレのデータを削除してコンテナを再作成する際、Redmineコンテナも作り直す必要なくね？と思ったのですが、以下のようにコンテナ間のリンクがおかしくなってしまう様です。Docker Networkを修正することで対処できるかもしれませんが、面倒ですしRedmineコンテナも作り直すが無難かなと思います。

```
$ docker exec -it redmine_web_1 /bin/bash
Error response from daemon: Cannot link to a non running container: /redmine_db_1 AS /redmine_web_1/db
```

