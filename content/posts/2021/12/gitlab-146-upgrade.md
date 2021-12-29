---
title: "GitLab 13.12 から 14.6 へのアップグレードでコケた話"
date: 2021-12-30T07:43:00+09:00
tags: ["GitLab", "Mattermost"]
---

GitLab 13.12 で運用していたんですが、先日リリースされた [GitLab 14.6](https://about.gitlab.com/releases/2021/12/22/gitlab-14-6-released/) へのアップグレードを行った際にいくつかの障壁があったのでメモ。

アップグレードは [Upgrade paths](https://docs.gitlab.com/ee/update/#upgrade-paths) を元に実施しています。今回は以下のようなパスでアップグレードを実施しました。

```
13.12 -> 13.12.12 -> 14.0.11 -> 14.1.8 -> 14.6
```

私は [gitlab/gitlab-ee - Docker Image | Docker Hub](https://hub.docker.com/r/gitlab/gitlab-ee) を使用しています。

**※アップグレードは、必ずバックアップを取ってから行いましょう。**

<!--more-->

## Hashed strage へのマイグレーションに失敗（to 14.0.11）

GitLab のストレージは Legacy storage と Hashed strage の2種類あり、GitLab 14.0 で Legacy storage は廃止となりました。このため 14.0 へのアップグレード前に Hashed strage へのマイグレーションを完了しておかなければなりません。

[Repository storage types | GitLab](https://docs.gitlab.com/ee/administration/repository_storage_types.html)

[Migrate to hashed storage](https://docs.gitlab.com/ee/administration/raketasks/storage.html#migrate-to-hashed-storage)

このマイグレーションが失敗して、リポジトリがロックされた状態になってしまいました。おそらく、マイグレーション中にGitLabの停止してしまった事により、ロックされっぱなしになってしまったようです。（使用頻度の低いリポジトリだったのでロックされていることに気づかなかった）

そして、 Legacy storage が残っていたために `14.0` へのアップグレードに失敗しました。

`.git/index.lock` を削除したんだっけな…忘れてしまった…後で調べて更新するとして…リポジトリのロックをどうにかした後、再度 Hashed strage へのマイグレーションを行い、 `14.0` へのアップグレードに成功しました。

```
gitlab-rake gitlab:storage:migrate_to_hashed
```

## Batched background migrations（to 14.6）

`14.1.8 -> 14.6` のアップグレードで失敗。 14.0 で新しくできた [Batched background migrations](https://docs.gitlab.com/ee/update/#batched-background-migrations) を正しく理解していなかったことが原因でした。

Batched background migrations は、必要なマイグレーションをバックグラウンドで自動実行してくれる機能で、 `14.0` から導入されています。

[14.2.0 | Upgrading GitLab](https://docs.gitlab.com/ee/update/#1420) にある通り、 `14.0` へのアップグレードした後 `14.2` へのアップグレードを行う前に Batched background migrations が終了していることを確認しなければなりません。

私はこれを確認せずに `14.6` へのアップグレードを実行してしまったために失敗してしまいました。

一度バックアップから戻して、 Batched background migrations がすべて完了するのを待ってから `14.6` へのアップグレードし直しました。

## GitLab Mattermost でエラー（to 14.6）

Omnibus GitLab には Mattermost が同梱されています。 GitLab 14.6 にアップグレードしてから Mattermost にアクセスするとエラーが発生してログインできなくなってしまいました。

同梱されている Mattermost は長らく 5.x でしたが、GitLab 14.6 で Mattermost 6.1 にメジャーバージョンアップが行われています。以下に書かれている通り基本的には「Mattermost のドキュメントをよく読め」という感じです。

[Upgrading GitLab Mattermost to 14.6](https://docs.gitlab.com/ee/integration/mattermost/#upgrading-gitlab-mattermost-to-146)

ログには Mattermost Discussion Forum の以下の投稿と同じエラーが出力されていました。（ `users` テーブルの `timezone` カラムへのキャストができない）

[PostgreSQL error when migrating from 5.39.0 to 6.0.0 - Troubleshooting - Mattermost Discussion Forums](https://forum.mattermost.org/t/postgresql-error-when-migrating-from-5-39-0-to-6-0-0/12303)

ワークアラウンドとして回答に書かれている通り、該当カラムの `DEFAULT` を削除して再起動することで正常にアクセスできるようになりました。

```console
gitlab-psql -d mattermost_production
```

で Mattermost のポスグレにログインして

```sql
ALTER TABLE users ALTER COLUMN timezone DROP DEFAULT;
```

また、GitLab 14.6 をクリーンインストールして `users` テーブルを確認したところ、 `timezone` カラムには `DEFAULT` が設定されていませんでした。この対応で問題なさそうです。

## まとめ

Batched background migrations のことを考えると、まとめてアップグレードすることは避け、こまめに更新するようにしたほうがいいんでしょう。