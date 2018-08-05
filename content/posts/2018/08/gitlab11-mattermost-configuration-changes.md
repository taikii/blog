---
title: "GitLab 11.0でMattermost関連の設定項目が削除されている"
date: 2018-08-06T06:30:45+09:00
tags: ["GitLab"]
image: "https://i.gyazo.com/7cea83183c243e5029c58e696f7bbf0d.png"
---

先日、GitLab Omnibus packageを 10.7 から 11.1 にバージョンアップを行ったところ、`gitlab-ctl reconfigure` でコケてしまいました。その時は訳がわからず、とりあえず 10.7 に戻して事なきを得たんですが、後日よくよくログを確認するとしっかり理由が書いてありました。。。ログはちゃんと読もうね＞自分

<!--more-->

### 原因

`gitlab-ctl reconfigure` がコケた原因は、11.0 で廃止されたMattermost系の設定が残っていた為でした。そうです、私はOmnibusにバンドルされているMattermostを使っているのです。これらの設定は `docker-compose.yml` 内でGitLabに環境変数として渡していたのですが、これらを削除することで無事バージョンアップできましたとさ。

{{< figure src="https://i.gyazo.com/efb7dcdfbeb20efc24d58a13974a7752.png" >}}

### Mattermostの設定の廃止について

廃止の理由等々については公式サイト上で説明されていますので読むといいと思います。

[Upgrading GitLab Mattermost from versions prior to 11.0 | GitLab Mattermost | GitLab](https://docs.gitlab.com/omnibus/gitlab-mattermost/#upgrading-gitlab-mattermost-from-versions-prior-to-11-0)

・・・これだけだと乱暴なので軽く解説したいと思います。

Mattermostの設定というのは、本来であれば管理コンソール画面で変更することができ、その設定は `config.json` というファイルに保存されます。しかしGitLab Omnibus packageでは、`gitlab.rb` の設定内容を元に `config.json` を自動生成するということをしていたため、管理コンソールで設定を変更したとしても、`gitlab.rb` の内容で上書きされ設定が元に戻ってしまう[^1]という問題がありました。

[^1]: `gitlab.rb`  から `config.json` が生成されるタイミングは `gitlab-ctl reconfigure` するときです。`gitlab-ctl reconfigure` は、バージョンアップやコンテナの再作成、`gitlab.rb`を変更して適用する際に実行します（されます）。

このため GitLab 10.x までは、Mattermostの設定を変更する際はまず `gitlab.rb` を編集し、そして `gitlab-ctl reconfigure` を実行する、ということをやっていました。本当なら管理コンソールでちゃちゃっと変更できるはずなのに。めんどくさ・・・。

で、11.0ではこの問題に対処するために、`gitlab.rb` から `config.json` を生成することをやめ、必要な設定だけを環境変数でMattermostに渡すように変更したそうです。`config.json` は晴れて管理コンソールの下に帰ってきたのです。

では、 `gitlab.rb` でMattermostの設定をすることが出来なくなったかというとそうではなく、書き方を変えることでこれまで通り設定することが可能です。前述の通りMattermostには環境変数で渡すようになったため、`mattermost[env]` という設定項目にMattermostのお作法に則った環境変数名[^2]で記述します。

[^2]: 大文字で `MM_<CATEGORY>SETTINGS_<ATTRIBUTE>` といった感じ。 --  [Configuration Settings — Mattermost 5.1 documentation](https://docs.mattermost.com/administration/config-settings.html)

一方で廃止されていない設定項目もあります。おそらくこれらの設定はGitLabとしても情報を取得する必要があるのでしょう。

**継続してサポートされるgitlab.rb項目**

```ruby
mattermost_external_url
mattermost['enable']
mattermost['username']
mattermost['group']
mattermost['uid']
mattermost['gid']
mattermost['home']
mattermost['database_name']
mattermost['env']
mattermost['service_use_ssl']
mattermost['service_address']
mattermost['service_port']
mattermost['service_site_url']
mattermost['team_site_name']
mattermost['sql_driver_name']
mattermost['sql_data_source']
mattermost['log_file_directory']
mattermost['file_directory']
mattermost['gitlab_enable']
mattermost['gitlab_secret']
mattermost['gitlab_id']
mattermost['gitlab_scope']
mattermost['gitlab_auth_endpoint']
mattermost['gitlab_token_endpoint']
mattermost['gitlab_user_api_endpoint']
```

`docker-compose.yml`に書くとしたら、10.xと11.xの違いはこんな感じ。抜粋です。

**GitLab 10.x**

```yaml
version: '2'
 
services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        mattermost_external_url 'https://mattermost.example.com'
        mattermost['email_smtp_server'] = 'mailhost.example.com'
        mattermost['email_smtp_port'] = 25
        mattermost['email_feedback_name'] = 'GitLab Mattermost'
        mattermost['email_feedback_email'] = 'mattermost@example.com'
```

**GitLab 11.x**

```yaml
version: '2'
 
services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        mattermost_external_url 'https://mattermost.example.com'
        mattermost['env'] = {
          'MM_EMAILSETTINGS_SMTPSERVER' => 'mailhost.example.com',
          'MM_EMAILSETTINGS_SMTPPORT' => '25',
          'MM_EMAILSETTINGS_FEEDBACKNAME' => 'GitLab Mattermost',
          'MM_EMAILSETTINGS_FEEDBACKEMAIL' => 'mattermost@example.com'
        }
```

### 廃止された設定は削除するだけでいいの？

イイんです！ `gitlab.rb` での設定方法を書きましたが、実際にはこんなことやらなくてイイんです！

`config.json` はコンテナ上では `/var/opt/gitlab/mattermost/config.json` に配置されています。あなたが[公式のマニュアル](https://docs.gitlab.com/omnibus/docker/)に従っているとすれば、`/var/opt/gitlab` 配下のファイルは永続化されているはずです。`config.json` も10.xのものが残っていることになりますので、改めて環境変数で設定を渡してやる必要はありません。ですので廃止された設定は `gitlab.rb` から削除するだけでイイんです！（たぶん）

### さいごに

ちなみにログはこんな感じで出てました。みんなもログはちゃんと確認しようね。

https://gist.github.com/taikii/37d8955a55f7124717e7a38d0673cb6f

